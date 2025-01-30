# Step 2: Data Augmentation Attack Against RAG

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
Below is a more in-depth explanation of the **Data Augmentation Attack** module (item 2). This module orchestrates rewriting item descriptions, evaluating their semantic changes, and measuring the impact on final recommendations.

**Paragraph 1**: In this module, we focus on **textual rewriting techniques** designed to evade naive detection systems while significantly affecting retrieval and ranking. The core idea is to limit token-level edits—e.g., changing only 10% to 30% of words—so that the manipulated descriptions still appear coherent. We incorporate various rewriting strategies, such as:
  - **Emotional/Sentiment Shaping** (inserting positive or negative terms to promote or demote items),
  - **Neighbor Borrowing** (borrowing short excerpts from thematically similar or dissimilar items),
  - **Chained Methods** (combining sentiment shaping with neighbor-based text merges in multiple steps).

**Paragraph 2**: This step also introduces **metrics to assess the stealthiness and severity** of the textual changes. For instance, we track:
  - **Lexical similarity** via BLEU scores,
  - **Semantic closeness** using Sentence-BERT embeddings,
  - **BERTScore F1** for deeper token alignment.
By comparing these metrics before and after modification, we quantify how “similar” the new text is to the original, while also verifying that the system’s recommendation outputs can indeed be manipulated in favor of (or against) certain items.

**Paragraph 3**: Once the new descriptions are injected, we re-run the retriever or re-index the knowledge store in a typical RAG pipeline. Because these textual changes often align with relevant or persuasive keywords, the pipeline is fooled into ranking these manipulated items higher (for promotion) or lower (for demotion). By automating this entire process—from selecting which items to target, generating multiple candidate rewrites, and finally picking the “best” poisoning strategy—we demonstrate that RAG-based systems can be systematically attacked with minimal textual edits.

**(Optional)** A fourth paragraph can discuss **defensive techniques** such as verifying text provenance, monitoring unusual changes in descriptions, or employing robust embedding methods that are less sensitive to minor textual additions. Such methods could be integrated alongside standard data-preprocessing pipelines, but the code here primarily focuses on showing how easily these vulnerabilities can be exploited in typical RAG-based recommenders.

```plaintext
[Here is where the actual attack code from the provided script would appear, 
including the rewrite functions, neighbor borrowing, emotional edits, 
and the logic for measuring semantic differences, etc.]
```

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

