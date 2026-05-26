-> Unidirected graph
-> Directed Graph (Digraph)
-> Weighted Graphs
-> Cyclic graph
-> Acyclic Graph
-> Bipartite Graph
-> Directed Acyclic Graph (DAG)

--- 

most important : 

### BFS and DFS

```python
class node:
    def __init__(self, val):
        self.val = val
        self.neighbors = []

    def __str__(self):
        return f"Node({self.val}, neighbors={self.neighbors})"
class Graph:
    def __init__(self):
        self.nodes = {}
        
    def add_node(self, val):
        if val not in self.nodes:
            self.nodes[val] = node(val)
            
    def add_edge(self, u, v, directed=False):
        """works both for dag and normal graphs"""
        self.add_node(u)
        self.add_node(v)
        self.nodes[u].neighbors.append(self.nodes[v])
        if not directed:
            self.nodes[v].neighbors.append(self.nodes[u])

    def get_node(self, val):
        return self.nodes.get(val)
        
g = Graph()
g.add_edge(1, 2)
g.add_edge(1, 3)
g.add_edge(2, 4)
g.add_edge(3, 4)
g.add_edge(4, 5)
g.add_edge(5, 6)
g.add_edge(6, 7)
g.add_edge(5, 7)

def traverse_bfs(g, start):
    from collections import deque
    
    q   = deque()
    vis = set()
    q.append(g.get_node(start))
    vis.add(g.get_node(start))
    
    while q :
        curr = q.popleft()
        print(curr.val , " -> ", end="")
        try:
            for neigh in curr.neighbors:
                if neigh not in vis:
                    q.append(neigh)
                    vis.add(neigh)
        except Exception as e:
            print("failed")
visted = set()

def traverse_dfs(g, curr):
    if curr not in visted :
        visted.add(curr)
    else:
        return
    print(curr.val, " -> " , end="")
    for nei in curr.neighbors:
        traverse_dfs(g, nei)    
        
traverse_bfs(g, 1)
print()
print("=" * 50)
traverse_dfs(g, g.get_node(1))
```

--- 
## Dijktras

```python 
def dijkstra(graph, start):
    import heapq
    pq = []
    
    heapq.heappush(pq, (0, start))
    dist = {node: float('inf') for node in graph}
    dist[start] = 0
    
    vis = set()
    while pq:
        d, node = heapq.heappop(pq)
        if node in vis:
            continue
            
        vis.add(node)
        for neigh, weight in graph[node]:
            ndist = d + weight
            # relaxation
            if ndist < dist[neigh]:
                dist[neigh] = ndist
                heapq.heappush(pq, (ndist, neigh))
    return dist
```

topics to do 

optimisation strats 
kahns algorithm and DAG wali shit 
graph theory ki theory

