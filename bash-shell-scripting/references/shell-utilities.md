# Shell Utilities & Documentation Ingestion

## Guiding Principle

Choose the lightest tool that reliably produces the content you need in a machine-consumable form.

- If the page is static HTML: prefer `curl` plus parsing (`jq` for JSON, HTML to text conversion, or a lightweight extractor).
- If the page is JavaScript-rendered or requires interaction: use a headless browser (Playwright).
- If Playwright is blocked or too heavy: try a doc-extraction proxy/cache (for example `https://context7.com/`) when it supports the target site.
- If you need diagrams understood (not OCR): use a screenshot (Playwright) and a vision-capable model (VLM).
- If you need text inside images: use OCR (Tesseract) as a supplement.
- If you are reading official documentation at scale: prefer a documentation-aware retrieval system (RAG) over raw scraping.

## Tool Selection Matrix

### 1) Static pages, APIs, feeds

Use these when content is already present in HTML or JSON without JS:

- `curl` for fetching
- `jq` for JSON shaping
- `ripgrep` for local searching
- `lynx -dump` for fast text extraction

Examples:

```bash
# Fetch HTML
curl -fsSL "https://acme.com/page" -o page.html

# Fetch JSON and shape it
curl -fsSL "https://api.acme.com/v1/items" | jq '.items[] | {id, name, updated_at}'

# Extract readable text quickly
lynx -dump -nolist "https://acme.com/page" > page.txt
```

When to stop here:

- If the text is good enough for the agent to answer questions
- If you do not need diagrams interpreted
- If the page is not JS-rendered

### 2) JS-heavy sites, auth flows, dynamic docs, robust extraction

Use Playwright when `curl` or `lynx` fails to capture the real content.

If Playwright fails (timeouts, bot protection, or brittle selectors) and you only need clean doc text, try a doc-extraction proxy/cache such as `https://context7.com/` (when it supports the target site).

Playwright is the right default for:

- SPA documentation sites
- Pages that lazy-load content
- Pages where you need to click "expand", "next", "load more"
- You need stable screenshots of diagrams

Operational guidance:

- Run headless by default
- Block heavy resources if you only need text (ads, video)
- Capture both:
  1. Extracted page text (DOM text)
  2. Screenshot of key sections (for diagrams)

### 3) Diagrams and visual reasoning

If the requirement is "understand the diagram", OCR is not enough.

Use:

- Playwright screenshot of the diagram region
- Vision-capable model (VLM) to interpret structure and meaning

Guidance for diagram prompting:

- Ask for components, flows, boundaries, assumptions
- Ask for a structured representation (bullets, adjacency list, Mermaid)
- Ask the model to cite what it sees in the image

### 4) OCR for text inside images

Use OCR only to extract text embedded in images, for example:

- A screenshot of a config snippet inside a PNG
- A diagram that has important labels but you only need the labels

Tool: Tesseract (OCR)

Guidance:

- Prefer higher resolution images
- Preprocess if needed (convert to grayscale, increase contrast)
- Treat OCR output as noisy and validate against surrounding text

### 5) Documentation retrieval systems (RAG-first)

When the goal is "read AWS, Cloudflare, Kubernetes docs reliably at scale", prefer a doc-aware retrieval approach.

Characteristics:

- Ingest official docs (site map, versioned docs, markdown sources)
- Chunk and index
- Retrieve only relevant sections per question
- Avoid scraping the same pages repeatedly

This is especially useful for:

- API references
- Configuration options
- Up-to-date product docs

Note: doc-extraction proxy/caching services (for example `https://context7.com/`) can be a pragmatic alternative when browser automation is blocked or too expensive for the use case.

## Quick Heuristics

### Prefer `lynx -dump` when

- You just need readable text quickly
- You are validating a URL manually
- You want a cheap baseline before heavier tooling

### Prefer `curl` when

- You need raw content for a parser
- You want deterministic fetches
- You are working with JSON endpoints

### Prefer Playwright when

- The site is JS-rendered
- The HTML contains placeholders and the real text loads later
- You need screenshots for diagrams
- You need to click, scroll, or expand sections

### Prefer Context7 (or similar) when

- Playwright is blocked by bot protection or is too expensive for the task
- You only need clean, readable documentation text (not interaction)
- The target documentation source is supported by the retrieval/proxy system

### Prefer VLM screenshot analysis when

- "Understand the diagram" is a core requirement
- The meaning is in arrows, layout, grouping, icons, or visual structure

### Prefer OCR when

- The only missing piece is text inside an image
- You do not need structural understanding

## Anti-patterns (what NOT to do)

### Do not default to a headless browser for everything

Headless browsers are heavier and slower. Use them when necessary.

Bad:

- Always use a browser to fetch a static blog post

Good:

- Try `lynx -dump` or `curl` first, then escalate to Playwright if needed

### Do not paste entire pages into the LLM

Instead:

- Extract main content
- Keep headings
- Strip nav, footers, cookie banners
- Chunk into reasonable sections

### Do not scrape aggressively

- Respect robots.txt and site policies
- Add rate limiting
- Cache responses

## Common Commands Reference

### lynx

