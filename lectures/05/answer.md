# Lecture 05 Answers

## Exercises

- Answering model: `gemini-3.6-flash` and `gemini-3.1-flash-lite`, run on the same
  question for comparison. Later method comparisons use `gemini-3.1-flash-lite`.
- Comparison question: "What constraints and cost estimates does the lecture give for
  model selection — for example, the per-question budget for a lecture Q&A chatbot?"

For gemini-3.6-flash , the answer given was:
* **Chatbot Budget Constraints Example**: When selecting a model for a lecture Q&A chatbot, one constraint is setting a maximum budget (e.g., at most **¥1** per question). At hypothetical pricing, a large model might cost about **¥2** per question while a smaller model costs about **¥0.2** per question (Lecture 2 · Japanese source p.44).

---

Key Takeaways on Cost & Budget in AI Engineering:
1. **Cost vs. Benefit ($Cost < Benefit$)**: Model selection often comes down to picking the fastest and cheapest model that satisfies your evaluation quality requirements, as lower unit costs expand the range of viable applications (Lecture 2 · Japanese source p.40, Lecture 2 · Japanese source p.44).
2. **Estimating Token Costs**: API billing is typically based on input and output token counts, where output tokens (and reasoning/thinking tokens) usually carry higher unit prices (Lecture 2 · Japanese source p.11, Lecture 2 · translated PDF p.97, Lecture 2 · translated PDF p.98).

[time] Answer completed in 7.41 seconds (including retrieval and embedding)
[usage] Input 95,801 tokens / Output 1,138 tokens / Model responses 1
[cost] Estimated generation API cost: $0.076118 USD (excludes embedding and index construction)

For the model gemini-3.1-flash-lite, the answer given was:

If you have a specific question about the course content (such as concepts regarding machine learning, AI engineering principles, or the topics covered in those first three lectures), please feel free to ask! 

**To help you get started, are you looking for information on:**
*   Defining AI Engineering vs. Data Science?
*   The lifecycle of an AI project discussed in the early lectures?
*   Specific tools or frameworks mentioned in the course? 

[time] Answer completed in 2.78 seconds (including retrieval and embedding)
[usage] Input 256 tokens / Output 190 tokens / Model responses 1
[cost] Estimated generation API cost: $0.000349 USD (excludes embedding and index construction)

**Comparison.** On raw numbers `gemini-3.1-flash-lite` is cheaper (about 1/218 the
cost), faster (2.78 s vs 7.41 s) and uses far fewer tokens than `gemini-3.6-flash`.
But the 256 input tokens are the decisive figure: that is too small for any lecture
material to have reached the model, so it never saw the pages and returned a
clarifying question instead of an answer. It cited nothing. Using the Lecture 5
framework for locating failures, this is the row "needed evidence missing → answer
wrong or incomplete", which points at the materials, search terms or result count —
not at the model.

A cheaper run that produces no grounded answer is not a cheaper way of doing the same
work. Cost per question is the wrong measure here; cost per correct, cited answer is
the right one, and for this run it is undefined. By the lecture's own criterion
(pick the fastest and cheapest model that still satisfies the quality requirement,
Lecture 2 · Japanese source p.40, p.44), only `gemini-3.6-flash` cleared the bar on
this question.

**Follow-up that changed this reading.** A later run of the LoRA/QLoRA question with
`gemini-3.1-flash-lite` produced a correct, correctly cited answer (Lecture 2 p.51–52)
once context actually reached it. So the failure above was in retrieval, not in model
capability. The root cause was found separately: the Codespaces interpreter was built
without `--enable-loadable-sqlite-extensions`, so `sqlite-vec` could not be loaded and
every vector-index call raised `AttributeError: 'sqlite3.Connection' object has no
attribute 'enable_load_extension'`. This comparison therefore measures the whole
pipeline under each model, not model answer quality in isolation.

- Target materials: course JSON `data/rag_data/lectures.json` — 9 documents, 438 pages
  in total, one slide page per chunk, each chunk keeping its lecture, chapter, page
  number and source.

  Scope difference between methods, as configured in this exercise:

  | Path | Current scope | How body text is fetched |
  |---|---|---|
  | Full context | All 9 documents, 438 pages | All JSON body text placed in the input at once |
  | Vector search | Lectures 1–3 main lectures, 214 pages | Up to 5 similar pages returned |
  | Tree | Lectures 1–3 main lectures, 214 pages | Follows lecture → chapter → page |
  | Graph | Lectures 1–3 main lectures, 214 pages | Top 3 search hits as entry points, up to 2 hops, up to 10 pages |

  Only full context sees the wider set. The other three are restricted to 214 of the
  438 pages, so full context can answer from material the others cannot reach at all,
  and any difference in correctness between full context and the rest may reflect that
  scope gap rather than retrieval quality. For questions whose evidence lies in
  Lectures 1–3 the comparison is fair; for anything outside that range it is not.
  Full context should therefore be read as the accuracy ceiling for this setup, and
  each retrieval method scored on what it missed relative to it. The methods also
  differ in how much text they place in the input, which is what drives the token and
  cost differences in the table below rather than the quality of the answers.
### Results by Method

