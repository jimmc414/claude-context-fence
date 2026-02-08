---
description: "HTTP request recipes reference (internal)"
when_to_use: "Receives prepared arguments from task-http with URL, method, and context included"
when-to-use: "Receives prepared arguments from task-http with URL, method, and context included"
context: fork
model: inherit
allowed-tools: Read
---

# HTTP Request Recipes

**Tools:** curl, wget, httpie, xh, aria2c, hurl, websocat, wscat

---

**Task**: $ARGUMENTS

---

> See task-json for advanced jq patterns when processing JSON responses. See task-api-test for load testing and API validation.

## Quick Reference

| Task | Command |
|------|---------|
| GET | `curl -s URL` |
| GET + parse JSON | `curl -s URL \| jq '.'` |
| POST JSON | `curl -s -X POST -H 'Content-Type: application/json' -d '{"k":"v"}' URL` |
| POST form | `curl -s -X POST -d 'key=value' URL` |
| Upload file | `curl -s -X POST -F 'file=@path' URL` |
| Download | `curl -LO URL` |
| Bearer auth | `curl -s -H 'Authorization: Bearer TOKEN' URL` |
| Response code | `curl -s -o /dev/null -w '%{http_code}' URL` |
| Timing | `curl -s -o /dev/null -w 'DNS:%{time_namelookup} Connect:%{time_connect} TTFB:%{time_starttransfer} Total:%{time_total}\n' URL` |
| Verbose debug | `curl -v URL` |
| Headers only | `curl -I URL` |
| Follow redirects | `curl -L URL` |

---

## When to Use What

| Task | Best Tool | Why |
|------|-----------|-----|
| API requests (scriptable) | curl | Universal, pre-installed |
| Human-readable requests | httpie / xh | Colored output, JSON support |
| Download files | wget | Resume, recursive, simple |
| Multi-connection download | aria2c | Parallel connections, fast |
| API test sequences | hurl | Chained HTTP requests in plain text |
| WebSocket testing | websocat | CLI WebSocket client |
| Mock API server | json-server | Instant REST API from JSON file |
| GraphQL queries | curl -X POST | Standard POST with JSON body |

---

## curl GET Requests

### Basic GET
```bash
curl https://api.example.com/users
curl -s https://api.example.com/users              # Silent (no progress bar)
curl -sS https://api.example.com/users             # Silent but show errors
curl --compressed https://api.example.com/users    # Accept gzip/deflate/br encoding
```

### Save Response to File
```bash
curl -s -o response.json https://api.example.com/users     # Custom filename
curl -s -O https://example.com/data.json                   # Keep remote filename
curl -s https://api.example.com/users > output.json        # Redirect stdout
```

### Custom Output Format
```bash
curl -s -w '\nHTTP %{http_code} - %{size_download} bytes in %{time_total}s\n' URL
curl -s -w '%{http_code}' -o /dev/null URL                 # Status code only
curl -s -w '%{content_type}\n' -o /dev/null URL            # Content type only
```

### Query Parameters
```bash
curl -s "https://api.example.com/search?q=term&limit=10"
curl -s -G -d "q=term" -d "limit=10" https://api.example.com/search
curl -s -G --data-urlencode "q=hello world" https://api.example.com/search
```

### Include Response Headers
```bash
curl -s -i URL                        # Headers + body
curl -s -D headers.txt -o body.json URL   # Headers to file, body to file
```

### Range Requests (partial content)
```bash
curl -s --range 0-499 URL             # First 500 bytes
curl -s -H 'Range: bytes=0-499' URL   # Same with explicit header
```

### Conditional Requests
```bash
curl -s -H 'If-Modified-Since: Mon, 01 Jan 2024 00:00:00 GMT' URL
curl -s -H 'If-None-Match: "etag-value-here"' URL
```

### Accept and User-Agent Headers
```bash
curl -s -H 'Accept: application/json' URL
curl -s -H 'Accept: text/csv' URL
curl -s -A 'MyApp/1.0' URL                       # Custom User-Agent
curl -s -A 'Mozilla/5.0 (X11; Linux x86_64)' URL # Browser-like UA
```

---

## curl POST Requests

### POST JSON
```bash
curl -s -X POST \
  -H 'Content-Type: application/json' \
  -d '{"name":"test","value":123}' \
  https://api.example.com/items
```

### POST JSON from File
```bash
curl -s -X POST \
  -H 'Content-Type: application/json' \
  -d @data.json \
  https://api.example.com/items
```

