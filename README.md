# Knowledge Graph Project

This project explores building a knowledge graph from news articles and using it to retrieve context for question answering. Its main workflow is documented in [`notebooks/knowledge_graph.ipynb`](./notebooks/knowledge_graph.ipynb).

## Project contents

- [`notebooks/knowledge_graph.ipynb`](./notebooks/knowledge_graph.ipynb) — an end-to-end tutorial covering article loading, entity extraction, Neo4j graph construction, community detection, retrieval, and an optional LLM integration.
- [`data/articles_all.json`](./data/articles_all.json) and [`data/articles_all.jsonl`](./data/articles_all.jsonl) — article data used by the project. The notebook reads `data/articles_all.json` by default.
- [`requirements.txt`](./requirements.txt) — Python dependencies used across the project.
- `.env` — local Neo4j connection settings; this file is ignored by Git and should remain private.

## Knowledge graph workflow

The notebook processes English articles that have a title and body text:

1. Extract named entities with spaCy (people, organizations, places, events, and facilities).
2. Store articles and entities in Neo4j, linking them with `MENTIONED_IN`.
3. Link entities that co-occur in an article with weighted `CO_OCCURS_WITH` relationships.
4. Detect entity communities with NetworkX's Louvain algorithm and visualize the graph.
5. Retrieve articles using entity matches, graph neighbors, and a Neo4j full-text index.
6. Produce a basic extract-based response, or pass the context to an optional LLM function.

The graph uses this model:

```text
(Entity {name, type})-[:MENTIONED_IN]->(Article {article_id, title, text, ...})
(Entity)-[:CO_OCCURS_WITH {weight}]->(Entity)
```

The co-occurrence `weight` is the number of distinct articles shared by an entity pair. `Entity` identity includes both its name and type.

## Requirements and setup

- Python 3.12 is the notebook's configured kernel version.
- A running, reachable Neo4j database (Neo4j Aura is supported).
- The article JSON input file.

Create and activate a virtual environment if desired, then install the notebook dependencies:

```bash
python -m pip install pandas neo4j python-dotenv spacy networkx matplotlib jupyterlab
python -m spacy download en_core_web_sm
```

The repository's `requirements.txt` also includes LangChain and LlamaIndex packages for the broader project environment.

Create a `.env` file in the repository root with your Neo4j settings:

```dotenv
NEO4J_URI=neo4j+s://your-instance.databases.neo4j.io
NEO4J_USERNAME=neo4j
NEO4J_PASSWORD=your-password
```

Use your own database values. Never commit actual credentials or share the `.env` file.

## Run the notebook

From the repository root, start Jupyter:

```bash
jupyter lab
```

Open `notebooks/knowledge_graph.ipynb` and run the cells in order. The notebook verifies its Neo4j connection before loading articles into the graph. The default sample limit is 50 articles; set `N_ARTICLES = None` to process all articles. Note that the notebook loads the JSON file before applying this limit, so a large input file can still require substantial memory.

The notebook currently contains a workspace-specific absolute `ARTICLES_PATH`. When using another checkout, update it to the local path of `data/articles_all.json`. If Jupyter is started from the repository root, `data/articles_all.json` is the corresponding relative path.

The basic `ask()` example returns article extracts and source links. An optional LLM can be supplied as a function that accepts a prompt and returns text. The notebook contains a commented Anthropic example; enabling it requires installing `anthropic` and setting `ANTHROPIC_API_KEY` in the local environment.

## Notes

- Neo4j schema creation uses constraints and a full-text index. Article, entity, and mention records use `MERGE` so those notebook steps can be rerun without creating duplicate records.
- Co-occurrence relationships and weights are recomputed from the articles currently represented in the graph.
- Close the Neo4j driver at the end of a run, as shown in the notebook.
