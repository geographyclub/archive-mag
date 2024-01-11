# archive-mag

## Scrape archive.org  
```bash
# first page
json=$(curl "https://archive.org/services/search/v1/scrape?fields=title,date,description,subject&q=date:1990&mediatype:image&count=10000")
cursor_value=$(echo "$json" | jq -r '.cursor')
echo "$json" > archive_1990_images_1.json
# next pages
for a in {2..23}; do
  json=$(curl "https://archive.org/services/search/v1/scrape?fields=title,date,description,subject&q=date:1990&mediatype:image&count=10000&cursor=${cursor_value}")
  cursor_value=$(echo "$json" | jq -r '.cursor')
  echo "$json" > archive_1990_images_${a}.json
done
```
