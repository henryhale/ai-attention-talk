---
theme: default
title: How much attention do we really need?
author: Henry Ssenono
info: |
  An AI talk about focus, context, and what intelligence chooses to notice.
colorSchema: dark
fonts:
  sans: Roboto
  serif: Roboto Slab
  mono: Fira Code
  provider: google
transition: fade
mdc: true
---

<div class="slide-shell intro-slide">
  <div class="eyebrow">
    <span class="pulse-dot"></span>
    AN AI TALK
  </div>

  <div class="title-wrap">
    <h1>How much <span class="accent">attention</span><br>do we really need?</h1>
    <p class="subtitle" v-click>Machines choose context. So do we.</p>
  </div>

  <div class="meta-row">
    <span>Henry Ssenono</span>
    <span class="meta-line"></span>
    <span>Founder, Conway Labs</span>
  </div>

  <div class="orb orb-one"></div>
  <div class="orb orb-two"></div>
</div>

<!--
Timing: 0:20 · elapsed 0:20

Open: “When a model produces the next word, what did it need to notice to make that choice?”

[click] Land the theme: machines choose context, and that choice is the through-line of the talk.

Set expectations: we will build from tiny memories to attention, then decide where full attention is actually worth paying for.

Transition: “First, a sentence about why I care about making these systems legible.”
-->

---

<AboutSlide />

<!--
Timing: 0:30 · elapsed 0:50

Keep this brief. Introduce yourself and Conway Labs, then make Conway Canvas the concrete example: “I built a product for designing neural networks visually.”

[click] Point to the highlighted Conway Canvas card and mention that the logo and card link to conwaylabs.org.

[click] The AI projects are on the top row: Embedding Explorer trains word embeddings from scratch and exposes the learned 3D space; WhatsApp Analyzer can analyze, summarize, and answer questions about exported chats. DepGraph and Worstcase sit beneath them.

Transition: “So let us make one of AI’s core problems legible: modeling a sequence.”
-->

---

<SequenceIntroSlide />

<!--
Timing: 0:55 · elapsed 1:45

Define sequence modeling in plain language: use ordered history to make a decision about what comes next.

[click] Stress that order matters. “Dog bites person” and “person bites dog” contain the same words but not the same sequence.

[click through task cards] Sequence models can classify or transform sequences, but this talk follows one concrete task: next-token prediction.

Clarify “token”: usually a word or word-piece, drawn from a fixed vocabulary.

Key line: “Given this context, assign a score to every possible next token and choose one.”

Transition: “And that one choice becomes a loop.”
-->

---

<NextTokenSlide />

<!--
Timing: 1:10 · elapsed 2:55

Invite the audience to silently predict the next word before revealing each of the first few rows.

[click through rows] Read left to right: available history, then the selected next token. Point out how the context grows by one token each time.

Call out the lower-triangle shape: at step t, the model may use earlier tokens, never future ones. This is causal context.

[click] The bars are a probability distribution, not certainty. “Opened” wins here, but alternatives still receive probability.

[click] Complete the loop: context → scores → token → enlarged context.

Key line: “Every approach we see solves this same loop; it changes how history is represented and reached.”

Transition: “That gives us a map for seventy years of sequence modeling.”
-->

---

<SequenceTimelineSlide />

<!--
Timing: 0:50 · elapsed 3:45

[click through milestones] Move quickly: count short windows; match explicit rules; compress history into state; protect it with gates; directly retrieve with attention; combine strategies at the frontier.

Do not frame this as old ideas disappearing. Each technique encodes a different trade-off and many remain useful.

[click] Land the frontier question: useful context versus the cost of reaching it.

Preview the route: “We will start with the smallest possible memory and add capability one constraint at a time.”

Transition: “First: what if history is only a few words and a counter?”
-->

---

<NgramSlide />

<!--
Timing: 0:55 · elapsed 4:40

Start on unigram: it ignores the prompt and predicts from overall frequency.

[click] Switch to bigram, then trigram. Point to ACTIVE CONTEXT and MATCHING WINDOWS so the audience sees exactly where the probabilities come from.

Use the default prompt. If time allows, change its last words and let the matches update.

Trade-off: more context gives a sharper clue, but the exact context becomes rarer; unseen windows produce no answer.

Key line: “N-grams remember literally—and only a fixed number of tokens.”

Transition: “Counting learns from examples. What if we specify the valid pattern ourselves?”
-->

---

<RegexSlide />

<!--
Timing: 0:35 · elapsed 5:15

Explain rules as explicit sequence recognizers: we write the pattern first, then test inputs against it.

[click] Use one passing and one failing example. If editing live, make one small change that flips the result.

