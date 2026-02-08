---
description: "HTTP requests and APIs - API not responding, getting 401 unauthorized, 403 forbidden, SSL certificate error, request timeout, curl GET/POST, downloading files, and authentication (curl, wget, httpie)"
when_to_use: "Use when: API is not responding or returning errors, getting 401 unauthorized or 403 forbidden, SSL certificate verification failing, request timing out, need to debug HTTP response headers, making API requests, downloading files, uploading data, handling authentication, testing webhooks. Triggers: API not responding, 401 unauthorized, 403 forbidden, SSL certificate error, request timeout, API returns error, can't authenticate, curl, http request, api call, download file, post json, http headers, rest api, wget, upload, bearer token, oauth, websocket, graphql, httpie, cookie, form data, multipart upload, http status, redirect, proxy, ssl verify."
when-to-use: "Use when: API is not responding or returning errors, getting 401 unauthorized or 403 forbidden, SSL certificate verification failing, request timing out, need to debug HTTP response headers, making API requests, downloading files, uploading data, handling authentication, testing webhooks. Triggers: API not responding, 401 unauthorized, 403 forbidden, SSL certificate error, request timeout, API returns error, can't authenticate, curl, http request, api call, download file, post json, http headers, rest api, wget, upload, bearer token, oauth, websocket, graphql, httpie, cookie, form data, multipart upload, http status, redirect, proxy, ssl verify."
argument-hint: "[describe the HTTP request you need to make]"
context: inherit
model: inherit
allowed-tools: Skill, Read
---

# HTTP Request Task Router

You have access to the **full conversation context**. Your job is to analyze the user's HTTP/API need and build a well-formed argument for the recipes skill.

**User's request**: $ARGUMENTS

---

## Your Task

HTTP requests often reference URLs, tokens, and data formats from earlier in the conversation. Analyze context to identify:

### 1. Goal
What HTTP operation is needed?
- Simple GET (fetch data, check status)
- POST/PUT/PATCH (create/update resource)
- Download file(s)
- Upload file(s)
- Authentication flow
- Debug connection issue
- WebSocket connection
- GraphQL query

### 2. Target
What endpoint/URL?
- URL from conversation
- API base URL + path
- Local or remote server
- Multiple URLs (batch)

### 3. Authentication Context
- Bearer token discussed?
- API key (header vs query)?
- Basic auth credentials?
- OAuth2 flow needed?
- Client certificates?
- Cookies from prior request?

### 4. Request Details
- Method (GET/POST/PUT/PATCH/DELETE)
- Headers needed
- Request body (JSON, form, multipart)
- Content type
- Query parameters

### 5. Error Context (if debugging)
- HTTP status code received (403, 500, etc.)
- SSL/TLS error
- Timeout
- Redirect loop
- CORS issue discussed

### 6. Output Needs
- Save to file?
- Extract specific field with jq?
- Timing/performance info?
- Headers only?

---

## Goal --> Tool Guide

| Goal | Primary Tool | Argument Prefix |
|------|-------------|----------------|
| API request | curl | "make request to" |
| Download file | curl/wget/aria2c | "download from" |
| Upload file | curl | "upload to" |
| Debug response | curl -v | "debug request to" |
| Auth flow | curl | "authenticate to" |
| WebSocket | websocat/wscat | "websocket connect to" |
| GraphQL | curl | "graphql query to" |
| Pretty API call | httpie/xh | "request with httpie" |
| Timing analysis | curl -w | "measure timing of" |
| Mirror site | wget | "mirror site" |

---

## Argument Format

```
<method> <url> - auth: <type>, body: <data description>, output: <format>, context: <prior findings>
```

### Examples

**Simple GET with auth:**
```
GET https://api.example.com/users - auth: bearer TOKEN_FROM_EARLIER, output: json
```

**POST JSON:**
```
POST https://api.example.com/items - body: {"name": "test", "status": "active"}, auth: api-key in X-API-Key header
```

**Download with resume:**
```
download https://releases.example.com/large-file.tar.gz - resume if interrupted, limit 10MB/s
```

**Debug SSL issue:**
```
debug request to https://api.example.com - getting SSL certificate verify failed
```

**Timing breakdown:**
```
measure timing of https://api.example.com/health - need DNS, connect, TTFB, total
```

---

## Invoke Recipes

Once you've built the argument, use the Skill tool:

```
Skill: task-http-recipes
Args: <your constructed argument>
```

### When to Ask Instead

**Ask the user if:**
- URL/endpoint is not clear from context
- Auth method is ambiguous
- Request body structure is unclear

**Use Read tool first if:**
- Need to check a JSON file for request body
- Need to verify an API spec file
- Need to read a token/config from a file
