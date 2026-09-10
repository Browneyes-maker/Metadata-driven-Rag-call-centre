# Technical Defence of the Computational Implementation

**Author:** Lineo Nkoebe  
**Purpose:** Technical explanation of the code accompanying the MSc Computing dissertation.

## 1. What problem does the code address?
The implementation investigates whether structured call-centre metadata can support useful, explainable decision assistance without requiring the full conversational transcript for every call. Instead of sending raw conversations to a language model, the pipeline constructs compact textual representations from operational metadata, retrieves historically similar cases, and uses the retrieved context to produce a concise support recommendation.

The code is a research proof of concept and evaluation implementation. It is not presented as a production call-centre application. Production deployment would require additional security, monitoring, integration, governance, scalability, and continuing validation controls.

## 2. End-to-end logic
The notebook is deliberately ordered as one auditable pipeline:

1. Configure reproducibility and portable file paths.
2. Check whether the primary language-model dependencies are available.
3. Load the authorised operational CSV when present.
4. Clean fields, parse time variables, create timestamp features, and hash identifiers.
5. Perform aggregate data-quality and exploratory checks.
6. Convert selected non-conversational operational fields into metadata documents.
7. Build a retrieval representation and cosine nearest-neighbour index.
8. Retrieve the most similar historical metadata records for a query.
9. Supply those retrieved records to the generation stage as grounding context.
10. Generate a constrained `TAG | ACTION` response.
11. Parse and validate the response against an allowed tag vocabulary.
12. Run demonstrations, retrieval diagnostics, and a final reproducibility summary.

This sequence allows the reviewer to trace how an input is transformed into a recommendation and to inspect each intermediate stage.

## 3. Why use metadata instead of relying on transcripts?
Full transcripts contain richer linguistic information, but they also increase storage, transcription, privacy, security, and computational requirements. The metadata-driven path investigates whether operational signals such as call type, direction, queue, waiting time, handling time, abandonment information, time of day, and business-hours status can provide sufficient context for a narrower decision-support task.

The purpose is not to claim that metadata contains everything a transcript contains. The research question is whether a lighter and more privacy-conscious representation can still provide useful operational decision support for defined scenarios.

## 4. Privacy controls in the implementation
The raw `Number` and `Customer Number` fields are not inserted into the metadata prompts. The preprocessing code includes one-way SHA-256 hashing for identifier fields, and the metadata-document constructor selects operational attributes rather than raw identifiers or conversational text. The GitHub copy does not contain the operational CSV and has no saved raw-record notebook outputs.

Hashing reduces direct exposure but should not be interpreted as making unrestricted publication of the underlying operational dataset appropriate. For this reason the source dataset remains outside the repository.

## 5. Why MiniLM embeddings?
The primary path uses `sentence-transformers/all-MiniLM-L6-v2`. Sentence embeddings map short metadata descriptions and queries into numerical vectors so that semantically similar descriptions can be compared even when their wording is not identical. MiniLM was selected as a compact Sentence-Transformer suitable for a lightweight proof of concept where computational efficiency is important.

The embedding stage is not itself the final decision. It supports retrieval of relevant historical metadata that can ground the later generation stage.

## 6. How similarity retrieval works in this research notebook
The primary research path uses FAISS `IndexFlatIP` for exact vector retrieval. MiniLM embeddings are L2-normalised before indexing, and query embeddings are normalised in the same way. With normalised vectors, inner product is equivalent to cosine similarity; therefore, a higher FAISS score indicates a more semantically similar metadata record.

The notebook retrieves the top-k historical metadata records and passes those records to the generation stage as contextual evidence. If the primary model dependencies are unavailable, the notebook uses a clearly labelled scikit-learn cosine nearest-neighbour fallback only to validate software flow.

## 7. Why FAISS and cosine similarity?
The primary implementation uses FAISS (`IndexFlatIP`) to retrieve the most similar historical metadata records efficiently. MiniLM embeddings are L2-normalised before they are added to the index. With normalised vectors, the inner product returned by `IndexFlatIP` is mathematically equivalent to cosine similarity, so higher scores indicate greater semantic similarity. This directly implements the dissertation's FAISS-based retrieval architecture while keeping the similarity interpretation straightforward.

FAISS was selected because it is designed for efficient vector similarity search and can scale beyond simple exhaustive comparisons. For the 10,000-record proof-of-concept index used here, `IndexFlatIP` provides exact similarity search without approximate-index tuning, which makes the research implementation easier to audit.

## 8. Why Retrieval-Augmented Generation (RAG)?
RAG separates evidence retrieval from language generation. Instead of asking the generator to respond only from its pretrained parameters, the implementation first retrieves metadata examples from the research corpus and places them in the prompt. This provides task-specific context and makes the evidence supplied to the generator inspectable.

RAG does not guarantee factual correctness and does not eliminate hallucination. In this implementation, hallucination risk is additionally constrained through a small allowed tag vocabulary, deterministic decoding, a strict parser, short actions, and a human-escalation fallback.

## 9. Why FLAN-T5 small?
The primary generation path uses `google/flan-t5-small`, a compact instruction-oriented sequence-to-sequence model. It is used for a deliberately narrow task: return one permitted category and one concise action, rather than produce unrestricted conversational text. A smaller model is consistent with the study's lightweight implementation objective and makes the proof of concept more feasible on modest computing resources than a much larger generative model.

