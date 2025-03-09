# Poison-RAG-Plus: Data Augmentation Attacks & Embedding Extraction

## Overview
This repository demonstrates a **data poisoning pipeline** against retrieval-augmented recommender systems (RAG) and collaborative filtering (CF) methods that leverage textual metadata. The project is split into three core parts:

1. **Code1**: Data Processing & Preliminary Steps  
2. **Code2**: Textual Data Augmentation Attacks  
3. **Code3**: Embedding Extraction & Post-Attack Analysis

Each component helps address different stages of our experiments, from preparing datasets to injecting stealthy text manipulations, to embedding the manipulated data for subsequent recommendation experiments.

Our **primary research goal** is to show how relatively small metadata rewrites (e.g., 10% token edits) can disproportionately affect item visibility during recommendation—either **promoting** long-tail items or **demoting** popular ones.

---

## Code1: Data Processing & Preliminary Steps
This script manages **initial data preparation**, covering tasks such as:
1. **Loading & Cleaning**: Reads the raw item metadata (e.g., movie or music descriptions) and cleans it (removing HTML tags, trimming whitespace, etc.).
2. **Long-Tail & Short-Head Division**: Implements the logic to separate items into “long-tail” (unpopular) vs. “short-head” (popular) categories. By default, we use thresholds based on interactions or ratings (e.g., top 20% item frequency = short-head, remainder = long-tail).  
3. **User Profile Setup**: Optionally prepares user preference vectors (e.g., from rating logs) or user descriptors (e.g., from text prompts). This helps later experiments where we test recommendation performance after poisoning.

> **Tip**: If you want to replicate the *exact* long-tail/short-head split for your own dataset, see the comments in the code where we define the popularity threshold. Adjusting that cutoff will change how many items end up in either category.

This code **does not** directly perform any LLM-based rewriting; it sets up the environment so that Code2 can inject malicious metadata.

---

## Code2: Textual Data Augmentation Attacks
**Core functionality** for injecting stealthy textual poisoning into item descriptions. The attacks are designed to:
- Keep semantic similarity relatively high (so the text remains coherent).
- Respect a token-edit budget (often 10-30% changes).
- Allow either **promotion** (adding positive phrases or borrowed text from highly-rated neighbors) or **demotion** (inserting negative or distracting phrases from low-rated neighbors).

### Attack Methods
1. **Emotional Attack**  
   - Injects emotionally charged words (e.g., “enchanting,” “exhilarating”) to boost item perception, or negative terms (“lackluster,” “forced”) to reduce attractiveness.
   - Example:  
     ```python
     promoted_text = generate_semantic_emotional_edit(
       original_text="A charming romantic comedy set in New York...",
       direction="promote",
       max_change_ratio=0.3
     )
     ```

2. **Neighbor (Borrowing) Attack**  
   - Scans a set of “neighbor” items (especially from the opposite popularity tier) to find phrases or keywords. Inserts these phrases in the target item’s description to shift embeddings.
   - Example:  
     ```python
     borrowed_text = borrow_text_from_neighbors(
       original_text="A charming romantic comedy set in New York...",
       neighbor_texts=neighbor_descriptions,
       direction="promote",
       max_change_ratio=0.3,
       max_neighbors=5
     )
     ```

3. **Chain Attack**  
   - Applies *Emotional* edits first, then *Neighbor* borrowing. This often amplifies the embedding shift, but with higher risk of detection if the changes become too noticeable.
   - Example:  
     ```python
     chained_text = chain_emotional_then_neighbor(
       original_text="...",
       neighbor_texts=neighbor_descriptions,
       direction="demote",
       max_change_ratio=0.2,
       max_neighbors=3
     )
     ```

4. **Trigger Attack**  
   - Inserts a multi-sentence trigger phrase carefully designed to cause a large embedding change with minimal token edits. Particularly potent if the trigger references high-sentiment or award-winning items.
   - Example:
     ```python
     trigger_text = modify_text_with_neighbors(
       original_text="...",
       neighbor_texts=neighbor_descriptions,
       direction="promote",
       max_change_ratio=0.3
     )
     ```

