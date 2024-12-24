---
title: 'Embeddings'
sidebar_label: 'Embeddings'
description: 'Embeddings YAML reference'
---

Embeddings convert text or other data into vector representations for machine learning and natural language processing tasks.

## embeddings

The `embeddings` section in your configuration specifies one or more embedding models for your datasets.

Example:

```yaml
embeddings:
  - from: huggingface:huggingface.co/sentence-transformers/all-MiniLM-L6-v2:latest
    name: text_embedder
    params:
      max_length: '128'
    datasets:
      - my_text_dataset
```

### from

This field specifies the source of your embedding model, whether it's local or remotely accessed. Spice currently supports the following platforms:

- `huggingface:huggingface.co` - Models from Hugging Face
- `file:` - Local file paths
- `openai:` - OpenAI models

#### Examples

**Hugging Face**:

```yaml
embeddings:
  - from: huggingface:huggingface.co/path/to/model-repo
```

**Local File**:

```yaml
embeddings:
  - from: file:path/to/local/file
```

**Open AI**:

```yaml
embeddings:
  - from: openai:o1
```

This field follows the same convention as [`models.from`](./models.md#from).

### name

A unique identifier for this embedding component. This is used when referring to this particular model at other places in your Spicepod.

### files

Optional. A list of files associated with this model. Each file has:

- `path`: The path to the file
- `name`: Optional. A name for the file
- `type`: Optional. The type of the file (automatically determined if not specified)

Follows the same convention as [`models.files`](./models.md#files).

### params

Optional. A map of key-value pairs for additional parameters specific to the embedding model.

#### Examples

- If using OpenAI as the provider for your embedding model, `params` must contain your OpenAI private key.

### dependsOn

Optional. A list of dependencies that must be loaded and available before this embedding model.