Generation is configured with `do_sample=False` and `num_beams=1` to reduce random variation between repeated runs.

## 10. Why `TAG | ACTION`?
The output contract is intentionally simple:

`TAG | ACTION`

The `TAG` represents a controlled operational category, while `ACTION` provides a short next-step recommendation. This makes the output easier to inspect, parse, test, and potentially integrate into an agent-support workflow than unrestricted prose.

The allowed tag set is defined in code. The parser checks generated text against that set. If a valid result cannot be established, the code can return `unknown` and a human-escalation action. This is a safety-oriented design decision: uncertainty should not be hidden behind fluent free text.

## 11. Why a reproducible 10,000-record metadata subset?
For the proof-of-concept retrieval index, the notebook uses up to 10,000 records selected with `random_state=42`. This provides a manageable computational subset while avoiding the bias of simply taking the first chronological records. The fixed seed makes the selection reproducible when the same source data is used.

This subset is an implementation choice for the retrieval proof of concept and should not be confused with a claim that 10,000 records represent a statistically optimal sample size for every call-centre setting.

## 12. Why retain the hold-time histogram?
The exploratory hold-time histogram is retained because it was part of the original implementation and it faithfully shows an important property of the source data: hold time is overwhelmingly concentrated at zero. The graph is therefore not presented as a rich behavioural distribution, but as a transparent data-quality and operational observation. The values are not altered to make the graph appear more variable.

## 13. What does the offline fallback mean?
The notebook checks whether Sentence-Transformers, Transformers, PyTorch, and FAISS are available. If they are unavailable, it can use an explicitly labelled fallback consisting of TF-IDF retrieval and deterministic rules. This exists so that software structure and control flow can be tested in an offline environment.

The fallback is **not** a replacement experiment, is not semantically equivalent to MiniLM + FAISS + FLAN-T5, and must not be used as evidence of the primary model's performance. The execution validation record makes this boundary explicit.

## 14. How should the demonstrations be interpreted?
The demonstration queries show that the retrieval, prompt-construction, generation, parsing, and output stages connect end to end. They are examples of pipeline behaviour, not a substitute for a labelled accuracy evaluation. The retrieval-consistency cell is likewise a diagnostic that checks simple type-oriented retrieval behaviour; it is not presented as a comprehensive performance metric.

Any performance claims in the dissertation should therefore be interpreted according to the evaluation method documented in the dissertation rather than inferred from four demonstration prompts in the notebook.

## 15. Reproducibility controls
The implementation includes a fixed random seed, repository-relative dataset path, explicit dependency file, deterministic primary decoding configuration, reproducible sampling, sequential notebook design, strict output parsing, and a final completion marker. These features are intended to reduce hidden state and make the implementation easier to audit remotely.

Exact empirical reproduction still requires the restricted dataset and an environment capable of loading the configured model dependencies/weights. The code repository alone cannot recreate data that has intentionally not been distributed.

## 16. Main limitations
The reviewer should be aware of the following limitations:

- Metadata is less semantically rich than full conversational content and may not capture complex intent or emotion.
- Exact dataset-derived results cannot be reproduced from the GitHub repository alone because the operational CSV is restricted.
- The validation environment used before submission could execute the offline fallback but could not download/run the primary MiniLM/FAISS/FLAN-T5 dependencies; fallback execution therefore validates software flow, not primary-model output.
- Some source fields are sparse; conclusions should not assume that every operational field is equally reliable.
- Demonstration prompts and retrieval diagnostics are not a full labelled end-to-end accuracy study.
- The code is a proof of concept, not a production deployment.

## 17. What is the technical contribution?
The contribution should be understood as the design, integration, and evaluation of a metadata-driven decision-support workflow for the call-centre context: privacy-conscious metadata preparation, semantic retrieval of related historical cases, retrieval-grounded lightweight generation, and constrained operational outputs. The study does **not** claim to have invented MiniLM, FAISS retrieval, RAG, or FLAN-T5 individually.

The research value lies in how these components are structured and evaluated for the study's call-centre decision-support problem and in examining the trade-off between transcript-rich processing and a lighter metadata-driven approach.

## 18. Reviewer quick trace through the notebook
For a rapid technical review, the reviewer can follow the notebook in this order: configuration/dependencies -> data loading -> preprocessing/privacy -> data-quality checks -> EDA -> metadata documents -> embeddings/retrieval -> retrieval sanity check -> FLAN-T5 dispatcher -> strict parser -> RAG orchestration -> demonstrations -> retrieval diagnostic -> final reproducibility summary.

The final message `PIPELINE EXECUTION COMPLETE` indicates that all data-dependent cells have executed sequentially in the current kernel.


## Exploratory figures retained from the implementation

The repository includes the two original exploratory figures generated from the authorised call-centre dataset: the call-type distribution and the hold-time histogram clipped at the 99th percentile. Their plotting specifications are preserved in the notebook so the figures can be regenerated when the authorised dataset is available. The near-zero hold-time distribution is retained without alteration because it is an empirical characteristic of the supplied source field and should not be cosmetically changed for presentation purposes.