### POST Form Data
```bash
curl -s -X POST \
  -d 'username=admin&password=secret' \
  https://example.com/login

# URL-encode special characters automatically
curl -s -X POST \
  --data-urlencode 'query=hello world & more' \
  --data-urlencode 'filter=status=active' \
  https://api.example.com/search
```

### Multipart Form Upload
```bash
# Single file
curl -s -X POST -F 'file=@document.pdf' https://api.example.com/upload

# File with explicit MIME type
curl -s -X POST -F 'file=@photo.jpg;type=image/jpeg' https://api.example.com/upload

# File + additional fields
curl -s -X POST \
  -F 'file=@report.pdf' \
  -F 'description=Monthly report' \
  -F 'category=finance' \
  https://api.example.com/upload

# Multiple files
curl -s -X POST \
  -F 'files=@file1.txt' \
  -F 'files=@file2.txt' \
  https://api.example.com/upload
```

### Raw Binary Upload
```bash
curl -s -X POST \
  -H 'Content-Type: application/octet-stream' \
  --data-binary @image.png \
  https://api.example.com/upload

# From stdin (streaming)
cat largefile.bin | curl -s -X POST \
  -H 'Content-Type: application/octet-stream' \
  --data-binary @- \
  https://api.example.com/upload

# Chunked transfer encoding
curl -s -X POST \
  -H 'Content-Type: application/json' \
  -H 'Transfer-Encoding: chunked' \
  --data-binary @large-payload.json \
  https://api.example.com/ingest
```

---

## curl Other Methods

### PUT (full replace)
```bash
curl -s -X PUT \
  -H 'Content-Type: application/json' \
  -d '{"name":"updated","status":"active"}' \
  https://api.example.com/items/123
```

### PATCH (partial update)
```bash
curl -s -X PATCH \
  -H 'Content-Type: application/json' \
  -d '{"status":"inactive"}' \
  https://api.example.com/items/123
```

### DELETE
```bash
curl -s -X DELETE https://api.example.com/items/123
curl -s -X DELETE -H 'Authorization: Bearer TOKEN' https://api.example.com/items/123
```

### HEAD (headers only, no body)
```bash
curl -s -I https://example.com/largefile.zip    # Check size before download
```

### OPTIONS (CORS preflight)
```bash
curl -s -X OPTIONS \
  -H 'Origin: https://mysite.com' \
  -H 'Access-Control-Request-Method: POST' \
  -i https://api.example.com/data
```

---

## curl Authentication

### Bearer Token
```bash
curl -s -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIs...' URL
TOKEN=$(cat token.txt)
curl -s -H "Authorization: Bearer $TOKEN" URL
```

### Basic Authentication
```bash
curl -s -u username:password URL
curl -s -u username URL                    # Prompts for password interactively
# Manual base64 encoding
curl -s -H "Authorization: Basic $(echo -n 'user:pass' | base64)" URL
```

### Digest Authentication
```bash
curl -s --digest -u username:password URL
```

### API Key in Header
```bash
curl -s -H 'X-API-Key: your-api-key-here' URL
curl -s -H 'api-key: your-api-key-here' URL
```

### API Key in Query Parameter
```bash
curl -s "https://api.example.com/data?api_key=YOUR_KEY"
curl -s -G -d "api_key=YOUR_KEY" https://api.example.com/data
```

### Client Certificates (mTLS)
```bash
curl -s --cert client.pem --key client-key.pem URL
curl -s --cert client.pem --key client-key.pem --cacert ca-bundle.pem URL
curl -s --cert client.p12 --cert-type P12 URL     # PKCS#12 format
```

### CA Bundle
```bash
curl -s --cacert /path/to/ca-bundle.crt URL
curl -s -k URL                                     # Skip SSL verify (dev only!)
```

### Cookie-Based Authentication
```bash
# Send specific cookies
curl -s -b 'session=abc123; token=xyz789' URL

# Save cookies from response
curl -s -c cookies.txt URL

# Load cookies from file
curl -s -b cookies.txt URL

# Full cookie jar round-trip (load + save)
curl -s -b cookies.txt -c cookies.txt URL

# Login then use session
curl -s -c cookies.txt -d 'user=admin&pass=secret' https://example.com/login
curl -s -b cookies.txt https://example.com/dashboard
```

### OAuth2 Client Credentials Flow
```bash
# Step 1: Get access token
TOKEN=$(curl -s -X POST \
  -d 'grant_type=client_credentials' \
  -d 'client_id=YOUR_CLIENT_ID' \
  -d 'client_secret=YOUR_CLIENT_SECRET' \
  https://auth.example.com/oauth/token | jq -r '.access_token')

# Step 2: Use the token
curl -s -H "Authorization: Bearer $TOKEN" https://api.example.com/resource
```

