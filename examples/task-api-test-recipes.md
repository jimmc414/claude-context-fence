---
description: "API and load testing recipes reference (internal)"
when_to_use: "Receives prepared arguments from task-api-test with endpoint, parameters, and context included"
when-to-use: "Receives prepared arguments from task-api-test with endpoint, parameters, and context included"
context: fork
model: inherit
allowed-tools: Read
---

# API & Load Testing Recipes

**Tools:** ab, wrk, wrk2, hey, vegeta, k6, artillery, hurl, schemathesis

---

**Task**: $ARGUMENTS

---

> See task-http for comprehensive curl usage (methods, auth, debugging). This skill focuses on load testing and API validation.

## Quick Reference

| Task | Command |
|------|---------|
| Quick benchmark | `ab -n 1000 -c 10 http://localhost:8080/` |
| Steady load | `wrk -t4 -c100 -d30s http://localhost:8080/` |
| Constant rate | `echo 'GET http://localhost:8080/' \| vegeta attack -rate=100 -duration=30s \| vegeta report` |
| Scripted test | `k6 run script.js` |
| Latency detail | `wrk -t4 -c100 -d30s --latency http://localhost:8080/` |
| API chain test | `hurl --test api_test.hurl` |
| Fuzz OpenAPI | `schemathesis run http://localhost:8080/openapi.json` |
| Quick artillery | `artillery quick --count 100 -n 10 http://localhost:8080/` |
| Simple hey test | `hey -n 1000 -c 50 http://localhost:8080/` |

---

## When to Use What

| Task | Best Tool | Why |
|------|-----------|-----|
| Quick HTTP benchmark | ab | Pre-installed with Apache, simple |
| Concurrent load test | wrk | Lightweight, Lua scriptable |
| Rate-limited load test | vegeta | Precise rate control |
| Simple Go-based bench | hey | Single binary, easy |
| Scripted load scenarios | k6 | JavaScript scenarios, CI-friendly |
| Complex test scenarios | artillery | YAML config, multiple protocols |
| API chain testing | hurl | Sequential HTTP requests in plain text |
| OpenAPI fuzz testing | schemathesis | Auto-generate tests from spec |

---

## Apache Bench (ab)

### Basic Usage
```bash
ab -n 1000 -c 10 http://localhost:8080/             # 1000 requests, 10 concurrent
ab -n 5000 -c 50 -k http://localhost:8080/           # Keep-alive enabled
ab -t 30 -c 20 http://localhost:8080/                # Time-limited: 30 seconds
```

### Custom Headers and Auth
```bash
ab -n 1000 -c 10 -H 'Authorization: Bearer TOKEN' http://localhost:8080/api/users
ab -n 1000 -c 10 -H 'Accept: application/json' -H 'X-API-Key: KEY123' http://localhost:8080/
```

### POST Requests
```bash
# POST with inline body (write body to file first)
echo '{"name":"test","email":"t@t.com"}' > /tmp/post-data.json
ab -n 1000 -c 10 -p /tmp/post-data.json -T 'application/json' http://localhost:8080/api/users
```

### Output and Reporting
```bash
ab -n 1000 -c 10 -g /tmp/ab-results.tsv http://localhost:8080/   # Gnuplot output
ab -n 1000 -c 10 -e /tmp/ab-percentiles.csv http://localhost:8080/ # CSV percentiles
ab -n 1000 -c 10 -w http://localhost:8080/                        # HTML table output
```

### Output Interpretation
- **Requests per second** - Throughput (higher is better)
- **Time per request (mean)** - Average latency per user
- **Time per request (mean, across all concurrent requests)** - Server processing time
- **Percentage of requests served within a certain time** - p50/p66/p75/p80/p90/p95/p98/p99/p100

### Gotchas
- URL **must** end with trailing slash for root path
- Only HTTP/1.1 (no HTTP/2 support)
- Single-threaded: limited to one CPU core
- `-c` cannot exceed `-n`

---

## wrk

### Basic Usage
```bash
wrk -t4 -c100 -d30s http://localhost:8080/                   # 4 threads, 100 connections, 30s
wrk -t8 -c200 -d60s --latency http://localhost:8080/         # With latency percentiles
wrk -t2 -c50 -d10s http://localhost:8080/api/health          # Quick health check
```

