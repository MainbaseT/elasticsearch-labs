# Ecommerce search reranking with Jev

Runnable companion to the Elastic blog post
[Using Jev to rerank Elasticsearch results](https://www.elastic.co/search-labs/blog/ecommerce-search-reranking-llm-alternative-jev).

[`ecommerce-search-reranking-llm-alternative-jev.ipynb`](ecommerce-search-reranking-llm-alternative-jev.ipynb)
builds the post's architecture end to end against your own deployment:

```text
query
  -> Elasticsearch retrieves a bounded candidate set
  -> Jev judges each query-product pair
  -> application code calculates a ranking score
  -> candidates are sorted and returned
```

The notebook indexes a small product catalog with `semantic_text`, runs two
Elasticsearch baselines (BM25 and hybrid RRF), reranks the same fixed candidate
sets with four Jev strategies, and scores every ranking with nDCG@10 and Exact
MRR against human relevance judgments.

## Prerequisites

- **Elasticsearch 9.x** — [Elastic Cloud](https://cloud.elastic.co/registration?utm_source=github&utm_content=elasticsearch-labs-samples),
  Serverless, or self-managed, with an API key that can create, write, read,
  and delete one index.
- **A text embedding inference endpoint** for `semantic_text`. The notebook
  defaults to `.jina-embeddings-v5-text-small` and prints the endpoints your
  deployment actually has if that one is missing.
- **A TypeSafe API key** from [typesafe.ai](https://typesafe.ai).
- **Python 3.10+** with Jupyter, VS Code, or Colab.

## Run locally

```bash
pip install -r requirements.txt
jupyter lab ecommerce-search-reranking-llm-alternative-jev.ipynb
```

Credentials are read from `ELASTICSEARCH_URL`, `ELASTICSEARCH_API_KEY`, and
`TYPESAFE_API_KEY` when set, and prompted for otherwise. Nothing is written to
disk except the model-response cache.

## What it costs

The bundled sample holds 10 queries and 231 query-product pairs. Jev is called
once per pair, so a full pass is 231 requests; the optional direct `Score` pass
adds another 231. Every response is cached to `.jev-notebook-cache.json`, so
re-running a cell never repeats a call. Set `QUERY_LIMIT = 3` in the
configuration cell for a cheaper first run.

## The data

[`data/esci-sample-10-queries.json`](data/esci-sample-10-queries.json) is a
slice of the [Amazon Shopping Queries Dataset
(ESCI)](https://github.com/amazon-science/esci-data): ten US-English queries
from the 50-query development sample used in the post, with their complete
official candidate lists and the human ESCI labels (Exact, Substitute,
Complement, Irrelevant).

The ten were selected by label diversity — most distinct labels, then highest
label entropy, then smallest candidate set, then query ID. That rule never looks
at any system's ranking quality, but it does favor queries whose judgments
disagree, which is where rerankers can separate.

**Ten queries is a demonstration, not a benchmark.** A single query moves the
mean by roughly a tenth of a point. The post's results come from a 250-query
held-out test set with bootstrap confidence intervals; use those numbers, not
these.

The dataset is used under the Apache License 2.0.