### Netrc File
```bash
# ~/.netrc format: machine api.example.com login user password pass
curl -s --netrc URL
curl -s --netrc-file /path/to/custom-netrc URL
```

---

## curl Advanced

### Write-Out Format Strings (-w)
```bash
# Common variables
curl -s -o /dev/null -w '%{http_code}\n' URL                # HTTP status
curl -s -o /dev/null -w '%{http_version}\n' URL              # HTTP version (1.1, 2, 3)
curl -s -o /dev/null -w '%{size_download}\n' URL             # Response size bytes
curl -s -o /dev/null -w '%{size_upload}\n' URL               # Upload size bytes
curl -s -o /dev/null -w '%{speed_download}\n' URL            # Download speed bytes/sec
curl -s -o /dev/null -w '%{speed_upload}\n' URL              # Upload speed bytes/sec
curl -s -o /dev/null -w '%{ssl_verify_result}\n' URL         # SSL verify (0=OK)
curl -s -o /dev/null -w '%{redirect_url}\n' URL              # Redirect target
curl -s -o /dev/null -w '%{num_redirects}\n' -L URL          # Number of redirects
curl -s -o /dev/null -w '%{content_type}\n' URL              # Content-Type
curl -s -o /dev/null -w '%{filename_effective}\n' -O URL     # Saved filename

# Complete timing template
curl -s -o /dev/null -w '\
DNS Lookup:    %{time_namelookup}s\n\
TCP Connect:   %{time_connect}s\n\
SSL Handshake: %{time_appconnect}s\n\
Pre-Transfer:  %{time_pretransfer}s\n\
TTFB:          %{time_starttransfer}s\n\
Redirect:      %{time_redirect}s\n\
Total:         %{time_total}s\n\
' URL
```

### Proxy
```bash
curl -s -x http://proxy.example.com:8080 URL                # HTTP proxy
curl -s -x https://proxy.example.com:8443 URL               # HTTPS proxy
curl -s --socks5 proxy.example.com:1080 URL                 # SOCKS5 proxy
curl -s --socks5-hostname proxy.example.com:1080 URL        # SOCKS5 with DNS through proxy
curl -s -x http://user:pass@proxy.example.com:8080 URL      # Proxy with auth
curl -s --proxy-header 'X-Forwarded-For: 1.2.3.4' -x http://proxy:8080 URL
```

### HTTP Versions
```bash
curl -s --http1.1 URL                   # Force HTTP/1.1
curl -s --http2 URL                     # Use HTTP/2 (with TLS upgrade)
curl -s --http2-prior-knowledge URL     # HTTP/2 without upgrade (h2c)
curl -s --http3 URL                     # Use HTTP/3 (QUIC)
```

### Retry and Resilience
```bash
curl -s --retry 3 URL                                        # Retry 3 times on transient errors
curl -s --retry 5 --retry-delay 2 URL                        # 2s between retries
curl -s --retry 3 --retry-max-time 60 URL                    # Max total retry time
curl -s --retry 3 --retry-all-errors URL                     # Retry on all errors (not just transient)
curl -s --max-time 30 --connect-timeout 10 URL               # Timeouts
curl -s --keepalive-time 60 URL                              # TCP keepalive interval
```

### Rate Limiting
```bash
curl -s --limit-rate 1M URL             # Limit to 1 MB/s
curl -s --limit-rate 500k URL           # Limit to 500 KB/s
```

### DNS and Connection Override
```bash
# Bypass DNS: resolve host to specific IP
curl -s --resolve api.example.com:443:127.0.0.1 https://api.example.com/test

# Unix domain socket (e.g., Docker API)
curl -s --unix-socket /var/run/docker.sock http://localhost/containers/json

# Local port binding
curl -s --local-port 8000-9000 URL
```

### Parallel Requests (curl 7.66+)
```bash
curl -s --parallel -Z \
  -o out1.json 'https://api.example.com/endpoint1' \
  -o out2.json 'https://api.example.com/endpoint2' \
  -o out3.json 'https://api.example.com/endpoint3'
```

### Config File
```bash
# curl.cfg contents: one flag per line
# -s
# -H "Authorization: Bearer TOKEN"
# -H "Accept: application/json"
curl -K curl.cfg URL
```

### Progress
```bash
curl --no-progress-meter URL            # Hide progress but show output (curl 7.67+)
curl -# URL                             # Progress bar instead of meter
```

