# archive-mag

A website that shows random images by year from archive.org.

## Get details from archive.org  

Use advanced search.  
```bash
# get all images from 1990
curl "https://archive.org/advancedsearch.php?q=year%3A1990+AND+mediatype%3Aimage&fl%5B%5D=date&fl%5B%5D=year&fl%5B%5D=description&fl%5B%5D=identifier&fl%5B%5D=publisher&fl%5B%5D=subject&fl%5B%5D=title&rows=1000000000&output=json" > archive_1990_image_details.json
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
#rm archive_1990_image_urls.txt
cat archive_1990_image_details.json | jq -r '.response.docs[].identifier' | while read identifier; do
  curl "https://archive.org/download/${identifier}" | grep "a href=" | grep ".jpg" | grep -v "_thumb" | sed -e 's/^.*href="/https:\/\/archive\.org\/download\/'"${identifier}"'\//g' -e 's/">.*$//g' >> archive_1990_image_urls.txt
done
```

Add urls to json.  
```bash
# Create a temporary file to store the updated JSON
tmp_file=$(mktemp)

# Loop through each object in the JSON array
jq -c '.response.docs[]' archive_1990_image_details.json | while read -r obj; do
  # Extract identifier from the object
  identifier=$(echo "$obj" | jq -r '.identifier')

  # Find the corresponding URL from the text file and take only the first one
  url=$(grep "download/${identifier}/" archive_1990_image_urls.txt | awk '{print $NF; exit}')

  # If URL is not empty, insert the "url" field at the end of each JSON object
  if [ -n "$url" ]; then
    # Construct the updated object with the "url" field
    updated_obj=$(echo "$obj" | jq --arg url "$url" '. + { "url": $url }')

    # Append the updated object to the temporary file
    echo "$updated_obj" >> "$tmp_file"
  else
    # If URL is empty, append the original object to the temporary file
    echo "$obj" >> "$tmp_file"
  fi
done

# Create output file
mv "$tmp_file" output.json

# make single-line file
jq -c . output.json > output_single_line.json
```