### Generating Attack Outputs & Reproducibility
- **Automated vs. Manual**: You can run `python Code2_Embedding_extraction.ipynb` or equivalent to automatically produce attacked descriptions for all items in the dataset. 
- **Optimization for Formula 1**: We balance the *edit distance* constraint (e.g., 10% tokens) and the *semantic similarity* (ensuring re-written text remains coherent). The code includes comments on how we check similarity scores via sentence embeddings (SBERT).
- **Examples**: Detailed examples of each method (emotional, neighbor, chain, trigger) are provided in the notebook to help new users see how final text changes are produced.

For a typical user-based experiment, we might run:
> 4 (attack strategies) × 2 (promote/demote) × 3 (LLM types) × 2 (user profile styles) × 120 (users) = ~6000 total evaluations.

Given these large numbers, we selectively showcase a few sample transformations in the paper and point to the complete logs in `logs/` for further transparency.

---

## Code3: Embedding Extraction & Post-Attack Analysis
**Goal**: Convert the newly “poisoned” item descriptions into vector embeddings for subsequent retrieval or re-ranking steps.

### Main Features
1. **Multiple Embedding Models**  
   - **OpenAI** (e.g., `text-embedding-ada-002`) via remote API calls  
   - **Sentence Transformers** (local)  
   - **LLaMA** (local, if configured)  
   You can switch models by changing the `--embed_model` flag in your command or code cell.

2. **Chunking & Parallelization**  
   - For large catalogs, the code processes items in batches to avoid timeouts or exceeding rate limits.
   - Embeddings are stored in compressed numpy arrays for efficient loading later.

3. **Integration with RAG**  
   - After embedding, you can feed these vectors into your retrieval engine (e.g., FAISS, Annoy, or Elasticsearch) for approximate nearest-neighbor searches.  
   - The updated embeddings reflect the attacker’s textual changes. Thus, when the recommendation engine queries them, rank shifts can occur.

4. **Detailed Examples**  
   - Sample usage:
     ```python
     # Pseudocode example
     from embeddings_extraction import generate_embeddings

     # Suppose 'poisoned_items.csv' has item IDs and attacked descriptions
     embeddings = generate_embeddings(
       input_file="poisoned_items.csv",
       model_type="openai"
     )
     # This returns an array of vectors, each corresponding to an item’s new text
     ```
   - Each approach (OpenAI, ST, LLaMA) has its own configuration block. See inline documentation in the `.ipynb` for how to run them.

---

## Frequently Asked Questions (FAQ)
**Q1: How do I replicate the long-tail vs. short-head split?**  
A1: In **Code1**, modify the `popularity_threshold` variable (or function). By default, it sets a certain percentile of interaction frequency to label items as short-head, with the rest considered long-tail.

**Q2: Where can I see a complete example of generated attacks?**  
A2: In **Code2**, check `examples/` or the sample Jupyter cells. They illustrate how we feed original text, neighbors’ text, and a direction (promote/demote) to get the final attacked description.

**Q3: Does the code show the optimization steps for Formula 1 explicitly?**  
A3: Yes. **Code2** includes commentary near the rewriting functions explaining how we balance token-edit constraints (`max_change_ratio`) with semantic similarity checks. We also log the final success or failure (e.g., if more tokens were changed than allowed).

**Q4: How do I test the system on datasets other than MovieLens?**  
A4: Both **Code1** and **Code2** can ingest different CSV or JSON item metadata. For LastFM, we provide an additional script (`lastfm_data_preprocessing.py`) that aligns audio track metadata with the same pipeline. Future versions will include more instructions for large-scale runs.

---

## Future Work
- **Extended Dataset Evaluations**: We plan to add more benchmarks (e.g., BookCrossing, real-time product catalogs) to show generalizability.
- **Advanced Defense Mechanisms**: Upcoming releases will include code to test anomaly detectors on textual metadata.
- **User-Friendly Interfaces**: We intend to create streamlined CLI commands or Docker containers so anyone can run these experiments locally with minimal setup.

---

## Conclusion
**Poison-RAG-Plus** demonstrates how subtle textual edits can cause outsized shifts in retrieval-based recommendations. By combining emotional phrasing, neighbor borrowing, and multi-sentence triggers, attackers can effectively promote or demote items without massive rewrites. This repository provides an end-to-end workflow—from data preparation, to text poisoning, to embedding extraction—allowing researchers to replicate, evaluate, and hopefully defend against such vulnerabilities in LLM-driven recommender systems.
