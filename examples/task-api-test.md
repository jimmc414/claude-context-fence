---
description: "API and load testing - API is too slow, server can't handle traffic, how many users can it support, breaks under load, HTTP benchmarking, and performance analysis (ab, wrk, hey, k6, vegeta, artillery, hurl)"
when_to_use: "Use when: API is too slow and need to measure, server can't handle enough traffic, want to know how many concurrent users it supports, application breaks under load, need to find the breaking point, measuring request latency, validating API correctness, stress testing before launch. Triggers: API too slow, server can't handle traffic, how many users, breaks under load, find breaking point, performance degraded, response time too high, load test, benchmark, stress test, api test, k6, wrk, vegeta, requests per second, latency, performance test, ab, hey, artillery, hurl, schemathesis, api performance, response time, concurrent users, rate limit test, graphql test, websocket test, soak test, spike test, latency percentile, throughput, endpoint benchmark."
when-to-use: "Use when: API is too slow and need to measure, server can't handle enough traffic, want to know how many concurrent users it supports, application breaks under load, need to find the breaking point, measuring request latency, validating API correctness, stress testing before launch. Triggers: API too slow, server can't handle traffic, how many users, breaks under load, find breaking point, performance degraded, response time too high, load test, benchmark, stress test, api test, k6, wrk, vegeta, requests per second, latency, performance test, ab, hey, artillery, hurl, schemathesis, api performance, response time, concurrent users, rate limit test, graphql test, websocket test, soak test, spike test, latency percentile, throughput, endpoint benchmark."
argument-hint: "[describe the API or load testing task]"
context: inherit
model: inherit
allowed-tools: Skill, Read
---

# API & Load Testing Task Router

You have access to the **full conversation context**. Your job is to analyze the user's testing need and build a well-formed argument for the recipes skill.

**User's request**: $ARGUMENTS

---

## Your Task

API/load testing depends on what to test, how hard, and what metrics matter. Analyze conversation to identify:

### 1. Goal
What kind of test?
- Quick benchmark (how fast is this endpoint?)
- Load test (how many concurrent users?)
- Stress test (find the breaking point)
- Spike test (sudden traffic burst)
- Soak test (long duration stability)
- API correctness (assertions on responses)
- Fuzz testing (find edge cases)

### 2. Target
- Endpoint URL from conversation
- Multiple endpoints?
- Authentication needed?
- Request body/method?

### 3. Parameters
- Concurrency level (users/connections)
- Duration or request count
- Target rate (requests/sec)
- Ramp-up pattern?
- Thresholds (p95 < 200ms, error rate < 1%)

### 4. Context
- What tool preference? (k6, wrk, vegeta, etc.)
- Prior results to compare against
- Server environment (local, staging, production)
- API spec file (OpenAPI/Swagger)

---

## Goal --> Tool Guide

| Goal | Best Tool | Argument Prefix |
|------|----------|----------------|
| Quick benchmark | ab/hey | "benchmark" |
| Steady load | wrk | "load test with wrk" |
| Constant rate | wrk2/vegeta | "rate test" |
| Scripted scenarios | k6 | "load test with k6" |
| Complex scenarios | artillery | "load test with artillery" |
| API validation | hurl | "validate api" |
| Fuzz testing | schemathesis | "fuzz test" |
| Latency histogram | wrk2/vegeta | "measure latency" |

---

## Argument Format

```
<test type> on <url> - concurrency: <N>, duration: <time>, rate: <rps>, auth: <if needed>, method: <if not GET>, body: <if needed>
```

### Examples

**Quick benchmark:**
```
benchmark http://localhost:8080/api/health - 1000 requests, 10 concurrent
```

**Load test with k6:**
```
load test with k6 http://localhost:8080/api/users - ramp 10 to 100 VUs over 2min, hold 5min, p95 < 500ms
```

**Rate-limited test:**
```
rate test http://localhost:8080/api/search - constant 200 req/s for 60s, POST body: {"q": "test"}, bearer auth
```

**API validation:**
```
validate api http://localhost:8080 - test GET /users returns 200, POST /users with body creates resource, check response schema
```

**Soak test:**
```
load test with k6 http://localhost:8080/api - 50 VUs for 30 minutes, watch for memory leaks, log errors
```

---

## Invoke Recipes

Once you've built the argument, use the Skill tool:

```
Skill: task-api-test-recipes
Args: <your constructed argument>
```

### When to Ask Instead

**Ask the user if:**
- Target URL not clear from context
- Test intensity unclear (could impact production)
- Auth credentials needed but not provided
- Tool preference not evident

**Use Read tool first if:**
- Need to check an existing k6 script
- Need to read an OpenAPI spec file
- Need to check a hurl test file
