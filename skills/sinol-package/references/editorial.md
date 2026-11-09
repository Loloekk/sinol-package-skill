# Editorial

The editorial explains the solution to the jury: what the task is, a quick
reminder of the solution, then the full explanation. Write it in Polish;
translate only if asked.

## When

| situation | what to do |
|---|---|
| new package | write it by default, after `verify`, because it quotes measured results; the user can opt out in the plan |
| existing package without one (new work, modernize) | ask once at the start of the work: "add an editorial?" |
| only answering a question about a package, no changes | don't ask |
| review | report that it is missing and ask at the end; don't add it silently |
| package has one and the change affects it (model, subtasks, points, tests) | update it as part of the work and say so in the report |
| the user declined | don't add it and don't ask again in this session |

## Files

`doc/<id>opr.tex`, next to the statement. There is no separate directory.

- `sinol-make doc` builds every `.tex` in `doc/`, so `doc/<id>opr.pdf` needs no
  extra step. PDFs in `doc/` are ignored by git.
- `sinol-make export` copies `doc/` as a whole, so the editorial goes into
  `<id>.tgz` for sio2, also through `contest.py export-all` and the release.
- **Never put it in `attachments/` or `dlazaw/`.** Every file in `attachments/`
  becomes a problem attachment, and `dlazaw/` is packed for contestants. The ocen
  archive holds only tests, so it can't pick the editorial up by itself.
- Use a plain `article`, not `sinol.cls` with the `opr` option, which requires
  a `Testy` section and its own macros. `logo.png` is already in `doc/`.
- **No `tikz`.** The release workflow installs a minimal LaTeX without it, so a
  package using `tikz` builds locally but fails the release, and with it the
  whole `<id>.tgz`. Draw figures with the built-in `picture` environment.

The audience is the jury. `<id>.tgz` already contains `prog/` with all
solutions, so the editorial may name solution files.

## Structure

All three parts are required, in this order:

1. **Zadanie w skrócie.** What the statement says, without the story: the task,
   the functions or the input and output, limits, scoring, the subtask table.
2. **Omówienie w punktach.** 5–10 bullets with a bold label each. Someone who
   knows the task should be able to rebuild the whole solution from them.
3. **Pełne omówienie:**
   - key observations;
   - every subtask: what works and what it costs;
   - the model solution, step by step;
   - correctness;
   - complexity, or the number of queries in an interactive task;
   - a table of approaches with their cost and points ("Co ile daje"), from
     the package's solutions;
   - other approaches, if the package has them;
   - typical mistakes.

Add a figure when it shows a structure better than words. An easy task needs
1–2 pages, but keeps all three parts.

## Stating only what is known

- Say which kind each claim is: proved, measured, or a rough estimate. If there
  is no proof of a bound, write that there is none.
- Numbers come only from real runs. Points come from `sinol_expected_scores`.
- When a measurement is converted to "c·n", divide by the `n` of the test it
  came from, not by the maximal `n`.
- Compute worked examples and figures; don't draw them by eye.

## Checks before reporting

- `sinol-make doc` succeeds, and `.cache/doc_logs/<id>opr.log` has no `Overfull`.
- `.cache/doc_logs/<id>opr.fls` loads only what `.github/workflows/release.yml`
  installs: a local build proves nothing if the local TeX Live is the full one.
- Render **every** page and look at it **after the last change**
  (`pdftoppm -r 80 -png doc/<id>opr.pdf <scratchpad>/opr`).
- The subtask table matches `config.yml`, and every named solution exists in `prog/`.
- Medium / hard task: give a subagent the editorial and the code, not your
  conclusions, and ask for claims that disagree with the code or have no support.
- After export, `tar tzf <id>.tgz` lists `doc/<id>opr.pdf`, and
  `attachments/` holds no editorial.
- In the report, say what is proved and what is only measured.

## Skeleton

```latex
\documentclass[11pt,a4paper]{article}
\usepackage[utf8]{inputenc}
\usepackage[T1]{fontenc}
\usepackage{lmodern}
\usepackage[polish]{babel}
\usepackage{amsmath}
\usepackage{graphicx}
\usepackage{color}

\setlength{\textwidth}{16.6cm}
\setlength{\textheight}{24.5cm}
\setlength{\oddsidemargin}{-0.2cm}
\setlength{\topmargin}{-1.5cm}
\setlength{\parindent}{0pt}
\setlength{\parskip}{0.6em}
\newenvironment{lemat}{\par\textbf{Lemat.}\itshape\ }{\par}
\newenvironment{proof}{\par\textit{Dowód.}\ }{\hfill\rule{1ex}{1ex}\par}

\begin{document}

\begin{minipage}[b]{0.72\textwidth}
  {\Large\bfseries Tytuł -- omówienie rozwiązania}\par
  \vspace{0.2em}
  Nazwa konkursu, zadanie \texttt{ABC}
\end{minipage}%
\hfill
\includegraphics[height=1.7cm]{logo}
\vspace{0.4em}\hrule\vspace{0.8em}

\section*{Zadanie w skrócie}
% The task without the story, limits, scoring, subtask table.

\section*{Omówienie w punktach}
\begin{itemize}
  \item \textbf{Etykieta:} jedno lub dwa zdania.
\end{itemize}

\section*{Pełne omówienie}
\subsection*{Obserwacje}
\subsection*{Podzadania częściowe}
\subsection*{Rozwiązanie wzorcowe}
\subsection*{Poprawność}
\subsection*{Złożoność}
\subsection*{Co ile daje}
\subsection*{Inne podejścia}
\subsection*{Typowe błędy}

\end{document}
```
