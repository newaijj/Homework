# Homework repo

Each course lives in its own top-level folder (`Network Optimization/`, `Linear Programming/`,
`Discrete Math/`), with one subfolder per problem set (`hw1/`, `hw2/`, ...) containing a single
`hwN.tex` plus its build artifacts. Only `.tex` files are tracked by git; everything else
(`.pdf`, `.aux`, `.log`, ...) is gitignored.

## Workflow: turning a problem-set PDF into a blank `hwN.tex`

The user drops a problem-set PDF in the repo root (e.g. `47835-ps2-2026.pdf`) and asks for a
scaffolded `.tex`. Steps:

1. **Read the PDF.** `pdftotext -layout <file>.pdf -` gives the full text. Check `pdfinfo` for
   page count. If the sheet has figures, note them — they must be redrawn in TikZ or omitted with
   a comment, since `pdftotext` will not extract them.

2. **Create the folder.** `<Course>/hw<N>/`, matching the numbering on the sheet (Homework 2 →
   `hw2/hw2.tex`).

3. **Copy the preamble from the previous homework in the same course**, verbatim, up to and
   including the last `\usepackage` line. Do not invent a new preamble — the courses share a
   near-identical macro block (`\calA`--`\calZ`, theorem environments, `\solution`), and the
   per-course differences are deliberate. For Network Optimization that is
   `sed -n '1,69p' hw1/hw1.tex > hw2/hw2.tex`.

4. **Set the metadata.** `\title{<course number>: <Course Name> \\ Homework <N>}`,
   `\author{Poon Tze-Yang}`, `\date{...}` (use the due date if the sheet states one, otherwise the
   month). Drop any `\bibliography` lines from the previous file unless a `refs.bib` actually
   exists.

5. **Transcribe every question, verbatim, with no solutions.** The house format is one
   `enumerate` block per question, closed immediately, followed by a bare `\solution` and blank
   space to write into:

   ```latex
   \begin{enumerate}[resume]
   \item <question text>
   \end{enumerate}

   \solution


   ```

   - The *first* question uses `\begin{enumerate}` (no `[resume]`); every later one uses
     `\begin{enumerate}[resume]` so numbering continues across the interleaved solutions.
   - Questions marked optional on the sheet keep their marker in an explicit label:
     `\item[7.*]`, `\item[10.**]`. Note these must be hard-coded, so they have to match the
     question's actual position.
   - Subparts (a), (b), (c) are a plain nested `\begin{enumerate}` — the default `article` label
     is already `(a)`, so do not pass a `label=` option.
   - Keep the preamble note about optional questions ("Students in the OR and ACO programs...")
     if the sheet has one.
   - Transcribe prose faithfully: convert math to LaTeX, use ``...'' for quotes, `\setminus` for
     set difference, and align multi-line constraint systems with `align*`.

6. **Compile and check.** `latexmk -pdf -interaction=nonstopmode hw<N>.tex`, then verify the
   numbering round-trips: `pdftotext -layout hw<N>.pdf - | grep -nE "^\s*[0-9]+\."` should list
   every question with the right number and star markers.

7. **Delete the source PDF from the repo root** once the transcription is verified. It is
   gitignored, so this is not recoverable — only do it after step 6 passes. (Some older folders,
   e.g. `Discrete Math/hw1/hw1_orig.pdf`, instead keep the original alongside the `.tex`; ask if
   unsure which the user wants.)

Solutions are written by the user afterwards, under each `\solution`. When a solution needs
structure, existing files use `\paragraph{(a)}` for subparts and `theorem`/`lemma`/`proof`
environments for anything longer.
