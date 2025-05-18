# RagChat App with Fan-Out Query Architecture

This repository contains an enhancement to the original RagChat app, implementing a fan-out query architecture for improved document retrieval performance.

## What is Fan-Out Architecture?

Fan-out architecture is an advanced RAG (Retrieval-Augmented Generation) technique that improves search results by:

1. Generating multiple semantically similar queries from a single user query
2. Searching the vector database with each query variant
3. Combining and deduplicating results for more comprehensive retrieval

This approach helps overcome vocabulary mismatch problems between queries and documents, significantly improving recall without sacrificing precision.

## Files

### `similarQuery.py`

This file implements query expansion functionality using OpenAI's GPT-4o model:

```python
def generate_similar_queries(query: str, num_queries: int = 5) -> list:
    """
    Generate similar queries to a given user query using OpenAI's GPT model.
    """
```

Key features:
- Uses a specialized system prompt to generate paraphrased search queries
- Returns a clean JSON array of similar queries
- Configurable number of similar queries to generate (default: 5)
- Error handling for JSON parsing

### `vectorstore.py`

This file handles vector storage operations with Qdrant, including the fan-out search implementation:

```python
def retrieve(query: str, collection_name: str, k: int=5):
    """
    Retrieve relevant documents from a collection based on a query using fan-out method.
    """
```

Key features:
- PDF document ingestion and chunking
- Vector embedding and storage with Qdrant
- Fan-out search methodology
- Result deduplication by document source
- Collection management (creation, listing, deletion)

## Benefits of Fan-Out Architecture

1. **Increased recall**: Finds relevant documents that might not match the original query's exact wording
2. **Enhanced coverage**: Captures a broader range of relevant information
3. **Improved result quality**: Returns the most relevant documents across all query variations
4. **Minimal latency impact**: Parallel processing of multiple queries
5. **Better handling of ambiguity**: Covers different interpretations of the same query

## Implementation Details

The fan-out architecture is implemented in the `retrieve()` function of `vectorstore.py`:

1. The original user query is expanded into multiple semantically similar queries
2. Each query is embedded and used to search the vector database
3. Results from all queries are combined
4. Duplicate documents are removed (based on source metadata)
5. The unique results are returned as a tuple of (texts, pages, metadata)

## Integration with RagChat App

To integrate these files with the original RagChat app:

1. Place `similarQuery.py` in the `app/utils/` directory
2. Replace the existing `vectorstore.py` with this enhanced version
3. Ensure the import paths are correctly set up
4. Update any calls to the retrieval function to use the new implementation

## Example Usage

```python
# Initialize a vector store with a PDF document
from pathlib import Path
from vectorstore import init_store_for_pdf, retrieve

# Index a PDF document
pdf_path = Path("path/to/document.pdf")
collection_name = init_store_for_pdf(pdf_path)

# Retrieve documents using the fan-out method
texts, pages, metadata = retrieve("What is machine learning?", collection_name)

# Use the retrieved documents in your RAG pipeline
```

## Dependencies

- LangChain
- Qdrant
- OpenAI Python SDK
- Various Python utilities (pathlib, logging, etc.)

## Performance Considerations

The fan-out approach may increase API costs and processing time due to:
- Multiple OpenAI API calls for query generation
- Multiple vector similarity searches

These costs are generally outweighed by the significant improvement in retrieval quality, especially for complex or ambiguous queries.

## Future Improvements

Potential enhancements to consider:
- Dynamic adjustment of the number of similar queries based on query complexity
- Weighted combination of results based on similarity scores
- Integration with hybrid search techniques (keyword + vector)
- Caching of similar queries for frequently asked questions
