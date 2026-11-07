---
name: careful-of-spoilers
description: Answer book and story questions while being careful to not reveal any spoilers.
---
# Careful of Spoilers (`/careful-of-spoilers`)
Be extremely cautious about accidentally revealing spoilers when answering the user's questions, using **the Draft → Censor → Revise pattern** to guard against every spoiler risk. Subtle hints, loaded tenses, hedge words (*"for now"*), or over-explaining in a way that would leak future information are critical failures. Follow these steps in order:
## Step 1: Model Capability Check
Check the active model (and any censor subagent model) **only if the model identity is visible in the runtime**:
1. **If Model Cannot Be Inferred:** If the runtime does not expose the underlying model identity, **assume the user selected an acceptable model and proceed silently to Step 2** (do not run a search or ask the user).
2. **If Model Is Known:** The model should be released within the **last 4 months** (verified via an available web-search tool against the current date) and must not be a small/lightweight tier (ex: `luna`, `haiku`, `flash`, `flash-lite`, mini/nano). If it fails either check, tell the user this model is not recommended because larger, newer models are better at reasoning about spoiler protection, and ask whether they want to proceed anyway before continuing.
3. **Subagents:** Inherit the active model by default, or use a different qualifying model if requested.
## Step 2: Lock In the "Spoiler Horizon"
Before answering any story question, confirm the user's bookmark:
* **Book Title** (and series Book #) + whether any prequels/spin-offs were read.
* **Exact Position:** Chapter # + the **last scene/event read** (guards against audiobook/edition numbering shifts). Only ask whether they **finished vs. are mid-chapter** if the answer depends on events *inside* that specific chapter; if unspecified, clamp to the **beginning of that chapter** (or to their stated scene landmark, if that is further along).
* **Zero-Plot Clarification Rule:** When asking the user to clarify their position, ask an **open-ended, plot-free question**. **NEVER** list what happens in specific chapters, offer multiple-choice scene descriptions, or mention any event/location from the book that the user did not already state in their message. If the user's reading-progress statements conflict, silently clamp to whichever bound is **earlier** without describing the chapters in between (including in the banner or notes).
If missing or ambiguous, ask before answering; once locked in, reuse this horizon across follow-up turns unless updated. Begin every final response with:
> 🔒 **Spoiler Horizon:** *[Book Title] — [Through Chapter X / Scene Landmark]*
## Step 3: Main Agent Draft & Triage
1. **Draft Internally (Keep Chat & Thought Trace Summary Spoiler-Free):**
   * Research chapter-specific recaps (`"[Book]" "Chapter X" summary`, avoiding wiki infoboxes; phrase search queries without spoiler terms when feasible) if needed, and compose a candidate response using only events up to the Spoiler Horizon and following the **Step 5 Zero-Tell Rules**.
   * **Citations & Supporting Snippets:** Attach inline chapter citations (`[Ch. X]` or `[Ch. X, optional brief scene]`; scene landmark is optional) to factual claims whenever the chapter is known. Include a short inline verbatim quote (1–2 sentences) when it directly supports a statement **only if** you are highly certain of the exact wording; **bias strongly toward omitting the quote** if there is any doubt (never put a paraphrase in quotation marks).
   * **Thought-Trace & Tool-Summary Hygiene:** Keep internal thinking/reasoning traces and tool-call summaries strictly generic and plot-free (ex: *"Preparing candidate draft for censor review"*—never write plot facts, twists, or the draft itself in reasoning traces, as UIs often summarize them into a visible header). Pass the candidate draft directly inside the review subagent prompt (or a hidden scratch file).
2. **Route by Complexity:**
   * **Path A — Trivial Questions** (e.g., *"What's the main character's name?"*, *"How old is Y when introduced?"*):
     * Verify the answer isn't a disguised later twist (e.g., secret identity/age). If a twist exists, escalate to Path B. Otherwise, self-audit via Steps 4–5 and output directly.
   * **Path B — Non-Trivial Questions** (plot, motives, relationships, lore, mysteries, theories):
     * **Mandatory Censor Review:** Send the user's question, the Spoiler Horizon, and your **complete candidate draft** to an isolated review subagent (inheriting the active model or using a user-specified qualifying model). Do **not** expose the unreviewed draft in the chat or thought-trace summary while waiting.
## Step 4: Censor Review & Feedback
1. **Inside the Censor Subagent:**
   * **Audit Every Atomic Claim & Citation:** Break the Main Agent's draft into individual facts/traits/events and verify the exact **Book and Chapter** where each is first revealed (using an available web-search tool on chapter summaries if needed).
     * `PASS`: Revealed strictly at or before the bookmark, with accurate `[Ch. X]` citation (and any inline quote verified as exact verbatim text from at/before the bookmark).
     * `FLAG / CUT`: Revealed after the bookmark, the chapter is uncertain (**Doubt = Cut**), or an inline quote is unverified/paraphrased (**Uncertain Quote = Cut Quote**).
   * **Audit Linguistic Tells:** Check every sentence against the **Step 5 Zero-Tell Rules**.
   * **Return Review Verdict:** Send structured feedback to the Main Agent listing either `APPROVED` or `REVISIONS REQUIRED` with every specific claim, quote, citation, or subtle hint that must be removed or rewritten.
2. **Main Agent Revision:**
   * Incorporate **all** Censor feedback—cutting or rewriting every flagged item so no awkward gaps remain—before outputting the final response to the user.
## Step 5: Linguistic "Zero-Tell" Scrub (Applied by Both Main Agent & Censor)
Enforce these 7 rules on the draft and final response:
1. **No Hedge Words:** Ban *"for now"*, *"at this point"*, *"so far"*, *"initially"*, *"seemingly"*, *"appears to be"*, *"currently believes"*, *"as far as X knows"*, *"yet"*. Frame directly (*"In Chapter 4, X is..."*).
2. **Literary Present Tense:** Describe ongoing states/characters in present tense anchored at the bookmark (*"X is..."*, never *"X was..."* for characters who later die/change). Use past tense only for scenes prior to the bookmark.
3. **Preserve Red Herrings & Name/Title Horizon:** If Chapter 5 presents a false narrative that Chapter 25 overturns, state the Chapter 5 narrative flatly without winking. Use only names, surnames, aliases, and rank titles known at the bookmark.
4. **Proportionality:** Match explanation depth to the character/item's prominence *at the bookmark*. Never over-explain a minor cameo just because they matter later.
5. **No "RAFO" / Loaded Framing:** Never say *"Keep reading!"*, *"Good catch!"*, or *"You'll see!"*, and avoid loaded terms like *"true motives"* or *"real identity"*. Use: *"Up through Chapter [X], the book hasn't covered that."*
6. **Chapter Citations & Verbatim-Only Snippets:** Cite chapters inline (`[Ch. X]`; adding a scene landmark is optional). Only include short inline quotes when 100% certain of exact wording; otherwise omit the quote and paraphrase without quotation marks.
7. **Clean Output:** Output only the `🔒 Spoiler Horizon` banner and the final scrubbed answer—never the unreviewed draft, audit list, or Censor feedback.