### Lua Scripting - Setup
```lua
-- post.lua: POST with JSON body
wrk.method = "POST"
wrk.body   = '{"username":"test","password":"secret"}'
wrk.headers["Content-Type"] = "application/json"
wrk.headers["Authorization"] = "Bearer TOKEN"
```
```bash
wrk -t4 -c100 -d30s -s post.lua http://localhost:8080/api/users
```

### Lua Scripting - Custom Request Function
```lua
-- random-path.lua: different path per request
counter = 0
request = function()
  counter = counter + 1
  return wrk.format("GET", "/api/items/" .. counter)
end
```

### Lua Scripting - Response Processing
```lua
-- log-errors.lua: track non-200 responses
errors = 0
response = function(status, headers, body)
  if status ~= 200 then
    errors = errors + 1
  end
end
done = function(summary, latency, requests)
  io.write(string.format("Errors: %d\n", errors))
  io.write(string.format("Avg latency: %.2fms\n", latency.mean / 1000))
  io.write(string.format("Max latency: %.2fms\n", latency.max / 1000))
  io.write(string.format("Requests/sec: %.2f\n", summary.requests / (summary.duration / 1000000)))
end
```

### Lua Scripting - Pipeline
```lua
-- pipeline.lua: send multiple requests per connection
init = function(args)
  local r = {}
  r[1] = wrk.format("GET", "/api/users")
  r[2] = wrk.format("GET", "/api/items")
  r[3] = wrk.format("GET", "/api/health")
  req = table.concat(r)
end
request = function()
  return req
end
```

### Output Interpretation
- **Latency avg/stdev/max/+Stdev** - Response time distribution
- **Req/Sec avg/stdev/max/+Stdev** - Per-thread throughput
- **Latency Distribution** (with --latency) - 50%/75%/90%/99% percentiles
- **Socket errors** - connect/read/write/timeout counts
- **Non-2xx or 3xx responses** - Error responses (only shown if > 0)

---

## wrk2

### Basic Usage (Constant Rate)
```bash
wrk2 -t4 -c100 -d30s -R200 http://localhost:8080/           # 200 req/s target rate
wrk2 -t4 -c100 -d30s -R200 --latency http://localhost:8080/ # With latency histogram
wrk2 -t2 -c50 -d60s -R500 --u_latency http://localhost:8080/ # Microsecond precision
```

### Why wrk2 Over wrk
- **wrk** measures max throughput (closed-loop), suffers from coordinated omission
- **wrk2** maintains constant request rate (open-loop), accurate latency measurement
- Use wrk for "how fast can this go?", wrk2 for "what's the latency at X req/s?"
- wrk2 uses same Lua scripting as wrk

### Latency Histogram Interpretation
- Detailed percentile breakdown: 50/75/90/99/99.9/99.99/99.999/100
- Compare p50 vs p99: large gap = high tail latency
- If p99.9 >> p99: occasional slow requests (GC pauses, cache misses)

---

## hey

### Basic Usage
```bash
hey -n 1000 -c 50 http://localhost:8080/                     # 1000 requests, 50 concurrent
hey -z 30s -c 100 http://localhost:8080/                     # 30 seconds duration
hey -n 500 -c 20 -q 50 http://localhost:8080/                # Rate limited: 50 req/s per worker
```

### Methods and Bodies
```bash
hey -n 1000 -c 50 -m POST -d '{"name":"test"}' \
  -T 'application/json' http://localhost:8080/api/users
hey -n 1000 -c 50 -m POST -D /tmp/body.json \
  -T 'application/json' http://localhost:8080/api/users
hey -n 1000 -c 50 -m PUT -d '{"status":"active"}' \
  -T 'application/json' http://localhost:8080/api/users/1
```

### Headers and Auth
```bash
hey -n 1000 -c 50 -H 'Authorization: Bearer TOKEN' http://localhost:8080/
hey -n 1000 -c 50 -H 'X-API-Key: KEY123' -H 'Accept: application/json' http://localhost:8080/
```

### Options
```bash
hey -n 1000 -c 50 -disable-keepalive http://localhost:8080/  # Disable keep-alive
hey -n 1000 -c 50 -disable-redirects http://localhost:8080/  # Don't follow redirects
hey -z 30s -c 50 -o csv http://localhost:8080/               # CSV output for analysis
hey -z 30s -c 50 -cpus 4 http://localhost:8080/              # Limit CPU cores
```

