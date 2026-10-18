# Teaching Methodology — learn by doing (shared — PI Phase 3, Coder Steps 3 and 7)

How the agents teach when the user's stated objective includes **learning**. The user learns by
building the real project, has little time, and writes only small pieces of code. The aim is
mastery of each skill and its core principles, one component at a time. Each rule cites its source
in the table at the end; rules without a tag are the user's own requirements.

## 1. Three artifacts, three jobs — never mixed

| Artifact | Job | Contains | Never contains |
|---|---|---|---|
| **Codebase** | Reference | Docstrings that state the contract (args, returns, side effects, exceptions — not how the code works, not the signature restated) [T1, T2]; comments only for non-obvious logic, saying *why* [T3, T4] | Tutorials, lecture text, line-by-line narration |
| **Build guide** `[Topic]_Build_Guide.tex` | Tutorial | Steps, expected results, how to test each component, the user's exercises, minimal explanation linked to the lecture [T5] | Long theory |
| **Lecture** `[Topic]_Lecture.tex` / `_Code_Lecture.tex` | Explanation | The why: concepts, principles, trade-offs, sources | Step-by-step procedures |

The guide, the lecture, and the reference solutions live **outside the repository**, in a sibling
directory `<repo>-learning/`. Nothing teaching-related is committed to the project.

## 2. The per-component loop: Learn → Do → Verify

Split the project into building blocks, each independently verifiable, and keep the gap between
learning and doing to minutes [P1]. Order them as a progression of harder tasks inside the real
project [M1 p. 45]. For each component, in this order:

1. **Recall.** 2 or 3 questions on earlier components, answered without looking [D1 pp. 12–13].
2. **Open.** One sentence of context, then *"By the end of this component you will be able to
   <verb> <content>"*. Each objective is a verb plus content [K1 p. 213] and maps to the check that
   tests it [P2].
3. **Pre-train.** Name the parts and the job of each before explaining how they work together [MM1 Table 1].
4. **Worked example.** Show one fully worked, annotated example of the new skill [S1 p. 267; KSC1 p. 80],
   with a short plain-English **subgoal label** above each group of lines; reuse the same labels
   across components [MG1 pp. 2–3].
5. **Predict, run, investigate, modify.** The user predicts the output, runs it, compares, and
   changes one thing before writing anything new [R1 p. 2; P3].
6. **The user's piece — a completion problem.** The agent writes the component with one small gap
   (a function body, a condition, a query) for the user to fill [S1 p. 267]. A few lines, never a
   whole module. Widen the gap on later components.
7. **Verify.** Each guide step states the action first and the result second, shows the exact
   expected output, and names the likely sign of failure [G1; T5]. The component ships a test that
   returns pass/fail **with a specific reason** [P4]. Offer a hint before the solution; keep a
   reference solution in `<repo>-learning/solutions/` so one failure never blocks the next
   component [P5].
8. **Explain gate.** The user says in 1 or 2 sentences what the component is *for* — relational,
   not line by line [F1 §2]. Then a short knowledge check whose wrong options are plausible
   misconceptions, each option explained [P6].
9. **Move on** only when the test passes and the explain gate is met. Set the next component's
   guidance from this one's result: once a skill is solid, replace the worked example with a plain
   problem and withdraw coaching [S1 p. 276; KSC1 p. 75; M1 p. 49].

## 3. Sizing

- 1 or 2 new concepts per unit, readable in 3 to 5 minutes; state the estimated time on every unit;
  a component fits one sitting [P7].
- Segment lectures into short parts, and end each section with questions rather than a summary
  [MM1 Table 1; D1].

## 4. Writing

- Plain language: address the reader as "you", active voice, present tense, one idea per sentence [W1; W2].
- Define or link every unfamiliar term at first use, in the body [W3].
- Procedures: a numbered list, one instruction per step, each a full sentence starting with an
  imperative verb; split long procedures [W4].
- **Code blocks:** context and a title before; after, what it does, the expected result, and 2 small
  changes to try. Numbered callouts when a block introduces 3 or more ideas. Show what happens when
  it fails [P8].
- Cite every argument in a footnote as "Name et al. (Year, locator)" — mechanics in
  `latex-conventions.prompt.md` § Citation Strategy.

## 5. Chat economy

Make the point and stop. Explanations live in the guide and lecture, not the chat. In chat: the
component just finished, the test result, the file path to read, and the one next action. No
repetition, no summaries of what a file already says.

## 6. Code hygiene during the loop

A completion gap is a stub that raises `NotImplementedError("exercise")`, and its test carries
`@pytest.mark.exercise` until the user's code passes; the agent then removes the marker. The
`exercise` marker is registered in `setup.cfg` and excluded by the `Stop` hook; while a gap is
pending, run the suite with `-m "not live and not exercise"` and the user's check with `-m exercise`. **No `exercise`
marker or stub may remain at the STEP 8 handoff.**

## Sources