---

## wget

### Basic Downloads
```bash
wget https://example.com/file.zip                     # Download to current dir
wget -O output.zip https://example.com/file.zip       # Custom output name
wget -q https://example.com/file.zip                  # Quiet mode
wget -q -O - URL                                      # Output to stdout (like curl)
```

### Resume and Retry
```bash
wget -c https://example.com/largefile.iso              # Resume interrupted download
wget --tries=5 --waitretry=2 URL                       # Retry with backoff
```

### Rate Limiting and Throttle
```bash
wget --limit-rate=1m URL                               # 1 MB/s max
wget --limit-rate=500k URL                             # 500 KB/s max
```

### Recursive Download
```bash
wget -r --level=2 --no-parent https://example.com/docs/
wget -r -l 1 --no-parent -A '*.pdf' https://example.com/papers/
```

### Mirror a Website
```bash
wget --mirror --convert-links --page-requisites \
  --no-parent --directory-prefix=./mirror \
  https://example.com/
# Equivalent shorthand: wget -m -k -p -np -P ./mirror URL
```

### Spider (check links without downloading)
```bash
wget --spider --recursive --level=2 https://example.com/ 2>&1 | grep 'broken'
wget --spider -q URL && echo "exists" || echo "missing"
```

### Timestamping (download only if newer)
```bash
wget -N https://example.com/data.csv                   # Only if remote is newer
```

### Input File (batch download)
```bash
wget -i urls.txt                                       # Download all URLs from file
wget -i urls.txt -P /tmp/downloads/ -q                 # Quiet, to specific dir
```

### Accept/Reject Patterns
```bash
wget -r --accept '*.pdf,*.doc' --reject '*.html' URL
wget -r -A 'report-*.pdf' URL                          # Shorthand accept
wget -r -R 'index.html*' URL                           # Shorthand reject
```

### Background and Logging
```bash
wget -b -o download.log URL                            # Background with log
tail -f download.log                                   # Monitor progress
```

### Authentication
```bash
wget --user=admin --password=secret URL
wget --header='Authorization: Bearer TOKEN' URL
wget --no-check-certificate URL                        # Skip SSL verify (dev only!)
```

### Convert Links for Offline Browsing
```bash
wget -r -k -p https://example.com/docs/               # -k converts links, -p gets assets
```

### Wait Between Requests
```bash
wget -r --wait=1 --random-wait https://example.com/    # Polite crawling
```

---

## httpie / xh

### GET Requests
```bash
http https://api.example.com/users                     # httpie GET
xh https://api.example.com/users                       # xh GET (faster, compatible)
http https://api.example.com/users Accept:application/json  # Custom header
```

### POST JSON
```bash
http POST https://api.example.com/items name=test count:=5 active:=true
xh POST https://api.example.com/items name=test count:=5
# := for non-string values (numbers, booleans, arrays, objects)
```

### Authentication
```bash
http -a user:pass https://api.example.com/secure       # Basic auth
http https://api.example.com/data Authorization:'Bearer TOKEN'
```

### Download
```bash
http --download https://example.com/file.zip
xh --download https://example.com/file.zip
```

### Sessions (persistent cookies/auth)
```bash
http --session=myapi POST https://api.example.com/login user=admin pass=secret
http --session=myapi https://api.example.com/dashboard    # Reuses session cookies
```

### Form Data
```bash
http -f POST https://example.com/login username=admin password=secret
```

### File Upload
```bash
http -f POST https://api.example.com/upload file@document.pdf
```

### Verbose and Output Control
```bash
http -v https://api.example.com/users                  # Show request + response
http --pretty=all URL                                  # Colors + formatting
http --pretty=none URL                                 # Raw output (for piping)
http -h URL                                            # Headers only
http -b URL                                            # Body only
```

### Raw Body from Stdin
```bash
echo '{"custom":"payload"}' | http POST https://api.example.com/data
cat data.json | http PUT https://api.example.com/items/1
```

---

## Downloading Files

### Single File Comparison
```bash
curl -LO https://example.com/file.tar.gz              # curl: follow redirects + remote name
wget https://example.com/file.tar.gz                   # wget: simpler syntax
```

### Progress Bar
```bash
curl -# -O URL                                         # curl with progress bar
wget --show-progress -q URL                            # wget minimal progress
```

### Custom Filename
```bash
curl -L -o myfile.tar.gz URL
wget -O myfile.tar.gz URL
```

### Resume Interrupted Download
```bash
curl -C - -O URL                                       # curl resume
wget -c URL                                            # wget resume
```

