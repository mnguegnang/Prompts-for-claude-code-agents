# LaTeX Conventions

Shared LaTeX compilation and citation standards for all agents producing `.tex` artifacts.

## Compilation

Use `pdflatex -interaction=nonstopmode <filename>.tex` via the Bash tool. Run **twice** for cross-references. If compilation fails, read the `.log` file, correct the `.tex` syntax, and recompile autonomously until successful.

## Required Packages

Include at minimum: `amsmath`, `hyperref`, `marginnote`. Add `algorithm2e` only when source papers explicitly feature pseudocode.

## Citation Strategy

- **With `.bib` file:** Use `\bibliography{}` and `\cite{}` commands. Run `bibtex` between the two `pdflatex` passes.
- **Without `.bib` file:** Use `\sourcenote{}` margin notes.

## Source Note Definition

```latex
\newcommand{\sourcenote}[1]{\marginpar{\footnotesize #1}}
```

Annotate every major claim with:
```latex
\sourcenote{Section X.Y, \href{DIRECT_PDF_URL}{Author et al., Year}}
```

## Citation Rules

- NO inventing or guessing citations. Rely strictly on credible, verified sources.
- Use direct download links (`pdf_url`), not landing pages.
- Cite code patterns from PEP standards, authoritative papers, and industrial guides.
- Use footnotes or margin notes with exact source links.