### Output Interpretation
- **Summary** - Total time, slowest, fastest, average, requests/sec
- **Response time histogram** - Visual distribution bars
- **Latency distribution** - 10%/25%/50%/75%/90%/95%/99% percentiles
- **Status code distribution** - Count per HTTP status code
- **Error distribution** - Connection errors grouped by type

### Advantages Over ab
- Supports HTTP/2
- Better default output format
- Cross-platform (single binary)
- Built-in rate limiting per worker

---

## vegeta

### Basic Attack
```bash
echo 'GET http://localhost:8080/' | vegeta attack -duration=30s | vegeta report
echo 'GET http://localhost:8080/' | vegeta attack -rate=100 -duration=30s | vegeta report
echo 'GET http://localhost:8080/' | vegeta attack -rate=0 -duration=10s -max-workers=50 | vegeta report  # Max throughput
```

### Rate and Duration
```bash
echo 'GET http://localhost:8080/' | vegeta attack -rate=200/s -duration=60s | vegeta report
echo 'GET http://localhost:8080/' | vegeta attack -rate=50/1s -duration=30s -timeout=5s | vegeta report
echo 'GET http://localhost:8080/' | vegeta attack -rate=1000 -duration=10s -workers=20 -connections=100 | vegeta report
```

### Headers and Auth
```bash
echo 'GET http://localhost:8080/api/users' | \
  vegeta attack -rate=100 -duration=30s -header 'Authorization: Bearer TOKEN' -header 'Accept: application/json' | \
  vegeta report
```

### POST with Body
```bash
echo 'POST http://localhost:8080/api/users
Content-Type: application/json
@/tmp/body.json' | vegeta attack -rate=50 -duration=30s | vegeta report
```

### Targets File (Multiple Endpoints)
```bash
# targets.txt:
# GET http://localhost:8080/api/users
# Authorization: Bearer TOKEN
#
# POST http://localhost:8080/api/items
# Content-Type: application/json
# @/tmp/item.json
#
# GET http://localhost:8080/api/health

vegeta attack -targets=targets.txt -rate=100 -duration=30s | vegeta report
```

### Reporting
```bash
# Text report (default)
vegeta attack -rate=100 -duration=30s < targets.txt | vegeta report

# JSON report
vegeta attack -rate=100 -duration=30s < targets.txt | vegeta report -type=json

# Latency histogram with custom buckets
vegeta attack -rate=100 -duration=30s < targets.txt | vegeta report -type='hist[0,5ms,10ms,25ms,50ms,100ms,250ms,500ms,1s]'

# HDR histogram plot
vegeta attack -rate=100 -duration=30s < targets.txt | vegeta report -type=hdrplot

# Interactive HTML plot
echo 'GET http://localhost:8080/' | vegeta attack -rate=100 -duration=30s | tee /tmp/results.bin | vegeta report
vegeta plot /tmp/results.bin > /tmp/plot.html
```

### Full Pipeline (Save and Analyze)
```bash
echo 'GET http://localhost:8080/' | \
  vegeta attack -rate=100/s -duration=30s | \
  tee /tmp/results.bin | \
  vegeta report

# Then analyze saved results
vegeta report -type=text < /tmp/results.bin
vegeta report -type='hist[0,50ms,100ms,200ms,500ms,1s]' < /tmp/results.bin
vegeta plot /tmp/results.bin > /tmp/vegeta-plot.html
```

### Distributed Testing
```bash
# Machine 1
echo 'GET http://target/' | vegeta attack -rate=500 -duration=60s > /tmp/results1.bin
# Machine 2
echo 'GET http://target/' | vegeta attack -rate=500 -duration=60s > /tmp/results2.bin
# Combine
vegeta encode /tmp/results1.bin /tmp/results2.bin | vegeta report
```

---

## k6

### Basic Script
```javascript
// basic.js
import http from 'k6/http';
import { check, sleep } from 'k6';

export const options = {
  vus: 10,
  duration: '30s',
};

export default function () {
  const res = http.get('http://localhost:8080/api/users');
  check(res, {
    'status is 200': (r) => r.status === 200,
    'response time < 500ms': (r) => r.timings.duration < 500,
  });
  sleep(1);
}
```
```bash
k6 run basic.js
```

