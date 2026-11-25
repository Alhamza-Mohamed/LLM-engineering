# RAG strategies

## Re-ranking

2 separate retrieval, first we retrieve large number of chunks from the vector database then use specialized re-ranker model to get the most relevant records among the retrieved chunks (top_n).

Pros:

- Significantly, better precision.
- More knowledge considered without overwhelming LLM.

Cons:

- Slightly slower than pure vector search.
- Uses more compute (as we use extra model for the re-ranking).

## Agentic RAG

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

## Knowledge graph

Combines vector search with graph database (such as Neo4j or FalkorDB) to capture entity relationships.

Pros:

- Capture relationships that vector miss.
- Great for Interconnected data.

Cons:

- Requires New4j setup, entity extraction and graph maintenance.
- Slower and more expensive.

## Contextual retrieval

Anthropic's method: Adds document-level context to each chunk before embedding. LLM generates 1-2 sentences explaining what the chunk discusses in relation to the whole document.

Pros:

- 35-49% reduction in retrieval failures.
- Chunks are self-contained.

Cons:

- Expensive (1 LLM call per chunk).
- Slower ingestion.