[click] Land the limitation: a rule can say valid or invalid, but it does not naturally rank plausible next tokens.

Transition: “To learn a flexible history, we need a state that changes as the sequence arrives.”
-->

---

<RnnSlide />

<!--
Timing: 0:50 · elapsed 6:05

Describe the hidden state as a running summary, not a storage box containing every earlier word.

[click through the sequence] At each token: combine the new token with the previous state, then overwrite the state.

Point out the narrow channel: all earlier information must survive through one repeatedly rewritten vector.

[click] Land the failure mode: distant clues fade or get overwritten as the chain grows.

Transition: “LSTMs keep the chain, but add controls for what is allowed to survive.”
-->

---

<LstmSlide />

<!--
Timing: 0:50 · elapsed 6:55

Use three verbs instead of gate jargon: keep, write, reveal.

[click through the example] Identify the early clue—“cat”—and show how the memory lane protects it across distracting words until it is needed.

Explain that gating helps gradients and useful information travel farther, but access is still sequential: the signal must pass through every step.

[click] Land the gain: selective memory is stronger than rewriting everything equally.

Transition: “Now remove the requirement that a useful clue must travel through the whole chain.”
-->

---

<div class="slide-shell history-slide">
  <header class="history-header">
    <div class="history-title">
      <span class="step-index">05</span>
      <h1>Attention makes every token reachable.</h1>
    </div>
    <span class="era-chip">2017</span>
  </header>

  <main class="history-grid">
    <section class="concept-panel">
      <p class="panel-label">THE SHIFT</p>
      <h2>Every token can look back.</h2>
      <figure v-click class="attention-overview-image">
        <img src="/Self-Attention-vs-Multi-headed-Attention.webp" alt="Scaled dot-product attention and multi-head attention architecture">
      </figure>
      <p v-click class="tradeoff"><span>Result</span> Relevance replaces distance.</p>
    </section>

<TransformerAttentionRoute />
  </main>

  <footer class="history-footer">
    <span>HISTORY OF SEQUENCE MODELING</span>
    <span class="slide-progress">10 / 20</span>
<ResourceLink href="https://arxiv.org/abs/1706.03762" label="Vaswani et al. · Attention Is All You Need (2017)" />
  </footer>
</div>

<!--
Timing: 1:20 · elapsed 8:15

State the shift first: every token can directly score every visible earlier token. Distance no longer determines access.

[click] Use the image on the left to distinguish one scaled dot-product attention operation from multiple parallel heads whose outputs are concatenated.

Under the self-attention example on the right, explain the roles: queries ask what is needed, keys advertise what each token contains, and values carry the selected context.

Use the route diagram to point out that links are weighted; attention is not merely seeing everything equally.

[click] Key line: “Relevance replaces distance—but making all those connections has a cost.”

Transition: “Now let us separate the mechanisms that are often collapsed into one word: attention.”
-->

---

<AttentionMechanismSlide mechanism="full" />

<!--
Timing: 0:40 · elapsed 8:55

Click “it” in the sentence or its matrix row. Point across the highlighted lower-triangle row: every earlier token is eligible to contribute.

Use Query, Keys, Values as plain-language roles. The matrix is causal: future positions are removed, but all past positions remain reachable.

Select “robot,” then switch CAUSAL ON to LOOK AHEAD. Its highlighted row now extends into future columns. Explain that this is useful for encoding or classification, but invalid during next-token generation because those future tokens do not exist yet.

Key line: “Full attention does not say every past token matters equally. It lets the model learn that decision.”

Transition: “FlashAttention preserves this exact picture and changes how the machine computes it.”
-->

---

<AttentionMechanismSlide mechanism="flash" />

<!--
Timing: 0:35 · elapsed 9:30

Start with the misconception: FlashAttention is not a sparse mechanism. The allowed connections are identical to full attention.

Click PROCESS NEXT TILE once or twice. Explain that the model works on small blocks in fast on-chip memory instead of repeatedly moving a giant intermediate matrix.

Click the “crossed” matrix row, then toggle LOOK AHEAD. The upper triangle appears while the tiled execution remains. Use this to separate the mask from the implementation.

Key line: “Same answer, same connections, less memory traffic.”

Transition: “To actually remove connections, we need a structural assumption.”
-->

---

<AttentionMechanismSlide mechanism="sliding" />

<!--
Timing: 0:35 · elapsed 10:05

Move the window slider from three tokens to five, then back. The diagonal band is the portion of history each query may inspect.

Connect the visual to language: syntax and local coherence are often nearby, so a moving local window can carry a lot of the workload cheaply.

State the failure clearly: an important name, instruction, or premise outside the window is unreachable without a global or recurrent path.