| ID | Citation | URL | Where |
|---|---|---|---|
| S1 | Sweller et al. (2019) | https://link.springer.com/content/pdf/10.1007/s10648-019-09465-5.pdf | p. 267 (completion problems); p. 276 (fading) |
| KSC1 | Kirschner et al. (2006) | https://www.davidlewisphd.com/courses/EDD8121/readings/2006-Kirschner_et_al.pdf | p. 75 abstract; p. 80 (worked-example effect) |
| MG1 | Margulieux et al. (2020) | https://stemeducationjournal.springeropen.com/counter/pdf/10.1186/s40594-020-00222-7.pdf | pp. 2–3 (subgoal labels) |
| R1 | Raspberry Pi Foundation (2023) — PRIMM summary of Sentance et al. (2019) | https://static.raspberrypi.org/files/curriculum/quickreads/11-Pedagogy_Summary_PRIMM_V4_2023.pdf | p. 2, "The five stages of PRIMM" |
| F1 | Fowler et al. (2021) | https://zilles.web.engr.illinois.edu/papers/how_to_eipe_icer_2021.pdf | §2, p. 2 |
| K1 | Krathwohl (2002) | https://ocw.metu.edu.tr/mod/resource/view.php?id=4265 | p. 213 |
| M1 | Merrill (2002) | https://mdavidmerrill.files.wordpress.com/2019/04/firstprinciplesbymerrill.pdf | p. 45 (problem progression); p. 49 (diminishing coaching) |
| D1 | Dunlosky (2013) | https://files.eric.ed.gov/fulltext/EJ1021069.pdf | pp. 12–13 |
| MM1 | Mayer & Moreno (2003) | https://www.uky.edu/~gmswan3/544/9_ways_to_reduce_CL.pdf | Table 1, p. 46 (segmenting, pre-training) |
| P1 | Neo4j GraphAcademy (n.d.); Microsoft (n.d.); IBM Skills Network (n.d.) | github.com/neo4j-graphacademy/courses docs/workshops/HOW-TO-BUILD-WORKSHOPS.md; github.com/microsoft/aed-learn-template module-design-template.md; author.skills.network/docs/guided-projects/quick-start | "Learn → Do → Verify", "The Building Block Pattern"; "Chunk your content into subtasks"; "Plan the Structure and Flow" |
| P2 | Neo4j GraphAcademy (n.d.); Microsoft (n.d.); IBM Skills Network (n.d.) | courses repo .claude/skills/lesson-author/SKILL.md; module-design-template.md; guided-projects quick-start | "Phase 3: Write Opening"; subtask assessment column; "Set Clear Learning Objectives" |
| P3 | Neo4j GraphAcademy (n.d.); Coursera (n.d.) | courses repo how-we-teach/…/2-pedagogy/lesson.adoc; https://blog.coursera.org/coursera-white-paper-details-the-pedagogy-underlying-guided-projects-on-coursera/ | "PRIMM"; worked examples and challenge mode |
| P4 | Neo4j GraphAcademy (n.d.); Google Skills (n.d.); SAP (n.d.) | HOW-TO-BUILD-WORKSHOPS.md; https://skills.google/catalog_lab/4676; https://developers.sap.com/tutorials/tutorial-first-steps.html | "Pattern 5: Automated Verification"; "Check my progress"; "Start to complete this tutorial" |
| P5 | Neo4j GraphAcademy (n.d.) | HOW-TO-BUILD-WORKSHOPS.md | "Pattern 4: Snapshot Resilience" |
| P6 | Microsoft (n.d.); Neo4j GraphAcademy (n.d.) | aed-learn-template README.md; courses repo .claude/skills/quiz-author/SKILL.md | "Knowledge Checks"; "Phase 1", "Phase 6: Write Solutions" |
| P7 | Neo4j GraphAcademy (n.d.); Microsoft (n.d.); IBM Skills Network (n.d.) | lesson-author SKILL.md; https://learn.microsoft.com/en-us/training/support/learn-content-types; guided-projects quick-start | "Overview" (1–2 concepts, 3–5 min); "Modules"; "What is a Guided Project" |
| P8 | Neo4j GraphAcademy (n.d.) | lesson-author SKILL.md; pedagogy lesson.adoc | "Code Block Pattern", "Numbered Annotations"; "Model everything" |
| T1 | Google (n.d.) — Python Style Guide | https://google.github.io/styleguide/pyguide.html | §3.8.3 |
| T2 | Goodger & van Rossum (2001) — PEP 257 | https://peps.python.org/pep-0257/ | "One-line Docstrings", "Multi-line Docstrings" |
| T3 | Google (n.d.) — Python Style Guide | https://google.github.io/styleguide/pyguide.html | §3.8.5 |
| T4 | van Rossum et al. (2001) — PEP 8 | https://peps.python.org/pep-0008/ | "Comments", "Inline Comments" |
| T5 | Procida (n.d.) — Diátaxis | https://diataxis.fr/tutorials/ ; https://diataxis.fr/reference/ | "Ruthlessly minimise explanation", "Maintain a narrative of the expected"; "Describe and only describe" |
| G1 | Google (n.d.) — developer documentation style guide | https://developers.google.com/style/procedures | "Steps with results or justifications" |
| W1 | Digital.gov (n.d.) — plain language | https://digital.gov/guides/plain-language/writing/ | "Use the active voice", "Use the present tense" |
| W2 | Google (n.d.) — Technical Writing One | https://developers.google.com/tech-writing/one/short-sentences | "Focus each sentence on a single idea" |
| W3 | Google (n.d.) — Technical Writing One | https://developers.google.com/tech-writing/one/words | "Define new or unfamiliar terms" |
| W4 | Microsoft (n.d.) — Writing Style Guide | https://learn.microsoft.com/en-us/style-guide/procedures-instructions/writing-step-by-step-instructions | "Multiple-step procedures", "Tips for writing steps" |
