## 1. CMD and ENTRYPOINT diffrence ? 
**The Real Difference:**

`CMD` — default command, **easily overridden** at runtime
```bash
docker run myapp echo "hello"  # overrides CMD completely
```

`ENTRYPOINT` — the **fixed executable** that always runs, can't be easily overridden
```bash
docker run myapp echo "hello"  # "echo hello" becomes ARGUMENTS to entrypoint
```

**The magic is when you combine them:**

dockerfile
```dockerfile
ENTRYPOINT ["node"]    # always runs node
CMD ["index.js"]       # default file, but user can override
```
```bash
docker run myapp                  # runs: node index.js
docker run myapp other.js         # runs: node other.js
```

ENTRYPOINT = **the verb** (what to run) CMD = **the default noun** (what to run it with)

---

**Real world analogy** — ENTRYPOINT is like a locked calculator app that always opens. CMD is the default equation pre-filled in — user can change the equation but can't change the app.


## 2. Diffrence between the add and copy in docker : 

Always prefer `COPY` unless you **specifically** need tar extraction or URL fetching. `ADD` with URLs is actually discouraged — use `RUN curl` or `RUN wget` instead because it gives you more control (and a single cacheable layer). 