Transition: “A fixed window guesses that distance equals relevance. DeepSeek asks the content instead.”
-->

---

<AttentionMechanismSlide mechanism="dsa" />

<!--
Timing: 0:45 · elapsed 10:50

Follow the two-stage path. First, a lightweight learned indexer scores the history. Then only its top candidates enter the main attention operation.

Use “it” as the query. The example scores are deliberately illustrative; the point is the learned selection pattern, not these values.

Point out that DSA is content-dependent: a distant token can win, while an irrelevant nearby token can be skipped.

Nuance if asked: the selector still performs work across the sequence; the expensive main attention is reduced to the selected set.

Transition: “Qwen3-8B uses a documented compromise inside dense attention: keep many query heads, but share the stored context.”
-->

---

<QwenAttentionImagesSlide />

<!--
Timing: 1:40 · elapsed 12:30

Name the model carefully: Qwen3-8B, with a hyphen. It uses grouped-query attention: the middle point between giving every query head its own keys and values, and making every query head share one pair.

Start on the conversion image. On the left, MHA keeps separate key and value projections per head. In the middle, MQA shares one key and value projection across all query heads.

Point to the right: an existing MHA checkpoint can be adapted by mean-pooling its key and value heads before uptraining.

[click] The second image introduces GQA between those extremes. Qwen3-8B uses 32 query heads and eight key/value heads—four queries per shared KV group.

Land the trade-off: fewer KV heads mean a smaller cache and cheaper decoding, while multiple groups preserve more diversity than a single MQA head.

Transition: “The next image returns to Qwen 3.8 Max—the separate flagship benchmark snapshot supplied for this deck.”
-->

---

<QwenBenchmarkSlide />

<!--
Timing: 0:20 · elapsed 12:50

Let the image carry the slide. Do not read every bar.

Point out the breadth of the benchmark groups—software engineering, agents, and visual tasks—and say that this is performance evidence, not architecture evidence.

Transition: “Across all these mechanisms, the useful design question is not which name wins. It is which connections the task actually needs.”
-->

---

<PositionSlide />

<!--
Timing: 0:55 · elapsed 13:45

Give the answer directly: the simplest useful pattern depends on how much context the model must handle.

[click] Small context: use full attention. The complete interaction is affordable and easy to reason about.

[click] Medium context: keep full attention, but use FlashAttention to reduce memory traffic. Clarify that it does not change connectivity.

[click] Large context: use a hybrid—local routes for most tokens, with selected global context when the task needs it.

Transition: “That is where I think the field is. Here is where I think it goes next.”
-->

---

<FutureAttentionSlide />

<!--
Timing: 1:10 · elapsed 14:55

Frame this as a prediction, not a research proposal: “I think the next important paradigm will learn continuously and remember selectively.”

[click] Contrast today’s pipeline: retain all previous token states, perform an attention lookup, then produce a next-token distribution.

[click] Walk down the replacement stack in four verbs: update, store, retrieve, predict.

Online recurrence updates a compact streaming state rather than rebuilding the entire history.

Selective episodic memory stores only useful or surprising moments. Hierarchical retrieval searches summaries before expanding into detail.

Latent multi-step prediction trains the model to anticipate future internal states and outcomes—not only the immediate next token.

Land the prediction: “The output can still be a next-token distribution. What changes is how the model learns, remembers, and reaches the evidence.”

Transition: “The reading path shows the ideas that point us in that direction.”
-->

---

<ResourcesSlide />

<!--
Timing: 0:35 · elapsed 15:30

Do not read the bibliography. Explain that the first column follows memory mechanisms, the second follows attention at scale, and the third offers interactive ways to keep exploring.

[click] Mention that the links are live in the deck. Recommend the architecture gallery for side-by-side model comparisons and TokenTown for an intuitive walkthrough of how an LLM works.

Transition: “The whole path leaves us with one practical principle.”
-->

---

<div class="slide-shell outro-slide">
  <div class="closing-mark">20</div>

  <div class="outro-copy">
    <p class="eyebrow">THE TAKEAWAY</p>
    <h1>Spend attention<br>where it earns <span class="accent underline">context</span>.</h1>
    <p class="subtitle" v-click>Full when small. Expand when needed.</p>
  </div>

  <div class="footer-row">
    <span>Henry Ssenono · @henryhale</span>
    <span>Questions?</span>
  </div>
</div>

<!--
Timing: 0:30 · elapsed 16:00

Close with the title’s answer: “Use as much attention as earns its cost. Full when small; expand when needed.”

[click] Let the final line sit for a beat.

Invite questions. Reserve 4:00 for Q&A and end at 20:00.

If no question comes immediately, seed one: “Where in your own systems is context currently being discarded—or over-collected?”
-->