### Ramping VUs (Stages)
```javascript
// ramp.js
import http from 'k6/http';
import { check, sleep } from 'k6';

export const options = {
  stages: [
    { duration: '30s', target: 10 },   // Ramp up to 10 VUs
    { duration: '1m', target: 50 },    // Ramp up to 50 VUs
    { duration: '2m', target: 50 },    // Hold at 50 VUs
    { duration: '30s', target: 0 },    // Ramp down to 0
  ],
};

export default function () {
  const res = http.get('http://localhost:8080/');
  check(res, { 'status 200': (r) => r.status === 200 });
  sleep(1);
}
```

### Scenarios (Advanced Configuration)
```javascript
// scenarios.js
import http from 'k6/http';

export const options = {
  scenarios: {
    // Constant VUs
    constant_load: {
      executor: 'constant-vus',
      vus: 50,
      duration: '5m',
      exec: 'browseAPI',
    },
    // Constant arrival rate (open-loop)
    constant_rate: {
      executor: 'constant-arrival-rate',
      rate: 100,
      timeUnit: '1s',
      duration: '5m',
      preAllocatedVUs: 50,
      maxVUs: 200,
      exec: 'hitEndpoint',
    },
    // Ramping arrival rate
    ramping_rate: {
      executor: 'ramping-arrival-rate',
      startRate: 10,
      timeUnit: '1s',
      stages: [
        { duration: '1m', target: 50 },
        { duration: '2m', target: 100 },
        { duration: '1m', target: 0 },
      ],
      preAllocatedVUs: 50,
      maxVUs: 200,
      exec: 'hitEndpoint',
    },
    // Per-VU iterations
    fixed_iterations: {
      executor: 'per-vu-iterations',
      vus: 10,
      iterations: 100,
      exec: 'browseAPI',
    },
    // Shared iterations
    shared_work: {
      executor: 'shared-iterations',
      vus: 10,
      iterations: 1000,
      exec: 'browseAPI',
    },
  },
};

export function browseAPI() {
  http.get('http://localhost:8080/api/items');
}

export function hitEndpoint() {
  http.get('http://localhost:8080/api/health');
}
```

### Checks and Thresholds
```javascript
import http from 'k6/http';
import { check } from 'k6';

export const options = {
  vus: 50,
  duration: '2m',
  thresholds: {
    http_req_duration: ['p(95)<500', 'p(99)<1000'],   // 95th < 500ms, 99th < 1s
    http_req_failed: ['rate<0.01'],                    // Error rate < 1%
    http_reqs: ['rate>100'],                           // Throughput > 100 req/s
    checks: ['rate>0.99'],                             // 99% of checks pass
  },
};

export default function () {
  const res = http.get('http://localhost:8080/api/users');
  check(res, {
    'status is 200': (r) => r.status === 200,
    'body not empty': (r) => r.body.length > 0,
    'has content-type': (r) => r.headers['Content-Type'] !== undefined,
    'json valid': (r) => JSON.parse(r.body) !== null,
    'has users': (r) => JSON.parse(r.body).length > 0,
  });
}
```

### Custom Metrics
```javascript
import http from 'k6/http';
import { Counter, Trend, Rate, Gauge } from 'k6/metrics';

const errorCount = new Counter('errors');
const responseTime = new Trend('custom_response_time');
const successRate = new Rate('success_rate');
const activeUsers = new Gauge('active_users');

export const options = {
  vus: 20,
  duration: '1m',
  thresholds: {
    custom_response_time: ['p(95)<500'],
    success_rate: ['rate>0.95'],
  },
};

export default function () {
  const res = http.get('http://localhost:8080/');
  responseTime.add(res.timings.duration);
  successRate.add(res.status === 200);
  activeUsers.add(__VU);
  if (res.status !== 200) {
    errorCount.add(1);
  }
}
```

### Groups and Tags
```javascript
import http from 'k6/http';
import { group } from 'k6';

export default function () {
  group('user flow', function () {
    http.get('http://localhost:8080/api/login', { tags: { name: 'login' } });
    http.get('http://localhost:8080/api/dashboard', { tags: { name: 'dashboard' } });
    http.get('http://localhost:8080/api/profile', { tags: { name: 'profile' } });
  });

  group('api calls', function () {
    http.get('http://localhost:8080/api/items', { tags: { name: 'list-items' } });
    http.get('http://localhost:8080/api/items/1', { tags: { name: 'get-item' } });
  });
}
```

