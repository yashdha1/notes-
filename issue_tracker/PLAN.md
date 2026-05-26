

scafolding : 

1. prek (ruff and biome and the testcases of the file changed) 
2. CI on the dev. 
3. fast-api project structure. 
4. js project structure. 
5. DockerFiles
6. docker-composes
7. runner on the gitlab


*backend* : 
1. connections 
	1. connection pools.
	2. schema making. models making. 
	3. following TDD flow
		1. testcases  pytest 
		2. write the code

	 4. commit

*frontend* :
1. stores making. 
2. Connection pools axios and zustand stores.
3. pages + components. 
4. connect the made pages from the backend to the frontend.
5. manual testing.



search operations : 

milisearch project integration

for the post-service few fields ; How can I do this using fastapi + message broker + docker ?

12:19 AM

2 / 2

## Claude responded: Here's the full architecture for integrating Meilisearch into your post-service using FastAPI + a message broker + Docker.

Here's the full architecture for integrating Meilisearch into your post-service using FastAPI + a message broker + Docker.

The idea: when a post is created/updated/deleted via FastAPI, it publishes an event to a message broker (RabbitMQ or Redis Streams). A separate consumer service listens and syncs to Meilisearch — keeping your write path fast and Meilisearch always in sync.

V

visualize

V

visualize show_widget

Click any box above to drill in. Here's how each layer works:

---

**Post Service (FastAPI)**

The service does two things atomically: write to Postgres, then publish an event. Use a pattern like _transactional outbox_ or keep it simple with a try/publish after commit.

python

```python
# schemas.py
class PostEvent(BaseModel):
    event: Literal["created", "updated", "deleted"]
    post_id: str
    data: dict | None  # None for deletes
```

python

```python
# routes/posts.py
@router.post("/posts")
async def create_post(payload: PostCreate, db: Session = Depends(get_db)):
    post = crud.create_post(db, payload)
    await broker.publish("posts.events", PostEvent(
        event="created", post_id=str(post.id), data=post.dict()
    ))
    return post
```

---

**Message Broker**

Two solid choices:

||RabbitMQ|Redis Streams|
|---|---|---|
|Best for|Complex routing, multiple consumers|Simple, already using Redis|
|Python lib|`aio-pika`|`redis-py` async|
|Durability|Durable queues|Persistent by default|

With RabbitMQ (`aio-pika`):

python

```python
# broker.py
async def publish(routing_key: str, event: PostEvent):
    message = aio_pika.Message(
        body=event.json().encode(),
        delivery_mode=aio_pika.DeliveryMode.PERSISTENT,
    )
    await exchange.publish(message, routing_key=routing_key)
```

---

**Search Consumer Service**

A standalone Python process (or second FastAPI app with a lifespan worker) that listens and calls Meilisearch:

python

```python
# consumer.py
import meilisearch

meili = meilisearch.Client("http://meilisearch:7700", "MASTER_KEY")
index = meili.index("posts")

async def handle_event(event: PostEvent):
    if event.event in ("created", "updated"):
        index.add_documents([{**event.data, "id": event.post_id}])
    elif event.event == "deleted":
        index.delete_document(event.post_id)
```

---

**Docker Compose**

yaml

```yaml
version: "3.9"
services:
  post-service:
    build: ./post-service
    ports: ["8000:8000"]
    environment:
      DATABASE_URL: postgresql://user:pass@postgres/posts
      RABBITMQ_URL: amqp://guest:guest@rabbitmq/
    depends_on: [postgres, rabbitmq]

  search-consumer:
    build: ./search-consumer
    environment:
      RABBITMQ_URL: amqp://guest:guest@rabbitmq/
      MEILI_URL: http://meilisearch:7700
      MEILI_KEY: your_master_key
    depends_on: [rabbitmq, meilisearch]

  postgres:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: posts
      POSTGRES_USER: user
      POSTGRES_PASSWORD: pass
    volumes: [pg_data:/var/lib/postgresql/data]

  rabbitmq:
    image: rabbitmq:3-management-alpine
    ports: ["5672:5672", "15672:15672"]

  meilisearch:
    image: getmeili/meilisearch:v1.7
    ports: ["7700:7700"]
    environment:
      MEILI_MASTER_KEY: your_master_key
    volumes: [meili_data:/meili_data]

volumes:
  pg_data:
  meili_data:
```

