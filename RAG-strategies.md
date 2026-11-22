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