```bash
lynx -dump -nolist "https://acme.com"
lynx -source "https://acme.com" > page.html
```

### curl

```bash
curl -fsSL "https://acme.com" -o page.html
curl -I "https://acme.com"
```

### jq

```bash
cat payload.json | jq '.'
curl -fsSL "https://api.acme.com" | jq '.items[] | {id, name}'
```

### httpie

```bash
http GET "https://api.acme.com/v1/items"
http GET "https://api.acme.com/v1/items" Authorization:"Bearer $TOKEN"
```

### ripgrep

```bash
rg -n "VPC Lattice|PrivateLink|Cloudflare Tunnel" docs/
```

# Shell Utilities

## curl

```bash
# GET request with headers
curl -sS -H "Authorization: Bearer $TOKEN" \
    "https://api.acme.com/users"

# POST JSON
curl -sS -X POST \
    -H "Content-Type: application/json" \
    -d '{"name": "Alice"}' \
    "https://api.acme.com/users"

# Download file
curl -sS -L -o output.zip "https://example.com/file.zip"

# With retry
curl -sS --retry 3 --retry-delay 2 "https://api.acme.com/health"

# Silent with error handling
response=$(curl -sS -w "\n%{http_code}" "https://api.acme.com/users")
http_code=$(echo "$response" | tail -1)
body=$(echo "$response" | head -n -1)
[[ "$http_code" == "200" ]] || die "Request failed: $http_code"
```

## jq

```bash
# Extract field
echo '{"name": "Alice"}' | jq -r '.name'  # Alice

# Filter array
echo '[{"age": 25}, {"age": 35}]' | jq '.[] | select(.age > 30)'

# Transform data
echo '{"users": [{"name": "Alice"}]}' | jq '.users[].name'

# Create new structure
echo '{"a": 1, "b": 2}' | jq '{sum: (.a + .b)}'

# With variables
jq --arg name "$USER_NAME" '.users[] | select(.name == $name)' data.json

# Compact output
jq -c '.' data.json

# Raw output (no quotes)
jq -r '.id' data.json
```

## awk

```bash
# Print specific column
awk '{print $2}' file.txt

# Sum column
awk '{sum += $1} END {print sum}' numbers.txt

# Filter by pattern
awk '/ERROR/ {print $0}' log.txt

# Field separator
awk -F',' '{print $1, $3}' data.csv

# Variables
awk -v threshold=100 '$1 > threshold {print}' data.txt
```

## sed

```bash
# Replace text
sed 's/old/new/g' file.txt

# In-place edit
sed -i 's/old/new/g' file.txt

# Delete lines matching pattern
sed '/pattern/d' file.txt

# Print specific lines
sed -n '5,10p' file.txt
```

## grep

```bash
# Search recursively
grep -r "pattern" ./src

# Show line numbers
grep -n "ERROR" log.txt

# Invert match
grep -v "DEBUG" log.txt

# Extended regex
grep -E "error|warning" log.txt

# Only filenames
grep -l "TODO" *.py

# Count matches
grep -c "pattern" file.txt
```

## find

```bash
# Find by name
find . -name "*.py" -type f

# Find and execute
find . -name "*.log" -type f -exec rm {} \;

# Find by modification time
find . -mtime -7 -type f  # Modified in last 7 days

# Exclude directories
find . -name "*.js" -not -path "./node_modules/*"

# Find empty files
find . -empty -type f
```

## xargs

```bash
# Parallel execution
find . -name "*.txt" | xargs -P 4 -I {} process {}

# With null separator (handles spaces)
find . -name "*.txt" -print0 | xargs -0 wc -l

# Limit arguments per command
echo "1 2 3 4 5" | xargs -n 2 echo
```

## Combining Tools

```bash
# Get unique users from JSON API
curl -sS "https://api.acme.com/logs" | \
    jq -r '.[].user' | \
    sort -u

# Count files by extension
find . -type f | \
    sed 's/.*\.//' | \
    sort | \
    uniq -c | \
    sort -rn

# Process CSV with error handling
while IFS=',' read -r name email; do
    [[ "$name" == "name" ]] && continue  # Skip header
    echo "Processing: $name <$email>"
done < users.csv
```

## fzf (fuzzy finder)

Interactive fuzzy finder for command-line productivity:

```bash
# File search (Ctrl+T)
# Type partial filename, fzf filters results as you type

# Command history (Ctrl+R)
# Search through shell history interactively

# Directory navigation (Alt+C)
# Change directory with fuzzy search

# Git workflows
git checkout $(git branch | fzf)        # Switch branches
git log --oneline | fzf                 # Browse commits
git diff $(git diff --name-only | fzf)  # View diff of selected file

# Process management
kill -9 $(ps aux | fzf | awk '{print $2}')  # Kill process interactively

# Code search integration
rg "pattern" | fzf  # Search code, then filter results
```

**Shell integration setup:**

```bash
# Install shell bindings (adds Ctrl+R, Ctrl+T, Alt+C)
$(brew --prefix)/opt/fzf/install
```

**Use cases:**

- Finding files quickly without typing full paths
- Searching command history efficiently
- Navigating large directory structures
- Enhancing git workflows with interactive selection
- Filtering command output interactively