### Parallel Downloads with aria2c
```bash
# Single file, 16 connections
aria2c -x 16 -s 16 https://example.com/largefile.iso

# Multiple files concurrently
aria2c -j 5 -i urls.txt                               # 5 concurrent files from list

# Per-server connection limit
aria2c --max-connection-per-server=8 URL

# Metalink download
aria2c https://example.com/file.metalink

# BitTorrent
aria2c https://example.com/file.torrent
aria2c 'magnet:?xt=urn:btih:...'

# Custom output
aria2c -o custom-name.zip -d /tmp/downloads URL
```

### Integrity Verification After Download
```bash
# Download then verify SHA256
curl -LO URL && echo "expected_sha256  filename" | sha256sum -c -

# Download then verify MD5
wget URL && md5sum -c checksums.md5

# Compare hash inline
EXPECTED="abc123def456..."
ACTUAL=$(sha256sum downloaded-file | cut -d' ' -f1)
[ "$EXPECTED" = "$ACTUAL" ] && echo "OK" || echo "MISMATCH"
```

### Conditional Download
```bash
curl -s -z existing-file.txt URL -o existing-file.txt  # Only if modified since file's date
wget -N URL                                            # Timestamping
```

### Large File Strategies
```bash
# Background with nohup
nohup curl -LO URL > download.log 2>&1 &

# Split download into parts (manual)
curl --range 0-999999999 -o part1 URL
curl --range 1000000000-1999999999 -o part2 URL
cat part1 part2 > complete-file
```

---

## WebSocket

### websocat
```bash
# Connect to WebSocket
websocat ws://localhost:8080/ws

# Secure WebSocket
websocat wss://echo.websocket.org

# Send message and receive (one-shot)
echo '{"type":"ping"}' | websocat ws://localhost:8080/ws

# Binary mode
websocat -b ws://localhost:8080/ws

# Auto-reconnect
websocat --autoreconnect ws://localhost:8080/ws

# Custom headers
websocat --header 'Authorization: Bearer TOKEN' ws://localhost:8080/ws

# Set origin
websocat --origin https://example.com ws://localhost:8080/ws

# Connect stdin/stdout to WebSocket (bidirectional pipe)
websocat -t ws://localhost:8080/ws
```

### wscat
```bash
# Connect
wscat -c ws://localhost:8080/ws

# With custom headers
wscat -c ws://localhost:8080/ws -H 'Authorization: Bearer TOKEN'

# Wait timeout (seconds)
wscat -c ws://localhost:8080/ws -w 30

# Execute single message then close
wscat -c ws://localhost:8080/ws -x '{"type":"hello"}'

# Slash commands inside wscat: /close, /ping, /pong
```

### curl WebSocket (curl 7.86+)
```bash
curl --include \
  --no-buffer \
  --http1.1 \
  -H 'Upgrade: websocket' \
  -H 'Connection: Upgrade' \
  -H 'Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==' \
  -H 'Sec-WebSocket-Version: 13' \
  https://echo.websocket.org
```

---

## GraphQL

### Basic Query
```bash
curl -s -X POST \
  -H 'Content-Type: application/json' \
  -d '{"query": "{ users { id name email } }"}' \
  https://api.example.com/graphql
```

### Query with Variables
```bash
curl -s -X POST \
  -H 'Content-Type: application/json' \
  -d '{
    "query": "query GetUser($id: ID!) { user(id: $id) { name email } }",
    "variables": {"id": "123"}
  }' \
  https://api.example.com/graphql
```

### Mutation
```bash
curl -s -X POST \
  -H 'Content-Type: application/json' \
  -H 'Authorization: Bearer TOKEN' \
  -d '{
    "query": "mutation CreateUser($input: UserInput!) { createUser(input: $input) { id name } }",
    "variables": {"input": {"name": "Alice", "email": "alice@example.com"}}
  }' \
  https://api.example.com/graphql
```

### Named Operations
```bash
curl -s -X POST \
  -H 'Content-Type: application/json' \
  -d '{
    "query": "query ListUsers { users { id name } } query ListPosts { posts { id title } }",
    "operationName": "ListUsers"
  }' \
  https://api.example.com/graphql
```

### Introspection Query
```bash
curl -s -X POST \
  -H 'Content-Type: application/json' \
  -d '{"query": "{ __schema { types { name kind description } } }"}' \
  https://api.example.com/graphql | jq '.data.__schema.types[] | select(.kind == "OBJECT")'
```

