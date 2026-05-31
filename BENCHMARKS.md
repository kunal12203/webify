# Benchmarks

Tested on June 1, 2026 — macOS, Python 3.11, no external dependencies.

## Summary

| Metric | Value |
|--------|-------|
| Reliability | **90%** (9/10 pages return useful content) |
| Avg token savings | **99%** (313 tokens vs 118,724 raw) |
| Cache-hit latency | **3.4ms** avg, 2.0ms p50, 13.1ms p99 |
| First-fetch time | **0.7s** avg (network-bound) |

## Full Results

### First Fetch (cold cache)

| Page | Status | Confidence | Tokens Used | Raw Tokens | Savings | Time |
|------|--------|-----------|-------------|------------|---------|------|
| Python json docs | success | 100 | 376 | 28,179 | 99% | 0.0s |
| Python asyncio docs | success | 100 | 552 | 43,837 | 99% | 0.8s |
| MDN Promise | success | 100 | 210 | 49,007 | 100% | 0.2s |
| React useState | success | 100 | 253 | 110,957 | 100% | 0.3s |
| GitHub REST API | success | 85 | 372 | 343,415 | 100% | 3.2s |
| OpenAI API docs | fallback_needed | 0 | 0 | 1,635 | — | 4.9s |
| Rust Tokio docs | success | 100 | 225 | 14,130 | 98% | 0.4s |
| Next.js routing | success | 100 | 262 | 249,694 | 100% | 0.5s |
| Tailwind CSS flex | success | 100 | 231 | 97,367 | 100% | 0.2s |
| Kubernetes pods | success | 100 | 334 | 131,932 | 100% | 0.6s |

### Cache Hit (warm cache, different queries)

| Query | Tokens | Latency |
|-------|--------|---------|
| json.dumps indent parameter | 299 | 2.4ms |
| gather multiple coroutines | 362 | 3.2ms |
| async await syntax | 411 | 1.9ms |
| update state based on previous | 272 | 3.3ms |
| list repositories for user | 371 | 13.1ms |
| runtime configuration | 99 | 2.0ms |
| route groups | 136 | 1.1ms |
| flex basis | 217 | 1.6ms |
| restart policy | 422 | 1.5ms |

## What "fallback_needed" means

OpenAI's docs use heavy client-side rendering (React SPA). The server returns minimal HTML — the actual content loads via JavaScript. Webify correctly detects this (confidence=0) and signals `fallback_needed` so the caller can use WebFetch instead.

This is by design: returning empty/garbage content silently would be worse than signaling failure.

## Cost comparison

For a typical Claude Code session reading 5 documentation pages:

| Method | Tokens consumed | Estimated cost |
|--------|----------------|----------------|
| WebFetch (full pages) | ~500,000 | ~$1.50 input |
| Webify (graph retrieval) | ~1,500 | ~$0.005 input |
| **Savings** | | **99.7%** |

## Reproduction

```bash
python webify.py build https://docs.python.org/3/library/json.html
python webify.py lookup https://docs.python.org/3/library/json.html "parse JSON string"
python webify.py stats https://docs.python.org/3/library/json.html
```
