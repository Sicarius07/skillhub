---
name: caching-strategy
description: Decide what to cache, where to place the cache, which read/write pattern to use, and how to invalidate safely — covering client/CDN/application/database layers, TTLs, cache-aside vs. write-through, stampede protection, and staleness trade-offs; use when responses are slow, load is high, costs are rising, or an existing cache is serving stale or inconsistent data.
---

# Caching Strategy

Caching trades freshness and complexity for speed and reduced load. The hard parts are choosing what is safe to cache, placing it at the right layer, and invalidating it correctly ("there are only two hard things..."). This skill provides a decision framework so caches speed things up without silently serving wrong data.

## When to use this skill

- Endpoints or queries are slow under load or expensive to compute.
- Downstream systems (databases, third-party APIs) are a bottleneck or cost driver.
- You need to reduce latency for read-heavy workloads.
- An existing cache is returning stale, inconsistent, or thundering-herd behavior.

## Instructions

1. **Confirm caching is the right tool.** Measure first. If the query can be made fast with an index or fewer round-trips, fix that before adding a cache.
2. **Classify the data.** Determine read/write ratio, tolerance for staleness, and whether it is per-user or shared. Cache read-heavy, staleness-tolerant, shared data most aggressively.
3. **Choose the layer.** Client/browser and CDN for static or public content; application/in-memory (e.g., LRU) for hot per-instance data; a distributed cache (e.g., Redis/Memcached) for shared state across instances; the database's own buffer/materialized views for query results.
4. **Pick a read pattern.** *Cache-aside* (app checks cache, loads on miss, populates) is the common default. *Read-through* delegates loading to the cache layer.
5. **Pick a write pattern.** *Write-through* (write cache + store together) keeps them consistent at write cost; *write-behind* (async flush) is faster but riskier; with cache-aside, invalidate or update the entry on write.
6. **Set TTLs intentionally.** Every cached entry should expire. Choose a TTL from the acceptable staleness window; add small random jitter to avoid synchronized expiry.
7. **Design invalidation.** Prefer explicit invalidation on write for correctness-sensitive data; rely on TTL for the rest. Use versioned keys (`user:123:v4`) to invalidate whole groups cheaply.
8. **Protect against stampedes.** Use request coalescing / a lock so only one caller recomputes a hot missing key; consider serving stale-while-revalidate.
9. **Key carefully.** Include all inputs that affect the result (user, locale, version) in the cache key; never cache private data under a shared key.
10. **Make it observable.** Track hit rate, miss rate, and eviction rate. A low hit rate means the cache isn't earning its complexity.

## Examples

Cache-aside read with TTL and jitter:

```python
def get_product(pid):
    key = f"product:{pid}:v2"
    cached = cache.get(key)
    if cached is not None:
        return cached
    product = db.load_product(pid)          # miss: load from source
    cache.set(key, product, ttl=300 + random.randint(0, 30))  # jitter
    return product

def update_product(pid, data):
    db.update_product(pid, data)
    cache.delete(f"product:{pid}:v2")       # invalidate on write
```

Stampede protection via single-flight:

```python
def get_expensive(key):
    val = cache.get(key)
    if val is not None:
        return val
    with lock(f"lock:{key}"):               # only one worker recomputes
        val = cache.get(key)                # double-check after acquiring
        if val is None:
            val = recompute()
            cache.set(key, val, ttl=60)
    return val
```

## Checklist

- [ ] The slowness was measured and caching is the appropriate fix.
- [ ] Data is classified by read/write ratio and staleness tolerance.
- [ ] The cache sits at the layer matching the access pattern.
- [ ] Read and write patterns are chosen explicitly (cache-aside, write-through, etc.).
- [ ] Every entry has a TTL with jitter.
- [ ] Correctness-sensitive data is invalidated on write, not just by TTL.
- [ ] Cache keys include all result-affecting inputs; no private data under shared keys.
- [ ] Stampede protection exists for hot keys.
- [ ] Hit/miss/eviction metrics are monitored.