### Fragment Usage
```bash
curl -s -X POST \
  -H 'Content-Type: application/json' \
  -d '{
    "query": "fragment UserFields on User { id name email } query { users { ...UserFields } }"
  }' \
  https://api.example.com/graphql
```

### GraphQL File Upload (multipart spec)
```bash
curl -s -X POST \
  -F 'operations={"query":"mutation($file: Upload!) { uploadFile(file: $file) { url } }","variables":{"file":null}}' \
  -F 'map={"0":["variables.file"]}' \
  -F '0=@photo.jpg' \
  https://api.example.com/graphql
```

### Query from File
```bash
# query.graphql contains the query text
curl -s -X POST \
  -H 'Content-Type: application/json' \
  -d "{\"query\": $(jq -Rs '.' query.graphql)}" \
  https://api.example.com/graphql
```

---

## API Patterns

### Pagination: Link Header (next URL)
```bash
URL="https://api.example.com/items?per_page=100"
while [ -n "$URL" ]; do
  response=$(curl -sS -D /tmp/headers.txt "$URL")
  echo "$response" | jq '.[]'
  URL=$(grep -i '^link:' /tmp/headers.txt | sed -n 's/.*<\([^>]*\)>; rel="next".*/\1/p')
done
```

### Pagination: Offset-Based
```bash
offset=0
limit=100
while true; do
  result=$(curl -s "https://api.example.com/items?offset=$offset&limit=$limit")
  count=$(echo "$result" | jq '.items | length')
  echo "$result" | jq '.items[]'
  [ "$count" -lt "$limit" ] && break
  offset=$((offset + limit))
done
```

### Pagination: Cursor-Based
```bash
cursor=""
while true; do
  if [ -z "$cursor" ]; then
    result=$(curl -s "https://api.example.com/items?limit=100")
  else
    result=$(curl -s "https://api.example.com/items?limit=100&cursor=$cursor")
  fi
  echo "$result" | jq '.items[]'
  cursor=$(echo "$result" | jq -r '.next_cursor // empty')
  [ -z "$cursor" ] && break
done
```

### Retry with Exponential Backoff
```bash
max_retries=5
retry=0
delay=1
while [ $retry -lt $max_retries ]; do
  response=$(curl -s -w '\n%{http_code}' URL)
  http_code=$(echo "$response" | tail -1)
  body=$(echo "$response" | sed '$d')
  if [ "$http_code" -ge 200 ] && [ "$http_code" -lt 300 ]; then
    echo "$body"
    break
  fi
  echo "Retry $((retry+1))/$max_retries (HTTP $http_code), waiting ${delay}s..." >&2
  sleep $delay
  retry=$((retry + 1))
  delay=$((delay * 2))
done
```

### OAuth2 Client Credentials Full Flow
```bash
# Get token
TOKEN_RESPONSE=$(curl -s -X POST \
  -d 'grant_type=client_credentials' \
  -d "client_id=$CLIENT_ID" \
  -d "client_secret=$CLIENT_SECRET" \
  -d 'scope=read write' \
  https://auth.example.com/oauth/token)

ACCESS_TOKEN=$(echo "$TOKEN_RESPONSE" | jq -r '.access_token')
EXPIRES_IN=$(echo "$TOKEN_RESPONSE" | jq -r '.expires_in')
echo "Token expires in ${EXPIRES_IN}s"

# Use token
curl -s -H "Authorization: Bearer $ACCESS_TOKEN" https://api.example.com/resource

# Refresh (if refresh_token provided)
REFRESH_TOKEN=$(echo "$TOKEN_RESPONSE" | jq -r '.refresh_token // empty')
if [ -n "$REFRESH_TOKEN" ]; then
  curl -s -X POST \
    -d 'grant_type=refresh_token' \
    -d "refresh_token=$REFRESH_TOKEN" \
    -d "client_id=$CLIENT_ID" \
    https://auth.example.com/oauth/token
fi
```

### JWT Decode Without Library
```bash
# Decode JWT payload (middle segment)
echo 'eyJhbGci...' | cut -d. -f2 | base64 -d 2>/dev/null | jq '.'

# From variable
echo "$JWT_TOKEN" | cut -d. -f2 | tr '_-' '/+' | base64 -d 2>/dev/null | jq '.'

# Decode header
echo "$JWT_TOKEN" | cut -d. -f1 | base64 -d 2>/dev/null | jq '.'
```

### Batch Requests Pattern
```bash
# Sequential with results
for id in 1 2 3 4 5; do
  curl -s "https://api.example.com/items/$id" | jq '{id: .id, name: .name}'
done | jq -s '.'

# Parallel with xargs
echo -e "1\n2\n3\n4\n5" | xargs -P 5 -I {} \
  curl -s "https://api.example.com/items/{}" -o "item-{}.json"
```

