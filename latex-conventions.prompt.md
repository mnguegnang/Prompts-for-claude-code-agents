# LaTeX Conventions

> **Read before drafting any prose artifact:** `~/.claude/agents/Scientific-writing-guide-anti-AI-text.md` — the
> human-centric scientific writing manual. It governs the *prose* inside every `.tex` file this
> pipeline produces (lectures, literature reviews, papers, defense slides); this file governs the
> *mechanics* (packages, citations, compilation). Apply the manual while drafting, not as a
> cleanup pass afterwards. Its hard bans: no em-dashes, digits for all numbers, sentence case
> headers, and none of "delve", "realm", "harness", or "tapestry". A `PreToolUse` writing guard
> surfaces the rules on your first `.tex` write of a session.

Shared LaTeX compilation and citation standards for all agents producing `.tex` artifacts.

## Compilation

Use `pdflatex -interaction=nonstopmode <filename>.tex` via the Bash tool, with `biber` between passes (see § Citation Strategy). If compilation fails, read the `.log` file, correct the `.tex` syntax, and recompile autonomously until successful.

## Pre-delivery Checks

Run before handing any compiled PDF to the user:

- **Acronyms at first use.** Grep the `.tex` for capitalised acronyms (`grep -oE '\b[A-Z]{2,}\b'`)
  and confirm each is expanded at its first use in the body. A back-matter glossary does not count.
- **Markdown artefacts.** After any scripted edit (Python heredoc, `sed`), grep the `.tex` for
  `**`, `##`, and `](`, then extract the PDF text (`pdftotext`) and confirm 0 hits. Markdown bold
  renders in LaTeX as literal asterisks and raises no compile error.

## Layout — single column, always

Lectures, literature reviews, code lectures, and build guides are **one column**:
`\documentclass[11pt,a4paper,onecolumn]{article}`. Never `twocolumn`, `IEEEtran`, a conference
class, or `multicol` — even when a review follows a venue's section structure. Footnote
citations and code listings need the full text width.

## Required Packages

Include at minimum: `amsmath`, `hyperref`, and the `biblatex` setup below. Add `algorithm2e` only when source papers explicitly feature pseudocode.

## Citation Strategy — author–year footnotes with locators

Every claim is cited in a footnote at the bottom of the page, rendering as
**"Name et al. (Year, locator)"** — e.g. "Vaswani et al. (2017, p. 12)", "Goodfellow et al.
(2016, pp. 150–152)", "Vaswani et al. (2017, § 3.2)". A citation with no locator is incomplete.

```latex
\usepackage[style=authoryear,maxcitenames=2,uniquelist=false,backend=biber]{biblatex}
\addbibresource{refs.bib}
\newcommand{\fnsrc}[2][]{\footnote{\textcite[#1]{#2}.}}
% ...as shown.\fnsrc[12]{vaswani2017}   pages: number only, biblatex adds "p."/"pp."
% ...as shown.\fnsrc[\S~3.2]{vaswani2017}   sections: pass the locator text
% end the document with \printbibliography
```

- Use `\textcite` inside `\footnote`, not `\footcite` — `\footcite` drops the parentheses
  ("Vaswani et al. 2017, p. 12"). [L1 §3.9.1–3.9.2, p. 110]
- Pass bare page numbers; biblatex prefixes "p."/"pp." itself. [L1 §3.15.3, p. 152]
- Organisation-authored web sources with no date cite as "Google (n.d., § Procedures)".
- Build: `pdflatex` → `biber` → `pdflatex` → `pdflatex`.

[L1] Lehman et al. (2023), *The biblatex Package*, v3.19 — https://ctan.org/pkg/biblatex

## Citation Rules

- NO inventing or guessing citations. Rely strictly on credible, verified sources.
- Use direct download links (`pdf_url`), not landing pages.
- Cite code patterns from PEP standards, authoritative papers, and industrial guides.
- Every `.bib` entry carries a `url` field with the direct link; the footnote itself stays short (name, year, locator).
