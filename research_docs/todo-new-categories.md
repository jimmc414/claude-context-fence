# TODO: New Inventory Categories

Add these categories to `agent-toolkit-comprehensive-inventory.md` in a future session.

---

## 1. Web Scraping / HTML Processing

**Why:** Agents often need to extract structured data from web pages without browser automation.

| Tool | Description | Agent UX | Package |
|------|-------------|----------|---------|
| **pup** | jq-like HTML parser | Excellent | `go install github.com/ericchiang/pup@latest` |
| **htmlq** | jq for HTML (Rust) | Excellent | `cargo install htmlq` |
| **hxselect** | CSS selector extraction | Good | `sudo apt install html-xml-utils` |
| **xidel** | XPath/XQuery/CSS on HTML | Good | manual install |
| **lynx** | Text browser (-dump mode) | Excellent | `sudo apt install lynx` |
| **w3m** | Text browser (-dump mode) | Good | `sudo apt install w3m` |

**Example usage:**
```bash
curl -s https://example.com | pup 'h1 text{}'
curl -s https://example.com | htmlq '.article-title' --text
lynx -dump -nolist https://example.com
```

---

## 2. Template Processing

**Why:** Essential for generating code, configs, IaC files from templates with variable substitution.

| Tool | Description | Agent UX | Package |
|------|-------------|----------|---------|
| **envsubst** | Substitute env vars in text | Excellent | `sudo apt install gettext-base` |
| **gomplate** | Powerful Go templating | Excellent | `go install github.com/hairyhenderson/gomplate/v4/cmd/gomplate@latest` |
| **jinja2-cli** | Jinja2 templates from CLI | Good | `pip install jinja2-cli` |
| **mustache** | Mustache templating | Good | `npm install -g mustache` |
| **tera-cli** | Tera templates (Rust) | Good | `cargo install tera-cli` |
| **mo** | Mustache in pure bash | Excellent | single script |

**Example usage:**
```bash
export NAME="world"; echo 'Hello ${NAME}!' | envsubst
gomplate -d config=config.yaml -f template.tmpl
jinja2 template.j2 data.yaml
```

---

## 3. OCR & Visual Data Extraction

**Why:** Enables agents to extract text from screenshots, images, PDFs; encode/decode QR codes.

| Tool | Description | Agent UX | Package |
|------|-------------|----------|---------|
| **tesseract** | OCR engine | Excellent | `sudo apt install tesseract-ocr` |
| **qrencode** | Generate QR codes | Excellent | `sudo apt install qrencode` |
| **zbarimg** | Decode QR/barcodes | Excellent | `sudo apt install zbar-tools` |
| **img2txt** | Image to ASCII art | Good | `sudo apt install caca-utils` |
| **djvu2txt** | DjVu to text | Good | `sudo apt install djvulibre-bin` |

**Example usage:**
```bash
tesseract screenshot.png stdout
qrencode -o qr.png "https://example.com"
zbarimg qr.png
```

---

## When Adding

1. Add each as a new category (37, 38, 39)
2. Update tool count in header
3. Consider creating corresponding task skills:
   - `task-scrape.md` - HTML/web scraping
   - `task-template.md` - Template processing
   - `task-ocr.md` - Image text extraction

---

---

## Pattern: Convert Existing Skills to Two-Tier

The `task-json` skill has been converted to the two-tier pattern:
- `task-json.md` - Router (context: inherit)
- `task-json-recipes.md` - Recipes (context: fork)

Consider converting these high-value skills next:
1. `task-tabular` - awk/miller recipes are dense
2. `task-git` - git commands benefit from context (branch names, file paths)
3. `task-reverse-engineer` - analysis benefits from knowing what file was discussed

See `skill-pattern-two-tier.md` for implementation guide.

---

**Created:** 2026-02-04
**Status:** Pending