### Conditional Requests (ETag / Last-Modified)
```bash
# First request: save ETag
ETAG=$(curl -sI URL | grep -i 'etag:' | tr -d '\r' | cut -d' ' -f2)

# Subsequent: conditional fetch (304 if unchanged)
curl -s -H "If-None-Match: $ETAG" -w '%{http_code}' -o /dev/null URL

# With Last-Modified
LAST_MOD=$(curl -sI URL | grep -i 'last-modified:' | cut -d: -f2- | tr -d '\r')
curl -s -H "If-Modified-Since: $LAST_MOD" -w '%{http_code}' -o /dev/null URL
```

### Rate Limit Handling
```bash
response=$(curl -sS -D /tmp/rate-headers.txt URL)
remaining=$(grep -i 'x-ratelimit-remaining:' /tmp/rate-headers.txt | tr -d '\r' | awk '{print $2}')
reset=$(grep -i 'x-ratelimit-reset:' /tmp/rate-headers.txt | tr -d '\r' | awk '{print $2}')

if [ "$remaining" = "0" ]; then
  now=$(date +%s)
  wait_time=$((reset - now))
  echo "Rate limited. Waiting ${wait_time}s..." >&2
  sleep "$wait_time"
fi
```

### Webhook Testing
```bash
# Terminal 1: Listen for incoming webhook
nc -l -p 9000

# Terminal 2: Send test webhook
curl -s -X POST \
  -H 'Content-Type: application/json' \
  -d '{"event":"test","payload":{"id":123}}' \
  http://localhost:9000/webhook

# Listen and respond with 200
while true; do
  echo -e "HTTP/1.1 200 OK\r\nContent-Length: 2\r\n\r\nOK" | nc -l -p 9000
done
```

---

## Debugging HTTP

### Verbose Output Walkthrough
```bash
curl -v https://api.example.com/data 2>&1
# * = connection info (DNS, TCP, SSL)
# > = request sent (method, headers)
# < = response received (status, headers)
# { } = data sent/received (with --trace)
```

### Timing Breakdown Interpretation
```bash
curl -s -o /dev/null -w '\
DNS Lookup:    %{time_namelookup}s\n\
TCP Connect:   %{time_connect}s\n\
SSL Handshake: %{time_appconnect}s\n\
Server Think:  calc(TTFB - SSL)\n\
TTFB:          %{time_starttransfer}s\n\
Download:      calc(Total - TTFB)\n\
Total:         %{time_total}s\n\
' URL
# If SSL is slow: check cipher negotiation
# If TTFB is slow: server-side issue
# If Download is slow: bandwidth/payload size
```

### SSL/TLS Debugging
```bash
# Full SSL trace
curl -v --trace-ascii /tmp/ssl-trace.txt https://api.example.com 2>&1

# OpenSSL direct inspection
openssl s_client -connect api.example.com:443 -servername api.example.com </dev/null 2>/dev/null

# View certificate chain
openssl s_client -connect api.example.com:443 -showcerts </dev/null 2>/dev/null | \
  openssl x509 -noout -subject -issuer -dates

# Check specific TLS version
curl -s --tlsv1.2 --tls-max 1.2 URL     # Force TLS 1.2
curl -s --tlsv1.3 URL                     # Require TLS 1.3

# Show negotiated cipher
curl -v URL 2>&1 | grep 'SSL connection'
```

### Redirect Chain Tracing
```bash
curl -v -L URL 2>&1 | grep -E '< HTTP|< location:|> GET|> Host:'
curl -s -L -w '%{num_redirects} redirects\n%{url_effective}\n' -o /dev/null URL
```

### Connection Reuse Verification
```bash
curl -v URL URL 2>&1 | grep -E 'Re-using|Connected to|Closing'
```

### curl Error Code Reference
```bash
# Common exit codes:
# 6  = Could not resolve host (DNS failure)
# 7  = Failed to connect (connection refused/timeout)
# 22 = HTTP error (with --fail)
# 28 = Operation timeout (--max-time exceeded)
# 35 = SSL/TLS handshake failed
# 47 = Too many redirects (--max-redirs exceeded)
# 52 = Server returned nothing (empty response)
# 56 = Failure receiving data (connection reset)

# Use --fail to make curl return error on HTTP 4xx/5xx
curl -sf URL || echo "Request failed with exit code $?"
curl -sf -w '%{http_code}' URL || echo "HTTP error"
```

