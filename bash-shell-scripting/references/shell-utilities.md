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

