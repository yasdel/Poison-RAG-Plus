# Poison-RAG-Plus
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
   *(Placeholder: code for generating or loading embeddings, typically used in RAG-based retrieval components.)*

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
```plaintext
[Placeholder for any embedding extraction code...]
```
*(No detailed description yet — to be updated as needed.)*

---

### Final Notes
The above code (Code 2) exemplifies how a single-phase or chained textual manipulation can drastically influence retrieval-augmented recommendations. By limiting edits to a constrained token budget and using clever injection of emotional or neighbor-based text, adversaries can stay under the radar while shifting item exposure in top-$N$ lists. The methodology highlights the pressing need for **robust textual integrity checks** and **defense mechanisms** in next-generation recommender systems that rely heavily on large language model retrieval pipelines.