### POST/PUT/PATCH with JSON
```javascript
import http from 'k6/http';
import { check } from 'k6';

export default function () {
  const payload = JSON.stringify({
    username: 'testuser',
    email: 'test@example.com',
  });
  const params = {
    headers: { 'Content-Type': 'application/json' },
  };

  const res = http.post('http://localhost:8080/api/users', payload, params);
  check(res, {
    'created': (r) => r.status === 201,
    'has id': (r) => JSON.parse(r.body).id !== undefined,
  });

  // Update
  const id = JSON.parse(res.body).id;
  http.put(`http://localhost:8080/api/users/${id}`,
    JSON.stringify({ email: 'new@example.com' }), params);
}
```

### Auth Patterns
```javascript
import http from 'k6/http';

// Bearer token
export default function () {
  const params = {
    headers: { 'Authorization': 'Bearer ' + __ENV.API_TOKEN },
  };
  http.get('http://localhost:8080/api/protected', params);
}

// Login flow with token capture
export function setup() {
  const res = http.post('http://localhost:8080/api/login',
    JSON.stringify({ username: 'admin', password: 'secret' }),
    { headers: { 'Content-Type': 'application/json' } }
  );
  return { token: JSON.parse(res.body).token };
}

export default function (data) {
  http.get('http://localhost:8080/api/protected', {
    headers: { 'Authorization': 'Bearer ' + data.token },
  });
}
```

### Data Parameterization
```javascript
import http from 'k6/http';
import { SharedArray } from 'k6/data';
import papaparse from 'https://jslib.k6.io/papaparse/5.1.1/index.js';

// JSON data file
const users = new SharedArray('users', function () {
  return JSON.parse(open('./users.json'));
});

// CSV data file
const csvData = new SharedArray('csv', function () {
  return papaparse.parse(open('./data.csv'), { header: true }).data;
});

export default function () {
  const user = users[Math.floor(Math.random() * users.length)];
  http.post('http://localhost:8080/api/login',
    JSON.stringify({ username: user.name, password: user.pass }),
    { headers: { 'Content-Type': 'application/json' } }
  );
}
```

### Environment Variables
```bash
k6 run -e BASE_URL=http://staging:8080 -e API_TOKEN=abc123 script.js
```
```javascript
const baseUrl = __ENV.BASE_URL || 'http://localhost:8080';
http.get(`${baseUrl}/api/users`);
```

### Lifecycle Hooks
```javascript
export function setup() {
  // Runs once before test: seed data, get auth tokens
  const res = http.post('http://localhost:8080/api/setup');
  return { seedId: JSON.parse(res.body).id };
}

export default function (data) {
  // Runs per VU iteration: main test logic
  http.get(`http://localhost:8080/api/items?seed=${data.seedId}`);
}

export function teardown(data) {
  // Runs once after test: cleanup
  http.del(`http://localhost:8080/api/setup/${data.seedId}`);
}
```

### CLI Flags
```bash
k6 run script.js                                              # Basic run
k6 run --vus 50 --duration 2m script.js                       # Override VUs/duration
k6 run --iterations 1000 script.js                            # Fixed iterations
k6 run --http-debug script.js                                 # Log HTTP requests
k6 run --summary-trend-stats='avg,min,med,max,p(90),p(95),p(99)' script.js
```

### Output Options
```bash
k6 run --out json=/tmp/results.json script.js                 # JSON output
k6 run --out csv=/tmp/results.csv script.js                   # CSV output
k6 run --out influxdb=http://localhost:8086/k6 script.js      # InfluxDB
k6 run --out experimental-prometheus-rw script.js             # Prometheus
```

### Common Test Types
```bash
# Smoke test: verify works under minimal load
k6 run --vus 1 --duration 1m script.js

# Load test: normal expected load
k6 run --vus 50 --duration 5m script.js

# Stress test: beyond normal capacity
k6 run --vus 200 --duration 10m script.js

# Spike test: sudden burst (use stages in script)
# Soak test: long duration stability
k6 run --vus 30 --duration 30m script.js
```

---

## artillery

### YAML Configuration
```yaml
# load-test.yml
config:
  target: "http://localhost:8080"
  phases:
    - duration: 60
      arrivalRate: 10
      name: "Warm up"
    - duration: 120
      arrivalRate: 50
      name: "Sustained load"
    - duration: 60
      arrivalRate: 10
      rampTo: 100
      name: "Ramp up"

