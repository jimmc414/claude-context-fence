# Tool Removal Analysis - Round 2

**Goal:** Identify 50 tools to remove from the agent toolkit inventory
**Criteria:** Redundant, niche, account-required, platform-specific, declining

---

## Cloud CLIs (Remove 12)

Keep: aws, gcloud, az, flyctl, vercel, wrangler

| # | Tool | Reason |
|---|------|--------|
| 1 | doctl | DigitalOcean declining, niche |
| 2 | netlify | Limited JSON, vercel more popular |
| 3 | railway | Niche platform |
| 4 | render | Niche, limited JSON support |
| 5 | heroku | Platform declining significantly |
| 6 | supabase | Niche, limited JSON |
| 7 | firebase | Limited JSON, use gcloud instead |
| 8 | amplify | Limited JSON, use aws instead |
| 9 | oci | Oracle Cloud is very niche |
| 10 | linode-cli | Niche provider |
| 11 | vultr-cli | Niche provider |
| 12 | hcloud | Niche (EU-focused) |

Also remove: exo, scaleway (very niche) = 14 total

---

## Kubernetes (Remove 10)

Keep: kubectl, helm, kustomize, stern, minikube, kind

| # | Tool | Reason |
|---|------|--------|
| 13 | k3d | kind is more popular |
| 14 | k3s | Edge-focused, niche use case |
| 15 | skaffold | Google tool, less adoption |
| 16 | tilt | Niche dev workflow tool |
| 17 | telepresence | Complex setup required |
| 18 | flux | argocd more popular for GitOps |
| 19 | krew | Plugin manager, not core |
| 20 | kubecolor | Just a wrapper, use kubectl |
| 21 | velero | Specialized backup tool |
| 22 | istio | Very complex service mesh |

Also consider: kubeseal, cert-manager, linkerd, argocd (keep argocd as most popular GitOps)

---

## Serverless (Remove 8)

Keep: serverless, sam, wrangler

| # | Tool | Reason |
|---|------|--------|
| 23 | chalice | Python-specific, AWS-only |
| 24 | zappa | Python-specific, less maintained |
| 25 | func | Azure Functions specific |
| 26 | functions-framework | GCP-specific |
| 27 | deno deploy | Deno-specific |
| 28 | openfaas-cli | Self-hosted, complex setup |
| 29 | fn | Oracle's tool, very niche |
| 30 | kubeless | K8s-specific, declining |

---

## Browser Automation (Remove 6)

Keep: playwright, puppeteer, cypress

| # | Tool | Reason |
|---|------|--------|
| 31 | selenium | Older, playwright better |
| 32 | webdriver | Low-level, use playwright |
| 33 | chromedp | Go-specific |
| 34 | rod | Go-specific, chromedp alternative |
| 35 | splash | Specialized rendering service |
| 36 | browserless | Account/service required |

Also: nightwatch, testcafe, taiko (less popular than cypress)

---

## Monitoring & Alerting (Remove 10)

Keep: prometheus, promtool, vector, journalctl

| # | Tool | Reason |
|---|------|--------|
| 37 | alertmanager | Requires Prometheus server |
| 38 | grafana-cli | Requires Grafana server |
| 39 | logcli | Requires Loki server |
| 40 | telegraf | InfluxDB ecosystem lock-in |
| 41 | influx | Requires InfluxDB server |
| 42 | datadog-agent | Paid account required |
| 43 | newrelic | Paid account required |
| 44 | dd-trace | Paid account (Datadog) |
| 45 | jaeger | Server setup required |
| 46 | uptimerobot | Account required |

Also: pingdom, otel-cli

---

## Additional Removals (4 more to reach 50)

| # | Tool | Category | Reason |
|---|------|----------|--------|
| 47 | lnav | Log Mgmt | TUI-based (missed in round 1) |
| 48 | swagger-codegen | Code Gen | Legacy, openapi-generator is newer |
| 49 | jenkins-cli | CD | Jenkins terrible for agents |
| 50 | travis | CD | Platform declining |

---

## Summary

| Category | Removed | Kept |
|----------|---------|------|
| Cloud CLIs | 14 | 6 |
| Kubernetes | 10 | 6 |
| Serverless | 8 | 3 |
| Browser Automation | 6 | 3 |
| Monitoring | 10 | 4 |
| Other | 4 | - |
| **Total** | **52** | - |

Trimmed to exactly 50 by keeping: hcloud, exo (some EU users), nightwatch