### DNS Debugging
```bash
# Check DNS resolution separately
dig api.example.com +short
nslookup api.example.com

# Force specific DNS resolution in curl
curl -s --resolve api.example.com:443:1.2.3.4 https://api.example.com/test
```

---

## hurl (Chained HTTP Requests)

### Basic Request with Assertions
```bash
# test.hurl
cat <<'HURL' > test.hurl
GET https://api.example.com/health
HTTP 200
[Asserts]
header "Content-Type" contains "json"
jsonpath "$.status" == "ok"
HURL
hurl test.hurl
```

### Chained Requests with Capture
```bash
cat <<'HURL' > chain.hurl
# Login and capture token
POST https://api.example.com/login
Content-Type: application/json
{"username": "admin", "password": "secret"}
HTTP 200
[Captures]
token: jsonpath "$.access_token"

# Use captured token
GET https://api.example.com/profile
Authorization: Bearer {{token}}
HTTP 200
[Asserts]
jsonpath "$.name" exists
HURL
hurl chain.hurl
```

### Test Mode
```bash
hurl --test test.hurl                   # Run assertions, report pass/fail
hurl --test --very-verbose test.hurl    # Detailed output on failure
```

### Variables
```bash
hurl --variable host=https://staging.example.com \
     --variable token=abc123 \
     test.hurl

# In hurl file: GET {{host}}/api/data
#               Authorization: Bearer {{token}}
```

### Multiple Assertions
```bash
cat <<'HURL' > validate.hurl
GET https://api.example.com/users
HTTP 200
[Asserts]
status == 200
header "Content-Type" == "application/json; charset=utf-8"
jsonpath "$.users" count == 10
jsonpath "$.users[0].id" isInteger
jsonpath "$.users[0].email" matches "^[^@]+@[^@]+$"
body contains "admin"
duration < 2000
HURL
hurl --test validate.hurl
```

### CI Integration
```bash
hurl --test --report-junit report.xml tests/*.hurl
hurl --test --report-html report/ tests/*.hurl
```

### Server-Sent Events (SSE)

```bash
# Consume SSE stream
curl -sN http://localhost:3000/events           # -N disables buffering

# Process SSE events line by line
curl -sN http://localhost:3000/events | while read line; do
  echo "$line" | grep "^data:" | cut -d: -f2-
done

# SSE with timeout
curl -sN --max-time 60 http://localhost:3000/events | grep "^data:"
```

### curl --json Shorthand

```bash
# --json is equivalent to -H "Content-Type: application/json" -H "Accept: application/json" -d
curl --json '{"name":"test"}' http://localhost:3000/api/items
curl --json @data.json http://localhost:3000/api/items     # From file
curl --json '{"status":"done"}' -X PATCH http://localhost:3000/api/items/1
```

### gRPC with grpcurl

```bash
# List services
grpcurl -plaintext localhost:50051 list
grpcurl -plaintext localhost:50051 list mypackage.MyService

# Describe service
grpcurl -plaintext localhost:50051 describe mypackage.MyService

# Call RPC method
grpcurl -plaintext -d '{"name": "world"}' localhost:50051 mypackage.Greeter/SayHello

# With TLS
grpcurl -cacert ca.pem -cert client.pem -key client-key.pem \
  localhost:50051 mypackage.MyService/Method
```

### Combined Workflows

```bash
# Tee response for inspection and processing
curl -s http://api.example.com/data | tee response.json | jq '.items | length'

# Timing pipeline
curl -s -o /dev/null -w "DNS: %{time_namelookup}s\nConnect: %{time_connect}s\nTTFB: %{time_starttransfer}s\nTotal: %{time_total}s\n" http://example.com

# Retry with backoff
for i in 1 2 4 8 16; do curl -sf http://api.example.com/health && break; sleep $i; done
```

---

## Installation

```bash
# curl and wget (usually pre-installed)
sudo apt install curl wget

# httpie
sudo apt install httpie
# or: pip install httpie

# xh (fast httpie-compatible)
# https://github.com/ducaale/xh/releases
# or: cargo install xh

# aria2c (parallel downloader)
sudo apt install aria2

# hurl (HTTP testing)
# https://hurl.dev/docs/installation.html
curl --proto '=https' --tlsv1.2 -sSf https://hurl.dev/install.sh | sudo bash

# websocat (WebSocket CLI)
# https://github.com/vi/websocat/releases
cargo install websocat

# wscat (Node WebSocket CLI)
npm install -g wscat
```
