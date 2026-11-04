# Homework repo

Each course lives in its own top-level folder (`Network Optimization/`, `Network 2/` (Network
Optimization II), `Linear Programming/`, `Discrete Math/`, `Convex Optimization/`), with one
subfolder per problem set (`hw1/`, `hw2/`, ...) containing a single `hwN.tex` plus its build
artifacts. Only `.tex` files are tracked by git; everything else (`.pdf`, `.aux`, `.log`, ...) is
gitignored.

## Workflow: turning a problem-set PDF into a blank `hwN.tex`

The user drops a problem-set PDF in the repo root (e.g. `47835-ps2-2026.pdf`), or straight into
the target folder (e.g. `Convex Optimization/hw1/assign1.pdf`), and asks for a scaffolded `.tex`.
Steps:

1. **Read the PDF.** `pdftotext -layout <file>.pdf -` gives the full text. Check `pdfinfo` for
   page count. If the sheet has figures, note them — they must be redrawn in TikZ or omitted with
   a comment, since `pdftotext` will not extract them. `pdftotext` also scrambles display math
   (matrices, limits under `\max`/`\sum`, square roots), and sheets typeset with Typst come out
   in Unicode math italics, so render the pages (`pdftoppm -r 130 -png <file>.pdf page`) and read
   every formula off the images.

2. **Create the folder.** `<Course>/hw<N>/`, matching the numbering on the sheet (Homework 2 →
   `hw2/hw2.tex`).

3. **Copy the preamble from the previous homework in the same course**, verbatim: every line
   above `\title`, e.g. `sed '/^\\title{/,$d' hw1/hw1.tex > hw2/hw2.tex`. Do not stop at the last
   `\usepackage` line — Linear Programming and Discrete Math load `hyperref`/`cleveref` near the
   top, so their whole macro block comes after it. Do not invent a new preamble — the courses
   share a near-identical macro block (`\calA`--`\calZ`, theorem environments, `\solution`), and
   the per-course differences are deliberate.

   **New course** (no earlier homework in its folder): merge the preambles of the latest homework
   in every other course, keeping every line that appears in any of them, once and verbatim (e.g.
   `tikz` comes only from Discrete Math; `dsfont`, `etoolbox`, `\argmax`, `\One` only from Linear
   Programming). Keep the Linear Programming order, with `hyperref`, `cleveref` and `etoolbox`
   loaded before the theorem block, since the `\crefname` and `\AtBeginEnvironment` lines need
   them. `Convex Optimization/hw1/hw1.tex` was built this way. From the repo root, this should
   print nothing:
   `comm -23 <(for f in */hw*/hw*.tex; do sed '/^\\title{/,$d' "$f"; done | sort -u) <(sed '/^\\title{/,$d' <Course>/hw1/hw1.tex | sort -u)`.

4. **Set the metadata.** `\title{<course number>: <Course Name> \\ Homework <N>}` (Homework even
   when the sheet says Assignment or Problem Set), `\author{Poon Tze-Yang}`, `\date{...}` (use the
   due date if the sheet states one, otherwise the month). Drop any `\bibliography` lines from the
   previous file unless a `refs.bib` actually exists. Header details from the sheet go on a
   `\noindent` line after `\maketitle`: `\noindent Instructor: Wesley Pegden`,
   `\noindent Fall 2026 Mini-1`.

5. **Transcribe every question, verbatim, with no solutions.** The house format is one
   `enumerate` block per question, closed immediately, followed by a bare `\solution` and blank
   space to write into:

   ```latex
   \begin{enumerate}[resume=q]
   \item <question text>
   \end{enumerate}

   \solution


   ```

   - The *first* question opens the named series with `\begin{enumerate}[series=q]`; every later
     one uses `\begin{enumerate}[resume=q]` so numbering continues across the interleaved
     solutions.
   - Use the named series, never a bare `[resume]`. Because each question's list is closed
     immediately, solution text sits *outside* any list, so any `enumerate` written inside a
     solution is itself a level-1 list sharing the `enumi` counter. A bare `[resume]` resumes from
     whichever level-1 list ended most recently, so one list in one solution silently shifts every
     subsequent question number (and desynchronises the hard-coded `\item[7.*]` labels below).
     `resume=q` resumes from the question series regardless of what lists appear in between.
   - Questions marked optional on the sheet keep their marker in an explicit label:
     `\item[7.*]`, `\item[10.**]`. Note these must be hard-coded, so they have to match the
     question's actual position.
   - Subparts (a), (b), (c) are a plain nested `\begin{enumerate}` — the default `article` label
     is already `(a)`, so do not pass a `label=` option. When the sheet labels them differently,
     mirror it: `[label=(\roman*)]`, `[label=\arabic*.]`, or
     `[label=\textbf{Part \Alph*.}, leftmargin=*]` (without `leftmargin=*` a wide label hangs out
     into the margin).
   - Question titles and point values open the item in bold, with the list number standing in for
     the sheet's own `Q1.`: `\item \textbf{[15 points]} ...`,
     `\item \textbf{Inner Products and Dual Norms (10 points)}`. Extras inside a question
     ("Extra, not to be handed in:", "Challenge (Zero Credits):") are kept with their bold lead-in
     where the sheet has them: as a subpart if it lists them as one, otherwise as a paragraph
     after the subparts.
   - Keep the preamble note about optional questions ("Students in the OR and ACO programs...")
     if the sheet has one.
   - Transcribe prose faithfully: convert math to LaTeX, use ``...'' for quotes, `\setminus` for
     set difference, and align multi-line constraint systems with `align*`.

6. **Compile and check.** `latexmk -pdf -interaction=nonstopmode hw<N>.tex`, then verify the
   numbering round-trips: `pdftotext -layout hw<N>.pdf - | grep -nE "^\s*[0-9]+\."` should list
   every question with the right number and star markers (nested `1.`-style subparts show up too,
   further indented).

7. **Delete the source PDF from the repo root** once the transcription is verified. It is
   gitignored, so this is not recoverable — only do it after step 6 passes. (Some older folders,
   e.g. `Discrete Math/hw1/hw1_orig.pdf`, instead keep the original alongside the `.tex`, and a PDF
   the user dropped straight into the target folder, e.g. `Convex Optimization/hw1/assign1.pdf`,
   stays there; ask if unsure which the user wants.)

Solutions are written by the user afterwards, under each `\solution`. When a solution needs
structure, existing files use `\paragraph{(a)}` for subparts and `theorem`/`lemma`/`proof`
environments for anything longer.

`AGENTS.md` is an identical copy of this file; keep the two in sync.
