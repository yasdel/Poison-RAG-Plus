# Poison-RAG-Plus: Detailed Explanation of the Code and Experimental Procedures

This repository demonstrates **data poisoning attacks** against recommender systems that rely on **Retrieval-Augmented Generation (RAG)** and **Collaborative Filtering (CF)** with side textual metadata. Below is an overview of our code files, the rationale behind each step, and how these scripts together answer the reviewers’ concerns about implementation details, optimization processes, and reproducibility.

## Table of Contents

1. [Scope of Experiments](#scope-of-experiments)  
2. [Code Overview](#code-overview)  
   1. [Code1_Data_Augmentation_Attacks.ipynb](#code1_data_augmentation_attacksipynb)  
   2. [Code2_Embedding_extraction.ipynb](#code2_embedding_extractionipynb)  
   3. [Code3_Original_Poison_RAG_plus_main.ipynb](#code3_original_poison_rag_plus_mainipynb)  
3. [Frequently Asked Questions (FAQ)](#faq)  
4. [Future Improvements and Reproducibility](#future-improvements)

---

## 1. Scope of Experiments
Our experiments cover a large range of scenarios, reflecting both **promotion** (boosting long-tail items) and **demotion** (suppressing popular items), across multiple LLM-based embedding methods. Concretely:

- **Attack Types**: 4 (Emotional, Neighbor, Chain, Trigger)  
- **Promote/Demote Settings**: 2  
- **LLM Types**: 3 (e.g., OpenAI, Sentence Transformers, LLaMA)  
- **User Profiles**: 2 (manual, LLM-generated)  
- **Number of Tested Users**: 120  

Hence, we have roughly **4 × 2 × 3 × 2 × 120 ≈ 6000** distinct rewriting experiments and evaluations. Each scenario can produce thousands of textual variants, making it impractical to store every single output in the paper. This is why our scripts dynamically generate (and log) the textual attacks, store them in CSV files, and then measure ranking impacts in separate steps.

---

## 2. Code Overview

### 2.1 Code1_Data_Augmentation_Attacks.ipynb
This notebook addresses **textual poisoning**. It:
1. **Loads or Downloads** the chosen dataset (MovieLens or Last.fm).
2. **Identifies** which items are **long-tail** (for promotion) vs. **short-head** (for demotion).
3. **Implements the Four Attack Methods**:
   - **Emotional** (inject positive or negative emotional words).
   - **Neighbor Borrowing** (insert verbatim phrases from neighboring items’ descriptions).
   - **Chain** (apply Emotional first, then Neighbor).
   - **Trigger** (a multi-sentence snippet that significantly shifts embeddings).
4. **Token-Edit Constraint**: Each attack carefully tracks how many words/tokens can be modified (e.g., 10% or 30%).
5. **Parallel or Serial Execution**: The user can switch `run_parallel_version = True` to speed up LLM calls for large item catalogs.
6. **Semantic Similarity Checks**: We measure changes with SBERT or BERTScore to ensure the final text remains semantically close.

**Key Sections**:
- **`_augment_one_item()`**: Central function that takes one item’s original text, finds neighbors, applies the chosen ratio of changes, and logs the new text plus semantic differences.
- **`apply_phase1_promotions_demotions()`**: Samples a fraction (e.g., 10%) of target items and runs the four attacker strategies at multiple edit ratios (10%, 40%, 60%, 90%). Results go into new columns (e.g., `phase1_10p_promote_emotional`).
- **Final**: The notebook saves a CSV file (gzipped) with augmented text columns and logs to highlight how each item’s textual description changed.

**Addressing Reviewer Concerns**:
- **Division Strategy**: This file includes `assign_popularity_class()` to label items as long-tail vs. short-head, typically top 20% as short-head and bottom 50% as long-tail.  
- **Attack Output Generation**: The user can see each step’s prompts and final text in the `_augment_one_item()` function (we limit the total text to show just representative examples).  
- **Optimization for Formula 1**: We are not using a gradient-based approach; rather, the script ensures each rewrite respects an edit-distance limit (`max_change_ratio`) and a semantic-similarity threshold (SBERT). This ensures that 1) we don’t exceed the token-edit budget, and 2) the text remains coherent.  

### 2.2 Code2_Embedding_extraction.ipynb
Focuses on converting the newly **poisoned item descriptions** into vector embeddings. This is crucial because a RAG or CF model relies heavily on textual embeddings for retrieval. We support:

1. **OpenAI** (Ada-002)  
2. **Sentence Transformers** (MiniLM, etc.)  
3. **LLaMA** (local embeddings)

**Core Steps**:
- **Load the augmented items** from the CSV produced by Code1 (includes columns like `phase1_10p_promote_emotional`).
- **For each column** representing an attack scenario, generate embeddings in parallel (to handle large catalogs).
- **Chunking and Output**: Because thousands of items can produce large embedding files, we chunk them (e.g., ~25MB each) and store them as multiple `.csv.gz` parts.

**Addressing Reviewer Concerns**:
- **Cost & Feasibility**: We show how we batch requests (OpenAI or local) to handle rate limits. Attack feasibility remains high as we only change ~10% of each text.  
- **Reproducibility**: Users can replicate the exact procedure by specifying the same `embedding_method` (OpenAI, ST, or LLaMA), the same chunk size, and ensuring their `.csv.gz` outputs match our logs.  
- **Long vs. Short-Head**: Not specifically coded here, but we do preserve the `pop_class` column from Code1 so you can differentiate items if needed.

### 2.3 Code3_Original_Poison_RAG_plus_main.ipynb
Demonstrates **an end-to-end pipeline** that:
1. **Builds** a small RAG-based recommendation flow:
   - User embedding from rating logs (temporal or average).
   - A kNN or nearest-neighbor retrieval using the item embeddings from Code2.
   - An optional LLM-based re-ranking or final generation step.
2. **Trains** or indexes the system on either original or attacked metadata, showing how item ranking changes.
3. **Provides** an evaluation framework (Recall@K, NDCG@K) and also logs the position of attacked items in top-K. This helps see how the promotion/demotion worked in practice.

**Key Sections**:
- **Retrieval**: `build_item_matrix()`, `retrieve_top_N_items()`
- **User Embedding**: We show `compute_user_embedding()`, letting you pick “temporal” or “average.”
- **Evaluation**: `compute_recall_at_k()`, `compute_ndcg_at_k()`, plus a final loop over many users to gather metrics.
- **LLM-based Summaries**: We illustrate how you might prompt an LLM to produce a final recommendation JSON. This part is optional for a basic CF or item-based approach, but crucial for RAG.

**Addressing Reviewer Concerns**:
- **Table 1 Correlation**: The code merges user-based retrieval results with final LLM re-ranking, highlighting that different models (OpenAI vs. ST) can yield unexpected rank shifts in demotion scenarios.
- **Attack Feasibility**: We do not forcibly check for “consistency” across the entire dataset, so small changes remain undetected in practice.
- **Multiple Datasets**: We primarily show MovieLens. However, the code includes placeholders for LastFM. This was a minor doc oversight; the code for LastFM is indeed in Code1 (functions like `load_data_lastfm()`), and the approach is the same.  

---

## 3. Frequently Asked Questions (FAQ)

1. **Q**: *How do I replicate the long-tail vs. short-head split exactly?*  
   **A**: In **Code1**, look for functions like `assign_popularity_class()` or `assign_popularity_class_based_on_rating_percentage()`. You can adjust the quantiles or thresholds for top 20% vs. bottom 50%.

2. **Q**: *Where are the outputs for the three example attacks in the paper?*  
   **A**: In **Code1**, we show sample lines at the bottom (e.g., `generate_semantic_emotional_edit()`) with print statements demonstrating the final text. Additionally, after generating the entire dataset, the script saves them in a `.csv.gz` so you can see *all* items’ new text in columns like `phase1_10p_promote_emotional`.

3. **Q**: *How exactly do you optimize Formula 1?*  
   **A**: We do a *constraint-based rewriting* in which we set a `max_change_ratio` and measure approximate semantic similarity. We rely on LLM prompts (with explicit instructions like “Do NOT exceed X words changed”). Thus, we do not use a gradient-based approach but a prompt-driven approach that meets the stealth and token budget constraints.

4. **Q**: *Why are results in the demotion scenario sometimes reversed?*  
   **A**: As mentioned, the retrieval stage can re-introduce certain items if the embeddings remain partially similar or if the LLM re-ranking places unexpected weight on certain cues. We plan more refined neighbor selection to avoid inadvertently boosting items.

5. **Q**: *How can I check if the attack was detected?*  
   **A**: The code does not enforce any detection method. However, you can run “consistency checks” (like text duplication or unusual phrases) by scanning the final `.csv.gz` for certain patterns. We plan to add a demonstration of that in future releases.

---

## 4. Future Improvements

- **Extended Datasets**: We will add BookCrossing or other real-world sets to demonstrate generality.  
- **Defense Mechanisms**: Next, we’ll show how certain textual analysis or anomaly detectors can flag over-edited or repetitive tokens.  
- **Enhanced Optimization**: We may incorporate a partial gradient step (fine-tuning an LLM) that strictly enforces the token distance.  

**All** these notebooks can be run in Google Colab or a local Jupyter environment, ensuring full **reproducibility** and **transparency** for the textual data-poisoning pipeline.

---

**Thank you for your interest in Poison-RAG-Plus!**  
Feel free to open issues or pull requests for any clarifications or improvements.