scenarios:
  - name: "Browse and create"
    flow:
      - get:
          url: "/api/items"
      - think: 1
      - post:
          url: "/api/items"
          json:
            name: "Test Item"
            price: 29.99
      - think: 2
      - get:
          url: "/api/items/{{ id }}"
```

### Capture and Reuse Variables
```yaml
scenarios:
  - name: "Login and use API"
    flow:
      - post:
          url: "/api/login"
          json:
            username: "admin"
            password: "secret"
          capture:
            - json: "$.token"
              as: "authToken"
      - get:
          url: "/api/protected"
          headers:
            Authorization: "Bearer {{ authToken }}"
```

### Assertions with Expect Plugin
```yaml
config:
  target: "http://localhost:8080"
  plugins:
    expect: {}
  phases:
    - duration: 30
      arrivalRate: 5

scenarios:
  - flow:
      - get:
          url: "/api/health"
          expect:
            - statusCode: 200
            - contentType: json
            - hasProperty: status
```

### CSV Payload
```yaml
config:
  target: "http://localhost:8080"
  payload:
    path: "users.csv"
    fields:
      - "username"
      - "password"
  phases:
    - duration: 60
      arrivalRate: 10

scenarios:
  - flow:
      - post:
          url: "/api/login"
          json:
            username: "{{ username }}"
            password: "{{ password }}"
```

### Run Commands
```bash
artillery run load-test.yml                                    # Run test
artillery run --output /tmp/report.json load-test.yml          # Save JSON report
artillery report /tmp/report.json --output /tmp/report.html    # Generate HTML report
artillery quick --count 100 -n 10 http://localhost:8080/       # Quick test (no config)
artillery quick --count 50 -n 5 -m POST \
  -p '{"key":"val"}' -t 'application/json' http://localhost:8080/api/items
```

---

## hurl

### Basic Request and Assertions
```hurl
# health-check.hurl
GET http://localhost:8080/api/health
HTTP 200
[Asserts]
header "Content-Type" contains "json"
jsonpath "$.status" == "ok"
duration < 1000
```

### Request Chaining with Capture
```hurl
# api-flow.hurl
# Step 1: Create user
POST http://localhost:8080/api/users
Content-Type: application/json
{
  "name": "Test User",
  "email": "test@example.com"
}
HTTP 201
[Captures]
user_id: jsonpath "$.id"
[Asserts]
jsonpath "$.name" == "Test User"

# Step 2: Get created user
GET http://localhost:8080/api/users/{{user_id}}
HTTP 200
[Asserts]
jsonpath "$.id" == {{user_id}}
jsonpath "$.email" == "test@example.com"

# Step 3: Update user
PUT http://localhost:8080/api/users/{{user_id}}
Content-Type: application/json
{
  "name": "Updated User"
}
HTTP 200
[Asserts]
jsonpath "$.name" == "Updated User"

