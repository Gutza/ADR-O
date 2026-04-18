## ADR-O outreach article — feedback assumptions (for assistants)

**Context.** The Medium piece is outreach for ADR-O (see [README](/README.md) and [MANIFESTO](/MANIFESTO.md)). The author has already iterated on structure and audience many times; avoid recycling generic blogging advice that contradicts choices already made; evaluate your own suggestions from first principles. Here are some a few examples:

**Audience.** Readers are **self-selected**: software teams, already familiar with ADRs (and often knowledge graphs). They clicked a title about ADRs and KGs. Do **not** frame feedback around a “broad non-technical Medium audience” or imply they need hand-holding on basics.

**Title.** The literal first words (**“Software Teams”**) already signal org-level, team practice. Do not recommend a separate “who is this for” paragraph for **audience signaling** unless the draft is genuinely unclear—**not** to name “platform vs architect vs doc lead” or “who owns the ADL” as a prerequisite for understanding.

**Structure.** After the H1/H2, the **first paragraph** states the thesis, includes a **literal time budget** (e.g. 15 minutes), and sets expectations. Content after a horizontal rule (`---`) will literally live under a separator (think `<hr>`), visually indicating we're entering the body of the copy and a change of pace. Do not criticize that block as “slow to the thesis” without acknowledging what already appeared above the rule.

**“Magically!” / paragraph break.** The sentence ending in “magically!” is a deliberate short beat across the paragraph boundary; the **very next paragraph** opens with *It’s not that the team doesn’t understand what they’re doing while they’re doing it*—the critique is **retrospective illusion and lost traceability**, not the quality or legitimacy of specs or upstream work. Do not recommend softening that line for “PM-adjacent” readers unless the draft genuinely reads as attacking product or spec practice; the guardrail is **immediate**, not a delayed callback. Skim-risk (someone who stops after “magically!”) is an edge case; for the intended audience, the setup–punchline pairing is intentional.

**The Wikipedia quote** seems busy in markdown, but when you render the links, the actual content looks like this to the reader:  
  ```
  Wikipedia's collaborative editorial process is not conducive to producing memorable quotes—but *suffering transcends process*:

    Software architecture design is a *wicked problem*, therefore architectural decisions are difficult to get right. – *anonymous*, *Architectural decision*

  Not only are architectural decisions difficult to get right, but the industry is prone to the infamous "build it now, fix it later" mindset under delivery pressure.
  ```