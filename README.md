# Step 2: LLM_Generated Data Augmentation Attack 
## Against RAG and CF using Side Textual Information

### Description of Research
Retrieval-augmented generation (RAG) architectures combine an external knowledge store (e.g., a corpus of item descriptions) with a large language model (LLM) to produce final outputs, such as recommendations. While this approach improves factual grounding, it also introduces a new vulnerability: **textual data poisoning**.

An attacker who can edit item metadata (e.g., descriptions) might inject subtle changes—like adding positive or negative wording, borrowing text from neighbors, or chaining multiple rewrite strategies—to influence how a recommendation system retrieves and ranks items. Even if only a small fraction of each item description is altered, these manipulations can lead to outsized shifts in exposure and recommendation rankings (promoting unpopular items or demoting popular competitors).

This repository’s second step focuses on a **code-based framework** to demonstrate and analyze these textual data-poisoning strategies. It uses various rewrite approaches (emotional edits, neighbor borrowing, chained methods) under a maximum token-edit constraint, then shows how these changes propagate once the system re-trains or re-indexes the data for retrieval-augmented recommendations.

---

## Files

1. **Processing**  
   *(Placeholder: initial data preparation, loading, cleaning, etc.)*

2. **Data Augmentation Attack**  
   **Brief Statement**: This is the core component where we implement the _textual manipulation and attack strategies_ to subtly alter item descriptions. By limiting the number of changed tokens, the attacker can stay below a detection threshold. We then show how these edits lead to significant shifts in recommended item rankings when the system is retrained or re-indexed.

3. **Embedding Extraction**  
   **Brief Statement**: This code handles **embedding generation** for item descriptions, supporting multiple embedding methods (OpenAI, Sentence Transformers, or LLaMA). It can download or load the previously augmented dataset, create embedding vectors for each item’s (rewritten) text, and then output them in compressed chunks.

---

## More Details

### Code 1
```plaintext
[Placeholder for any preliminary or data-processing code...]
```
*(No detailed description yet — to be updated as needed.)*

---

### Code 2


# Textual Data-Poisoning for RAG-based Recommenders

**Filename**: `attack_generation.py`  

This code demonstrates how to **promote** or **demote** items using LLM-Generated textual attacks in recommender systems. We explore **four attacker strategies**:

1. **Emotional Attack**  
   Inject sentiment-laden words (positive or negative) while respecting a max-change limit.

2. **Neighbor Attack**  
   Borrow exact phrases from popular (or unpopular) "neighbor" items to subtly sway embedding similarity.

3. **Chain Attack**  
   Combine Emotional and Neighbor rewriting in two steps for a stronger effect.

4. **Trigger Attack**  
   Insert a short verbatim "trigger phrase" into the item description to anchor the embedding in a specific semantic direction.

These strategies are inspired by the accompanying paper, which studies stealthy textual poisoning with minimal edits. **Even a 10% token change** can strongly shift item rankings without harming overall system metrics like Recall or nDCG.

## Usage Examples

- **Promote** an unpopular movie by injecting emotional superlatives:
  ```python
  attacked_text = generate_semantic_emotional_edit(
      original_text="A quiet family drama set in the countryside.",
      direction="promote",
      max_change_ratio=0.10
  )
- **Promote** an unpopular movie by borowwing text from neighbors
    ```python
  borrowed_promote = borrow_text_from_neighbors(
    original_text="An under-the-radar indie flick from 1990s.",
    neighbor_texts=some_popular_descriptions_list,
    direction="promote",
    max_change_ratio=0.15
  )
  ---

### Code 3
**(Embedding Extraction)**

**Paragraph 1**: This code enables **embedding extraction** for item metadata, including any newly augmented descriptions. It supports multiple embedding providers or models (OpenAI Embeddings, Sentence Transformers, and LLaMA). Each approach can be configured via a flag, allowing for a flexible workflow that aligns with the user’s chosen pipeline.

**Paragraph 2**: First, it retrieves a post-attack dataset—i.e., items that have potentially been promoted or demoted—and loads it into a DataFrame. It then generates vector embeddings for the content. For OpenAI, it calls their Embeddings API (e.g. `text-embedding-ada-002`), while for local approaches it uses either Sentence Transformers or LLaMA for on-device embedding.

**Paragraph 3**: We have additional utility functions for chunking, batching, and parallelizing the embedding calls to handle large item catalogs. The resulting embeddings are then saved in a compressed, chunked format to avoid memory or storage overhead, facilitating easy integration with retrieval-augmented recommendation modules.

**Paragraph 4**: After embeddings are created, each row’s textual representation is effectively turned into a high-dimensional vector. Downstream tasks—like approximate nearest neighbor (ANN) search, re-ranking, or LLM-based inference—can utilize these embeddings. The code also illustrates how to handle special tokens, concurrency, and potential rate limits when calling external APIs.

```plaintext
[The full code snippet for embedding extraction is included below, covering 
OpenAI, Sentence Transformers, and LLaMA-based embeddings, with chunk splitting 
and compression for efficient storage.]
```

---

### Final Notes
The above code (Code 2) exemplifies how a single-phase or chained textual manipulation can drastically influence retrieval-augmented recommendations. By limiting edits to a constrained token budget and using clever injection of emotional or neighbor-based text, adversaries can stay under the radar while shifting item exposure in top-$N$ lists.

Meanwhile, **Code 3** provides the foundation for embedding these newly rewritten descriptions, bridging the gap between the data poisoning step and a fully retrained or reindexed RAG pipeline. Overall, this method highlights the pressing need for **robust textual integrity checks** and **defense mechanisms** in next-generation recommender systems that rely heavily on large language model retrieval pipelines.