# Step 4: Delete user
DELETE http://localhost:8080/api/users/{{user_id}}
HTTP 204
```

### Advanced Assertions
```hurl
GET http://localhost:8080/api/users
HTTP 200
[Asserts]
jsonpath "$.users" count == 10
jsonpath "$.users[0].id" >= 1
jsonpath "$.total" isInteger
jsonpath "$.page" == 1
body contains "users"
header "X-Total-Count" exists
duration < 2000
```

### Variables and Files
```bash
hurl --variable host=localhost --variable port=8080 test.hurl
hurl --variables-file vars.env test.hurl
hurl --test --report-junit /tmp/report.xml tests/*.hurl       # CI mode with JUnit
hurl --test --report-html /tmp/report-dir tests/*.hurl        # HTML report
hurl --very-verbose test.hurl                                  # Full request/response debug
hurl --retry 3 --delay 1000 test.hurl                         # Retry with delay
```

---

## schemathesis

### CLI Usage
```bash
schemathesis run http://localhost:8080/openapi.json                         # Basic run
schemathesis run http://localhost:8080/openapi.json --checks all            # All validation checks
schemathesis run http://localhost:8080/openapi.json --workers 4             # Parallel workers
schemathesis run http://localhost:8080/openapi.json --hypothesis-max-examples=200
schemathesis run http://localhost:8080/openapi.json --stateful=links        # Follow API links
schemathesis run http://localhost:8080/openapi.json --base-url http://staging:8080  # Override base URL
schemathesis run http://localhost:8080/openapi.json --dry-run               # Validate without sending
schemathesis run http://localhost:8080/openapi.json --show-errors-tracebacks
```

### Filtering Endpoints
```bash
schemathesis run http://localhost:8080/openapi.json -E /api/users           # Only /api/users
schemathesis run http://localhost:8080/openapi.json -M POST                 # Only POST methods
schemathesis run http://localhost:8080/openapi.json --tag users             # By tag
```

### Test Types
- **Property-based** - Generates random valid inputs from schema, checks for 5xx errors
- **Stateful** - Follows link relations, tests state transitions (create -> read -> update -> delete)
- **Negative** - Tests invalid inputs (wrong types, missing required fields, boundary values)

### Output
- Reports failing test cases with reproducible curl commands
- Shrinks failing inputs to minimal reproduction
- Supports JUnit output: `--junit-xml=/tmp/report.xml`

---

## Performance Interpretation

### Percentile Guide
| Percentile | Meaning | Use |
|-----------|---------|-----|
| p50 | Median - typical user experience | Baseline performance |
| p90 | 90% of requests faster than this | Most users' experience |
| p95 | Standard SLA target | Common SLA threshold |
| p99 | Tail latency, 1 in 100 requests | Important for UX |
| p99.9 | Worst case for most users | Indicates outlier issues |

### Throughput vs Latency
- Increasing load increases latency (not linear - hockey stick curve)
- Saturation point: where latency begins exponential increase
- Operating at 70-80% of max throughput is typical safe zone
- Beyond saturation: queue buildup, timeouts, cascading failures

### Coordinated Omission
- **Closed-loop** (wrk, ab): waits for response before next request - hides server slowness
- **Open-loop** (wrk2, vegeta, k6 constant-arrival-rate): sends at fixed rate regardless - accurate latency
- Use open-loop for latency measurement, closed-loop for max throughput
- If p99 looks "too good," suspect coordinated omission

### Statistical Best Practices
- Run tests multiple times (3+ runs minimum)
- Ensure enough samples (1000+ requests for stable percentiles)
- Discard warm-up period (first 10-30 seconds)
- Compare same percentile across runs for consistency
- Watch for bimodal distributions (two peaks = two code paths)

---

## Load Testing Patterns

### Spike Test
- Sudden jump from low to high load, then immediate drop
- Tests: auto-scaling, error handling, recovery time
- k6 stages: `[{duration:'10s',target:10},{duration:'1s',target:200},{duration:'10s',target:200},{duration:'1s',target:10},{duration:'30s',target:10}]`

### Soak Test
- Constant moderate load for extended period (30min-2hr)
- Tests: memory leaks, connection pool exhaustion, log rotation, disk usage
- k6: `--vus 30 --duration 30m`

### Stress Test
- Gradually increase load until failure
- Identify breaking point and failure mode
- k6 stages: `[{duration:'2m',target:50},{duration:'2m',target:100},{duration:'2m',target:200},{duration:'2m',target:400}]`

### Breakpoint Test
- Keep increasing until system fails
- Record max throughput and corresponding latency
- Note: what fails first (CPU, memory, connections, DB)

### Capacity Planning
- Run at known load, measure resource usage
- Extrapolate: if 100 req/s = 40% CPU, max ~250 req/s
- Account for headroom: plan for 70% max utilization
- Test with realistic data sizes and query patterns

### Newman / Postman CLI

```bash
# Run Postman collection
newman run collection.json                            # Run collection
newman run collection.json -e env.json                # With environment file
newman run collection.json --folder "Auth Tests"      # Run specific folder
newman run collection.json --reporters cli,json       # Multiple reporters
newman run collection.json --reporter-json-export results.json  # Export JSON results
newman run collection.json --env-var "BASE_URL=http://localhost:3000"  # Override env var
newman run collection.json --iteration-count 5        # Run multiple iterations
```

### GraphQL Testing

```bash
# k6 GraphQL
# import http from 'k6/http';
# export default function() {
#   const query = '{ users { id name email } }';
#   const res = http.post('http://localhost:4000/graphql',
#     JSON.stringify({ query }), { headers: { 'Content-Type': 'application/json' } });
#   check(res, { 'status is 200': (r) => r.status === 200 });
# }

# hurl GraphQL
# POST http://localhost:4000/graphql
# Content-Type: application/json
# { "query": "{ users { id name } }" }
# HTTP 200
# [Asserts]
# jsonpath "$.data.users" count > 0

# vegeta with GraphQL body
echo 'POST http://localhost:4000/graphql' | vegeta attack \
  -body <(echo '{"query":"{ users { id } }"}') \
  -header "Content-Type: application/json" \
  -rate=100 -duration=30s | vegeta report
```

### WebSocket Testing

```bash
# k6 WebSocket
# import ws from 'k6/ws';
# export default function() {
#   const res = ws.connect('ws://localhost:8080/ws', {}, function(socket) {
#     socket.on('open', () => socket.send('hello'));
#     socket.on('message', (msg) => console.log(msg));
#     socket.setTimeout(() => socket.close(), 5000);
#   });
# }

# artillery WebSocket
# config:
#   target: "ws://localhost:8080"
#   phases: [{ duration: 60, arrivalRate: 10 }]
#   engines:
#     ws: {}
# scenarios:
#   - engine: ws
#     flow:
#       - send: "hello"
#       - think: 1
#       - send: "world"
```

### File Upload Testing

```bash
# curl baseline
curl -X POST http://localhost:3000/upload \
  -F "file=@testfile.pdf" \
  -F "description=test upload"

# hey with form data — use curl for upload, hey for concurrency
hey -n 100 -c 10 -m POST \
  -H "Content-Type: application/octet-stream" \
  -D testfile.pdf \
  http://localhost:3000/upload

# k6 file upload
# import http from 'k6/http';
# const file = open('testfile.pdf', 'b');
# export default function() {
#   http.post('http://localhost:3000/upload', { file: http.file(file, 'testfile.pdf') });
# }
```

### CI/CD Integration

```bash
# k6 with thresholds (exit code 99 on failure)
k6 run --out json=results.json script.js
# In script: export const options = { thresholds: { http_req_duration: ['p(95)<500'] } };

# Parse k6 JSON results
jq -r '.metrics.http_req_duration.values | "p50=\(.med)ms p95=\(.["p(95)"])ms p99=\(.["p(99)"])ms"' results.json

# artillery with --ensure (fail if conditions not met)
artillery run --ensure "p99 < 500" loadtest.yml
```

### Combined Workflows

```bash
# vegeta → jq error analysis
echo "GET http://localhost:3000/api/data" | vegeta attack -rate=100 -duration=30s | \
  vegeta encode | jq -r 'select(.code != 200) | "\(.timestamp) \(.code) \(.latency/1e6)ms"'

# Dynamic targets from API
curl -s http://localhost:3000/api/endpoints | \
  jq -r '.[] | "GET http://localhost:3000" + .' | \
  vegeta attack -rate=50 -duration=30s | vegeta report

# hey timing extraction — slowest requests
hey -z 30s -o csv http://localhost:3000/api | awk -F, 'NR>1{print $NF}' | sort -n | tail -20
```

---

## Installation

```bash
# Apache Bench
sudo apt install apache2-utils

# wrk
sudo apt install wrk

# wrk2
git clone https://github.com/giltene/wrk2.git && cd wrk2 && make && sudo cp wrk /usr/local/bin/wrk2

# hey
go install github.com/rakyll/hey@latest
# or: wget https://hey-release.s3.us-east-2.amazonaws.com/hey_linux_amd64 -O /usr/local/bin/hey && chmod +x /usr/local/bin/hey

# vegeta
go install github.com/tsenart/vegeta@latest
# or: wget https://github.com/tsenart/vegeta/releases/latest/download/vegeta_linux_amd64.tar.gz

# k6
sudo snap install k6
# or: sudo gpg -k && sudo gpg --no-default-keyring --keyring /usr/share/keyrings/k6-archive-keyring.gpg --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys C5AD17C747E3415A3642D57D77C6C491D6AC1D68 && echo "deb [signed-by=/usr/share/keyrings/k6-archive-keyring.gpg] https://dl.k6.io/deb stable main" | sudo tee /etc/apt/sources.list.d/k6.list && sudo apt update && sudo apt install k6

# artillery
npm install -g artillery

# hurl
cargo install hurl
# or: sudo apt install hurl  (if available)

# schemathesis
pip install schemathesis
```