| Method | Key points and correctness of the answer | Cited pages and evidence | Time (seconds) | Input/output tokens | Model responses | Estimated generation API cost (USD) |
|---|---|---|---|---|---|---|
| Full context | | | | | | |
| Vector search | | | | | | |
| Tree | | | | | | |
| Graph | | | | | | |

<!-- State when a method was not run or a cost was unavailable. Use a separate table for each additional question. -->

- Pages found by running search alone, and what their text confirmed:
- Tool calls shown by `--trace` (names, arguments, counts, table-of-contents paths followed by Tree, etc.):

- Full context: no tool calls (tools: 0). One model response; the entire JSON body
    text was placed in the input, 369,490 characters.
  - Tree: one tool, `browse_tree`, called four times, descending the table of
    contents one level per call:
    1. `browse_tree({})` — root, listing lec01-main, lec02-main, lec03-main
    2. `browse_tree({'node_id': 'lec02-main'})` — Lecture 2's pages and chapters;
       chose the chapter "When and how to adapt the selected model" (lec02-main-s045)
    3. `browse_tree({'node_id': 'lec02-main-s045'})` — 19 child pages listed with
       first sentences only; p.052 is titled "LoRA and QLoRA: adapting a large model
       on one GPU"
    4. `browse_tree({'node_id': 'lec02-main-p052'})` — leaf page, full text,
       1,238 characters
    Five model responses in total: four tool-calling turns plus the final answer.
    The intermediate candidate lists are first sentences only; full text was fetched
    for exactly one page.
  - Vector search / Graph: no tool calls completed. The agent did emit one
    `search_materials` call (vector) and one `search_graph` call (graph), but both
    raised the sqlite-vec error inside the tool and the run aborted.

- Examples where retrieved information was insufficient, or was retrieved but not used
  in the answer:

  - **Nothing retrieved.** The cost/budget question under `gemini-3.1-flash-lite`
    reported 256 input tokens and returned a clarifying question with no citations —
    too few tokens to hold any lecture page, so no evidence reached the model. This is
    the "evidence missing → answer wrong" case; the cause was the `sqlite-vec` load
    failure, not the model.
  - **Method unable to retrieve.** Vector search and Graph returned no evidence at all
    on the LoRA/QLoRA question, both failing at app/retrieval.py:121. Graph fell with
    vector search because it seeds its entry points from it.
  - **Retrieved but unused.** Full context supplied 95,823 input tokens; the answer
    cited two pages (Lec. 2 p.51–52). Over 99% of the text contributed nothing — the
    answer was still correct, but this is where the 8× cost gap against Tree comes
    from.
  - **Retrieved and correctly discarded.** Tree listed 19 candidate pages but fetched
    full text for only one (p.052), rejecting the rest from titles and first sentences
    before they entered the answer context.
  - **Not yet tested.** No multi-page or not-in-materials question has been run, so
    under-retrieval across chapters and correct refusal on unanswerable questions
    remain unverified.
- Method you would adopt for this use case, your reasons, and limitations of the
  comparison conditions:

  **Provisional choice: Tree, pending the two missing runs.** It matched full
  context's correctness and citations while using about 10% of the input tokens
  (9,437 vs 95,823) and 12% of the cost ($0.0029 vs $0.0242). Full context also scales
  badly — every question pays for all 438 pages — which matters against a per-question
  budget. The trade-off is latency: 5.54 s vs 2.18 s, because Tree made five
  sequential model calls to walk the table of contents instead of one.

  Provisional because vector search has not run, and it is the obvious candidate to
  beat Tree on both axes — one retrieval, one model call. Graph is untested too, and
  is the method most likely to help on cross-chapter questions.

  **Limitations:**
  - *Unequal scope* — full context sees 438 pages, the others 214 (Lectures 1–3), so
    correctness differences may reflect scope, not retrieval quality.
  - *One question, and an easy one* — single-page evidence on p.52, the case Tree
    handles best. No multi-page or not-in-materials question tested.
  - *Half the table empty* — vector search and Graph failed on the `sqlite-vec` error.
  - *Single run each* — no averages, and API latency varies.
  - *Costs exclude* query embedding and index construction.
  - *One answering model* — `gemini-3.1-flash-lite` throughout.

  **To settle it:** fix the interpreter, align full context to Lectures 1–3, then run
  all four methods on the same single-page, multi-page and not-in-materials questions.

- Observations and questions:

  `gemini-3.1-flash-lite` answered the LoRA/QLoRA question correctly with the right
  citations once context reached it, so its earlier non-answer was a retrieval failure,
  not a capability limit. At 1/218 the cost of `gemini-3.6-flash`, the open question is
  which question types actually need the larger model.

## Homework

<!-- Completing any one of Basic, Implementation, or Advanced is sufficient. -->

- Level attempted: Basic / Implementation / Advanced

Basic

- What you did (files, settings, or processing changed, or retrieval method designed):

- Comparison question and conditions held fixed (model, target materials, etc.):
- Results (answers and evidence before/after, `--trace` output, time, token counts, estimated generation API cost, etc.):
- Examples of improvements and remaining failures or limitations:
- Observations and questions:
