# Kafka — Usage Guide

Apache Kafka 4.x running in **KRaft mode** (no ZooKeeper).  
UI: **Kafdrop** at `http://localhost:9000`

---

## Topics

| Topic | Variable | Purpose |
|---|---|---|
| `notifications` | `KAFKA_TOPIC_NOTIFICATIONS` | Email / in-app notification events |
| `search` | `KAFKA_TOPIC_SEARCH` | Search index sync events |

Both topics are created automatically on first container start via `init/init-topics.sh`.

---

## Event Types

| Event key         | Variable                      | Routed to                 |
| ----------------- | ----------------------------- | ------------------------- |
| `issue.created`   | `KAFKA_EVENT_ISSUE_CREATED`   | `notifications`, `search` |
| `project.created` | `KAFKA_EVENT_PROJECT_CREATED` | `notifications`, `search` |

---

## Publishing Events

Use `aiokafka` (async) or `kafka-python` (sync) from any backend service.

```python
import json
import os
from aiokafka import AIOKafkaProducer

BOOTSTRAP = os.getenv("KAFKA_BOOTSTRAP_SERVERS_INTERNAL", "kafka:9092")
TOPIC     = os.getenv("KAFKA_TOPIC_NOTIFICATIONS", "notifications")

async def publish(event_type: str, payload: dict) -> None:
    producer = AIOKafkaProducer(bootstrap_servers=BOOTSTRAP)
    await producer.start()
    try:
        message = {"event": event_type, "data": payload}
        await producer.send_and_wait(TOPIC, json.dumps(message).encode())
    finally:
        await producer.stop()
```

**Example — issue created event:**
```python
await publish(
    event_type=os.getenv("KAFKA_EVENT_ISSUE_CREATED"),
    payload={"issue_id": "abc123", "project_id": "proj1", "assignee_email": "user@example.com"},
)
```

From the host (local dev) use `KAFKA_BOOTSTRAP_SERVERS_EXTERNAL` (`localhost:9094`).  
From inside the Docker network use `KAFKA_BOOTSTRAP_SERVERS_INTERNAL` (`kafka:9092`).

---

## Consuming Events

```python
import json
import os
from aiokafka import AIOKafkaConsumer

BOOTSTRAP = os.getenv("KAFKA_BOOTSTRAP_SERVERS_INTERNAL", "kafka:9092")
TOPIC     = os.getenv("KAFKA_TOPIC_NOTIFICATIONS", "notifications")

async def consume() -> None:
    consumer = AIOKafkaConsumer(
        TOPIC,
        bootstrap_servers=BOOTSTRAP,
        group_id="notification-service",
        auto_offset_reset="earliest",
    )
    await consumer.start()
    try:
        async for msg in consumer:
            event = json.loads(msg.value)
            print(f"[{event['event']}] {event['data']}")
    finally:
        await consumer.stop()
```

Set `group_id` to a unique name per service so each service gets its own offset tracking.

---

## Environment Variables

### Broker identity & KRaft

| Variable | Value | Description |
|---|---|---|
| `KAFKA_NODE_ID` | `1` | Unique broker node ID |
| `KAFKA_PROCESS_ROLES` | `broker,controller` | Node acts as both broker and KRaft controller (single-node) |
| `CLUSTER_ID` | from `KAFKA_KRAFT_CLUSTER_ID` | Base64 UUID that permanently identifies the KRaft cluster |
| `KAFKA_CONTROLLER_QUORUM_VOTERS` | `1@kafka:9093` | Controller quorum membership — `<node_id>@<host>:<port>` |
| `KAFKA_CONTROLLER_LISTENER_NAMES` | `CONTROLLER` | Name of the listener used for controller traffic |

### Listeners

| Variable | Value | Description |
|---|---|---|
| `KAFKA_LISTENER_SECURITY_PROTOCOL_MAP` | `CONTROLLER:PLAINTEXT,...` | Maps each listener name to its security protocol |
| `KAFKA_LISTENERS` | `PLAINTEXT://0.0.0.0:9092,...` | Interfaces the broker binds to inside the container |
| `KAFKA_ADVERTISED_LISTENERS` | `PLAINTEXT://kafka:9092,EXTERNAL://localhost:9094` | Addresses clients use to connect. `PLAINTEXT` for Docker-internal, `EXTERNAL` for host access |
| `KAFKA_INTER_BROKER_LISTENER_NAME` | `PLAINTEXT` | Listener used for broker-to-broker communication |

### Replication (single-node defaults)

| Variable | Value | Description |
|---|---|---|
| `KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR` | `1` | Replication factor for `__consumer_offsets` — must be `1` with a single broker |
| `KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR` | `1` | Replication factor for the transaction log — must be `1` with a single broker |
| `KAFKA_TRANSACTION_STATE_LOG_MIN_ISR` | `1` | Minimum in-sync replicas for the transaction log |

### Application / routing

Stored in `infra/.env`, injected into backend services (not the broker itself).

| Variable | Example | Description |
|---|---|---|
| `KAFKA_KRAFT_CLUSTER_ID` | `MDEyMzQ1Njc4...` | Cluster ID used to format KRaft storage |
| `KAFKA_BOOTSTRAP_SERVERS_INTERNAL` | `kafka:9092` | Bootstrap address for services running inside Docker |
| `KAFKA_BOOTSTRAP_SERVERS_EXTERNAL` | `localhost:9094` | Bootstrap address for local development / host tools |
| `KAFKA_TOPIC_NOTIFICATIONS` | `notifications` | Topic name for notification events |
| `KAFKA_TOPIC_SEARCH` | `search` | Topic name for search index sync events |
| `KAFKA_EVENT_ISSUE_CREATED` | `issue.created` | Event type key for issue-created messages |
| `KAFKA_EVENT_PROJECT_CREATED` | `project.created` | Event type key for project-created messages |
| `SEARCH_INDEX_ISSUES` | `issues` | Meilisearch index name consumed by the search service |
| `SEARCH_INDEX_PROJECTS` | `projects` | Meilisearch index name consumed by the search service |

---

## Quick Reference

```bash
# Start Kafka + UI
docker compose up -d kafka kafka_ui

# Tail Kafka logs (includes topic init output)
docker compose logs -f kafka

# Open Kafdrop UI
open http://localhost:9000
```



1. caching. -> search, featching for the project. and evection strategies. optimisations. 
2. pagination in the issues page.
3. Any adhoc in the entire project.