---

**Meilisearch index settings** (run once on startup):

python

```python
index.update_settings({
    "searchableAttributes": ["title", "content", "tags"],
    "filterableAttributes": ["author_id", "status", "created_at"],
    "sortableAttributes": ["created_at", "updated_at"],
    "rankingRules": ["words", "typo", "proximity", "attribute", "sort", "exactness"]
})
```

---

**Key points to keep in mind:**

- The consumer is **idempotent** — `add_documents` with the same `id` is an upsert, safe to retry.
- Add a **dead-letter queue** in RabbitMQ for failed indexing events so nothing is silently lost.
- For search _queries_, your client can hit Meilisearch directly or proxy through FastAPI — direct is faster for read-heavy workloads.
- Run `docker compose up --build` and all five services come up together.

## You said: ok so how does the search works ?

ok so how does the search works ?

12:36 AM

## Claude responded: Great question.

Great question. Let me break it into two parts: what happens inside Meilisearch when you index a document, and what the actual query request/response looks like from your FastAPI side.

First, here's the internal pipeline — what Meilisearch does when a search query arrives:

V

visualize

V

visualize show_widget

Now, here's how you actually call it from your FastAPI service — two options depending on whether you proxy or go direct.

---

**Option A — client queries Meilisearch directly (recommended for read-heavy)**

The frontend hits Meilisearch's HTTP API using a _search-only API key_ (not the master key):

python

```python
# On startup — create a restricted key
meili = meilisearch.Client("http://meilisearch:7700", "MASTER_KEY")

search_key = meili.create_key({
    "actions": ["search"],
    "indexes": ["posts"],
    "expiresAt": None,
})
# expose search_key.key to the frontend safely
```

Frontend calls:

```
GET http://meilisearch:7700/indexes/posts/search?q=fastapi+tutorial&limit=10
```

---

**Option B — proxy through FastAPI (adds auth, rate limiting, field hiding)**

python

```python
# routes/search.py
from meilisearch import Client

meili = Client("http://meilisearch:7700", settings.MEILI_KEY)

@router.get("/search")
async def search_posts(
    q: str,
    page: int = 1,
    limit: int = 20,
    author_id: str | None = None,
    current_user = Depends(get_current_user),  # your auth
):
    filters = []
    if author_id:
        filters.append(f"author_id = {author_id}")

    results = meili.index("posts").search(q, {
        "limit": limit,
        "offset": (page - 1) * limit,
        "filter": filters or None,
        "attributesToHighlight": ["title", "content"],
        "highlightPreTag": "<mark>",
        "highlightPostTag": "</mark>",
        "attributesToRetrieve": ["id", "title", "author_id", "created_at"],
    })
    return results
```

---

**What the response looks like:**

json

```json
{
  "hits": [
    {
      "id": "abc123",
      "title": "FastAPI tutorial",
      "author_id": "user_1",
      "created_at": "2026-04-01T10:00:00Z",
      "_formatted": {
        "title": "<mark>FastAPI</mark> <mark>tutorial</mark>"
      }
    }
  ],
  "query": "fastapi tutorial",
  "processingTimeMs": 3,
  "limit": 20,
  "offset": 0,
  "estimatedTotalHits": 42
}
```

---

**Key things to know about how the search works:**

Meilisearch is typo-tolerant by default — "tutoriel" finds "tutorial" because it allows up to 2 character edits (Levenshtein distance). It also does prefix matching, so typing "fast" already returns results for "fastapi".

Ranking happens in order through the rules you set. The default order is words (more query words matched = higher), then typo (fewer typos = higher), then proximity (words closer together = higher), then attribute (match in `title` > match in `content`), then your custom sort rules like `created_at:desc`.

Filters (`author_id = X`, `status = published`) run _before_ ranking — they don't affect the score, they just cut candidates. That's why `filterableAttributes` must be declared upfront in index settings; Meilisearch pre-builds a structure for those fields.

For pagination, prefer `offset`/`limit` for small datasets and use `totalHits` for total count. For deep pagination (thousands of pages), Meilisearch caps `estimatedTotalHits` at a threshold for performance — keep that in mind for your UI.




--- 

