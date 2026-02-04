# MS MARCO Neural Semantic Search System

This project implements a high-performance, two-stage semantic search engine using the MS MARCO dataset. The system leverages a Bi-Encoder for fast initial retrieval and a Cross-Encoder for high-precision re-ranking, supplemented by Pseudo-Relevance Feedback (PRF) for query expansion.

# System Architecture

Stage 0: Query Expansion (Pseudo-Relevance Feedback)

  The initial user query is used to perform a "pilot" search against the FAISS index.
  The system extracts terms from the top k=3 retrieved passages to expand the original query, helping to mitigate vocabulary mismatch.

Stage 1: Fast Dense Retrieval (Bi-Encoder)

  Model: msmarco-distilbert-base-v4.
  Indexing: Uses FAISS (IndexFlatIP) for efficient similarity search.
  Operation: The expanded query is encoded into a 768-dimensional vector and compared against 100,000 indexed document embeddings using normalized cosine similarity.

Stage 2: High-Precision Re-ranking (Cross-Encoder)

  Model: ms-marco-MiniLM-L-6-v2.
  Optimization: Implements FP16 (Half-Precision) to double inference speed.
  Operation: The top 40 candidates from Stage 1 are re-scored by the Cross-Encoder to provide the final ranked results.

# Why I did not implement BM25 and hybrid system:

Semantic Depth over Lexical Matching: I prioritized Semantic Search over BM25 as Dense retrieval captures the actual intent of a query whereas BM25 is limited to exact keyword matches.

Infrastructure & Memory Efficiency: To minimize the System RAM and speed of execution, I avoided the overhead of maintaining a dual-index (Inverted Index for BM25 + Vector Index for FAISS). A single-stream neural pipeline is more quick and streamlined for deployment on hardware like T4 GPU.

Redundancy in Two-Stage Architectures: The high precision of the Cross-Encoder re-ranker effectively compensates for the gaps a lexical layer would fill. Adding BM25 significantly increases Query Latency for only marginal gains in accuracy.

Query Expansion as a Modern Alternative: Instead of a hybrid BM25 system, I used Pseudo-Relevance Feedback (PRF). This semantically "boosts" the query before retrieval, addressing the same recall challenges as lexical search.

# Results and Benchmarks of neural search system:

  Based on an analysis of 100 test queries, the system demonstrates the following performance metrics:

![evaluation](results/evaluation.png)

![evaluation](results/efficiency.png)

![evaluation](results/memory.png)

