# Lab 7: Vector Store (ChromaDB)

## Learning Objectives
- Set up and use ChromaDB for vector storage
- Create embeddings for text data
- Implement semantic search
- Manage vector collections
- Optimize similarity search

## Prerequisites
- Labs 1-6 completed
- Understanding of embeddings
- Basic vector mathematics

## Setup Requirements
1. Install: `pip install chromadb sentence-transformers`
2. Create lab7 directory
3. Review embedding concepts

## Step-by-Step Instructions

### Step 1: ChromaDB Setup
```python
import chromadb
from chromadb.config import Settings
from sentence_transformers import SentenceTransformer

# Initialize ChromaDB (persistent local storage)
client = chromadb.Client(Settings(
    chroma_db_impl="duckdb+parquet",
    persist_directory="./chroma_db"
))

# Create a collection with cosine similarity
collection = client.create_collection(
    name="knowledge_base",
    metadata={"hnsw:space": "cosine"}
)

# Initialize embedding model
embedder = SentenceTransformer('all-MiniLM-L6-v2')
```

### Step 2: Adding Documents
```python
def add_documents(docs: list, metadatas: list = None, ids: list = None):
    """Add documents to the collection."""
    # Generate embeddings
    embeddings = embedder.encode(docs).tolist()
    
    # Generate IDs if not provided
    if ids is None:
        ids = [f"doc_{i}" for i in range(len(docs))]
    
    # Add to collection
    collection.add(
        embeddings=embeddings,
        documents=docs,
        metadatas=metadatas,
        ids=ids
    )
    
    return f"Added {len(docs)} documents"

# Example usage
documents = [
    "Python is a high-level programming language.",
    "Machine learning is a subset of artificial intelligence.",
    "Natural language processing deals with text data.",
    "Deep learning uses neural networks with multiple layers.",
    "Computer vision enables computers to understand images."
]

metadatas = [
    {"category": "programming", "language": "python"},
    {"category": "ai", "subcategory": "ml"},
    {"category": "ai", "subcategory": "nlp"},
    {"category": "ai", "subcategory": "deep-learning"},
    {"category": "ai", "subcategory": "cv"}
]

add_documents(documents, metadatas)
```

### Step 3: Semantic Search
```python
def semantic_search(query: str, n_results: int = 3):
    """Search for similar documents."""
    # Generate query embedding
    query_embedding = embedder.encode([query]).tolist()
    
    # Search collection
    results = collection.query(
        query_embeddings=query_embedding,
        n_results=n_results
    )
    
    return results

# Example searches
queries = [
    "What is machine learning?",
    "How does neural networks work?",
    "Tell me about Python programming."
]

for query in queries:
    results = semantic_search(query)
    print(f"\nQuery: {query}")
    print(f"Results: {results['documents'][0]}")
```

### Step 4: Filtering with Metadata
```python
def filtered_search(query: str, filter_category: str, n_results: int = 3):
    """Search with metadata filtering."""
    query_embedding = embedder.encode([query]).tolist()
    
    results = collection.query(
        query_embeddings=query_embedding,
        n_results=n_results,
        where={"category": filter_category}
    )
    
    return results

# Search only in "ai" category
results = filtered_search("deep learning techniques", "ai")
```

### Step 5: Managing Collections
```python
def list_collections():
    """List all collections."""
    return client.list_collections()

def delete_collection(name: str):
    """Delete a collection."""
    client.delete_collection(name)

def get_collection_stats(name: str):
    """Get collection statistics."""
    collection = client.get_collection(name)
    return {
        "name": collection.name,
        "count": collection.count(),
        "metadata": collection.metadata
    }
```

## Exercises
1. **Basic**: Build knowledge base from documents
2. **Intermediate**: Implement hybrid search (keyword + vector)
3. **Advanced**: Add metadata filtering and faceted search

## Next Steps
- Lab 8: Security oversight in agentic systems
- Apply vector stores to memory systems
