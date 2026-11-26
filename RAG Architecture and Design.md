# RAG Architecture and Design

<style>
/* General note styling */  
  body {
    font-size: 130%;
  }
</style>

## Outline

- [RAG Architecture and Design](#rag-architecture-and-design)
  - [Outline](#outline)
  - [RAG strategies](#rag-strategies)
    - [Re-ranking](#re-ranking)
    - [Agentic RAG](#agentic-rag)
    - [Knowledge graph](#knowledge-graph)
    - [Contextual retrieval](#contextual-retrieval)
    - [Query expansion](#query-expansion)
    - [Multi-query RAG](#multi-query-rag)
    - [Context-aware chunking](#context-aware-chunking)
    - [Late chunking](#late-chunking)
    - [Hierarchial RAG](#hierarchial-rag)
    - [Self-reflecting RAG](#self-reflecting-rag)
    - [Fine-tuned embeddings](#fine-tuned-embeddings)
  - [Retrieval strategies](#retrieval-strategies)
    - [Spars retrieval](#spars-retrieval)
    - [Dense retrieval](#dense-retrieval)
    - [Hybrid search](#hybrid-search)
      - [Weighted score fusion](#weighted-score-fusion)
      - [Reciprocal rank fusion (RRF)](#reciprocal-rank-fusion-rrf)
  - [External sources](#external-sources)

## RAG strategies

### Re-ranking

2 separate retrieval, first we retrieve large number of chunks from the vector database then use specialized re-ranker model to get the most relevant records among the retrieved chunks (top_n).

Pros:

- Significantly, better precision.
- More knowledge considered without overwhelming LLM.

Cons:

- Slightly slower than pure vector search.
- Uses more compute (as we use extra model for the re-ranking).

### Agentic RAG

Agentic RAG is an agent that uses RAG as tools interacting with it.  
It have more capabilities like the ability of searching using different vector database based on the user query, in addition to having other tools that can search the knowledge in other way.  
It gives the agent the intelligently of explore the data and not just work with what is given in first shot.

Architecture

- AI agent application.
- Persistent knowledge.
- Available functions, Example:
  - Tool functions.
  - Action functions.
  - LLM functions.
  - Retrieval functions.
- LLM layer.

Pros:

- Flexible
- Agent adapt to the query needs automatically.

Cons:

- More complex.
- Less predictable behavior.

### Knowledge graph

Combines vector search with graph database (such as Neo4j or FalkorDB) to capture entity relationships.

Pros:

- Capture relationships that vector miss.
- Great for Interconnected data.

Cons:

- Requires New4j setup, entity extraction and graph maintenance.
- Slower and more expensive.

### Contextual retrieval

Anthropic's method: Adds document-level context to each chunk before embedding. LLM generates 1-2 sentences explaining what the chunk discusses in relation to the whole document.

Pros:

- 35-49% reduction in retrieval failures.
- Chunks are self-contained.

Cons:

- Expensive (1 LLM call per chunk).
- Slower ingestion.

### Query expansion

Expands a single brief query into a more detailed, comprehensive version by adding context, related terms, and clarifying intent.  
Uses an LLM with a system prompt that describes hwo to enrich the query while maintaining the original intent.

Pros:

- Improve retrieval precision by adding relevant context and specificity.

Cons:

- Extra LLM call adds latency.
- May over-specify simple queries.

### Multi-query RAG

Generates multiple different query variations/perspectives with an LLM  (e.g., 3-4 variations), runs all searches concurrently, and deduplicates results.  
Unlike query expansion which enriches one query, this creates multiple distinct phrasings to capture different angles.

Pros:

- Comprehensive coverage
- Better recall an ambiguous queries.

Cons:

- 4x database queries (though parallelized).
- Higher cost.

### Context-aware chunking

Intelligent document splitting that uses semantic similarity and document structure analysis to find natural chunk boundaries, rather than naive fixed-size splitting.

Pros:

- Free.
- Fast.
- Maintains document structure.

Cons:

- Slightly more complex than naive chunking.

### Late chunking

Embed the full document through transformer first, then chunk the token embeddings (not the text). preserve full document context in each chunk's embedding.

Pros:

- Maintain full document context.
- Leverages long-context models.

Cons:

- More complex than standard chunking.

### Hierarchial RAG

Parent-child chunk relationships: Search small chunks for precision, return large parent chunks for context.

Pros:

- Balance precision (search small) with context (return big)

Cons:

- Requires parent-child database schema.

### Self-reflecting RAG

Self correcting search loop that perform initial search, the LLM grades relevance (on scale 1-5), if the score < 3, refine query and search again.

Pros:

- Self correcting.
- Improve over time.

Cons:

- Highest latency (2-3 LLM calls).
- Most expensive.

### Fine-tuned embeddings

Train embedding models on domain-specific query-document pairs to improve retrieval accuracy for specialized domains (medical, legal, financial, etc.).

Pros:

- 5-10% accuracy gains.
- Smaller models can outperform larger generic ones.

Cons:

- Requires training data, infrastructure, ongoing maintenance.

## Retrieval strategies

Retrieval strategies focus specifically on finding relevant documents.

### Spars retrieval

Spars retrieval (keyword search) uses lexical matching techniques to finds documents based on matching terms and their statistical importance.

Example on lexical matching techniques:

- TF-IDF.
- BM25

Pros:

- Simple.
- Fast.
- scalable.
- Cost-effective.

Cons:

- Does not handle synonyms or context well.

It used in situation where exact wording matters like well-defined queries, code, search logs, or legal clauses.  

It does not requires embeddings.

### Dense retrieval

Dense retrieval (semantic search) uses embedding modules to represent both queries and documents as vectors.

Embedding model: takes text and converts it into a vector of numbers. Texts with similar meaning land close together in the vector space, where similarity is calculated as algorisms like K nearest neighbor (KNN) or approximate nearest neighbor (ANN).

Example if the user query: "What invalidates fasting" the system might retrieve a document containing: "Things that break the fast including eating, drinking, and intentional vomiting".  
Even though the wording differs, the semantic meaning is similar.

Examples on embedding models:

- VICE.
- Jvector.

The weakness: Dense retrieval can struggle with exact terminology, rare names, unusual Arabic expressions, and specific identifies.

### Hybrid search

It combine vector and keyword search where the semantic matching handles synonyms and concepts, while the keyword matching ensures that rare but critical terms don't get lost.

**How it works?**  
The query runs both ways in parallel, once as a vector embedding against the embedded knowledge set, and again as a keyword search. It then uses a fusion algorithm to merge results based on scores from both.  

Example:  
Having documents A,B, and C.

Running dense retrieval might produces cosine similarity scores:

- Document A: dense score = 0.91.  
- Document B: dense score = 0.84.  
- Document A: dense score = 0.71.

BM25 might produce:  

- Document A: BM25 score = 12.5.  
- Document B: BM25 score = 8.2.  
- Document C: BM25 score = 15.1.  

The scores distributions are completely different. There are 2 common approaches to this problem

#### Weighted score fusion

Normalize or calibrate the scores, then combine them.  
Formula is:

$$\
S(d)=\alpha S_{dense}(d)+(1-\alpha)S_{sparse}(d)
$$

Where:

- d: document.
- $S_{dense}$: dense retrieval score.
- $S_{sparse}$: spares retrieval score.
- $\alpha$: weight assigned to dense retrieval.

For example $\alpha = 0.6$.  
This means we give dense retrieval 60% of hte weight and spares retrieval 40%.

#### Reciprocal rank fusion (RRF)

Instead of combining raw scores, RRF combines document rankings.  
Its formula is:

$$\
RRF(d)=\sum_{r\in R}\frac{1}{k+rank_r(d)}
$$

Where:

- $R$: the different retrievers.
- $rang_{r}(d)$: the position of document d in a retriever's result.
- $k$: A constant, commonly 60.

## External sources

**Video source:** <a href="https://www.youtube.com/watch?v=tLMViADvSNE&t=515s">Every RAG Strategy Explained</a>  
**GitHub repo:** <a href="https://github.com/coleam00/ottomator-agents/tree/main/all-rag-strategies">Advanced RAG Strategies - Complete Guide</a>  
**Video source:** <a href="https://www.youtube.com/watch?v=r0Dciuq0knU&t=68s">RAG Retrieval Strategies: Sparse, Dense, & Hybrid Explained</a>  
