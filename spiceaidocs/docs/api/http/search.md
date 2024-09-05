---
title: 'POST /v1/search'
sidebar_label: 'POST /v1/search'
description: ''
sidebar_position: 7
pagination_prev: null
pagination_next: null
---

Performs a basic vector similarity search from one or more dataset(s). 

Request Body
 - `datasets` (array of strings): Dataset component names to perform similarity search against. Each dataset is expected to have one and only one column augmented with an embedding. 
 - `text` (string): Query plaintext used to retrieve similar rows from the underlying datasets listed in the `from` request key.
 - `limit` (integer): The number of rows to return, per `from` dataset. Default: 3.
 - `where` (string): An SQL filter predicate to apply within the search.
 - `additional_columns` (array of strings): Additional columns, from the datasets, to return in the response (under `.matches[*].metadata`).
 
#### Example

Spicepod
```yaml
embeddings:
  - name: embedding_maker
    from: openai
datasets:
  - name: app_messages
    from: file://my.csv
    embeddings:
      - column: document_text
        use: embedding_maker
```

Request
```shell
curl -XPOST http://localhost:3000/v1/search \
  -d '{
    "datasets": ["app_messages"],
    "text": "Tokyo plane tickets",
    "where": 'user=1234321',
    "additional_columns": ["timestamp"],
    "limit": 3
  }'
```

Response
```json
{
  "matches": [{
    "value": "I booked use some tickets",
    "dataset": "app_messages",
    "primary_key": {"id": "6fd5a215-0881-421d-ace0-b293b83452b5"},
    "metadata": {"timestamp": 1724716542}
   },
   {
    "value": "direct to Narata",
    "dataset": "app_messages",
    "primary_key": {"id": "8a25595f-99fb-4404-8c82-e1046d8f4c4b"},
    "metadata": {"timestamp": 1724715881}
   },
   {
    "value": "Yes, we're sitting together",
    "dataset": "app_messages",
    "primary_key": {"id": "8421ed84-b86d-4b10-b4da-7a432e8912c0"},
    "metadata": {"timestamp": 1724716123}
   }],
  "duration_ms": 42,
}
```