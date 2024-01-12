# archive-mag

A website that shows random images from the past.

## Scrape archive.org  

Use advanced search.  
```bash
curl "https://archive.org/advancedsearch.php?q=year%3A1990+AND+mediatype%3Aimage&fl%5B%5D=date&fl%5B%5D=year&fl%5B%5D=description&fl%5B%5D=identifier&fl%5B%5D=publisher&fl%5B%5D=subject&fl%5B%5D=title&rows=1000000000&output=json" > archive_1990_images.json
```

Use scraping service.  
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
