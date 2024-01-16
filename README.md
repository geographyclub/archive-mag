# archive-mag

Website and scraper for archive.org.

## Get details from archive.org  

Use advanced search.  
```bash
# get details
url='https://archive.org/advancedsearch.php?q=-collection:afghanmediaresourcecenter+AND+year:1990+AND+mediatype:image&fl[]=year&fl[]=description&fl[]=identifier&fl[]=publisher&fl[]=subject&fl[]=title&rows=1000000000&output=json'
curl ${url} > archive_1990_image_details.json
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

## Get images

Get image urls.  
```bash
rm archive_1990_image_urls.txt
cat archive_1990_image_details.json | jq -r '.response.docs[].identifier' | while read identifier; do
  curl "https://archive.org/download/${identifier}" | grep "a href=" | grep -E '\.jpg|\.png' | grep -v "_thumb" | sed -e 's/^.*href="/https:\/\/archive\.org\/download\/'"${identifier}"'\//g' -e 's/">.*$//g' >> archive_1990_image_urls.txt
done
```

Add urls to json.  
```bash
file=archive_1990_image_urls.txt
# Create a temporary file to store the updated JSON
tmp_file=$(mktemp)

# Start the array in the temporary file
echo "[" > "$tmp_file"

# Loop through each object in the JSON array
jq -c '.response.docs[]' archive_1990_image_details.json | while read -r obj; do
  # Extract identifier from the object
  identifier=$(echo "$obj" | jq -r '.identifier')

  # Find the corresponding URL from the text file and take only the first one
  url=$(grep "download/${identifier}/" archive_1990_image_urls.txt | awk '{print $NF; exit}')

  # If URL is not empty, insert the "url" field at the end of each JSON object
  if [ -n "$url" ]; then
    # Construct the updated object with the "url" field
    updated_obj=$(echo "$obj" | jq --arg url "$url" '. + { "url": $url }' | tr -d '\n' | tr -s ' ')

    # Append the updated object to the temporary file
    echo "$updated_obj," >> "$tmp_file"
  fi
done

# Remove the trailing comma and close the array in the temporary file
sed -i '$s/,$/]/' "$tmp_file"

# Create the final output file
mv "$tmp_file" ${file%.*}.json
```
