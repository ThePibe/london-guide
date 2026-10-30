# London guide

A Quarto project: one content document (`london.qmd`) rendered to many formats, plus a separate dashboard (`london-dashboard.qmd`). Both read the same CSVs in `data/`.

## Commands

```bash
export QUARTO_PYTHON=$PWD/.venv/bin/python   # needed: .venv holds pandas/plotly/matplotlib/ipykernel; system python3 has no pandas
quarto render                                 # builds the whole website into _site/ (every *.qmd in the root)
quarto render london.qmd --to docx            # one format (html, docx, typst, pptx, epub, revealjs)
quarto render london-dashboard.qmd            # dashboard only
quarto preview --no-browser --port 4321      # serve the site; pick a free port, 4200 is often taken
```

There are no tests or linters. "Passing" means `quarto render` finishes with no errors.

## Architecture

- `_quarto.yml` is a `website` project (output in `_site/`, ignored by git along with `.quarto/`) that renders only `*.qmd`, so the Markdown docs (`AGENTS.md`, READMEs) are not built. The navbar links Guide (`london.qmd`), Dashboard, and a Downloads menu pointing at `london.docx/.pdf/.pptx/.epub` and `london-slides.html`; those files only exist after `london.qmd` renders every format, so rename an `output-file` there and the menu breaks. `index.qmd` is the landing page. Its `format: html` block (cosmo theme, toc) applies to `london.qmd`; the dashboard declares `format: dashboard` itself.
- `london-email.qmd` (docx), `london-flyer.qmd` and `london-social.qmd` (typst PDFs) are standalone deliverables: they build into `_site/` but are not in the navbar.
- `london.qmd` declares all its output formats in its own front matter: html, docx, typst (PDF), pptx, epub, and revealjs (written to `london-slides.html`). The html format's `format-links` produces the download links to the others. Add or remove a format there, not in `_quarto.yml`.
- Content is shared across formats, so it must work in all of them: image alt text, captions with licence credits, and a Python-computed inline values section (`` `{python} ...` ``) rather than hard-coded numbers.
- `london-dashboard.qmd` loads `data/attractions.csv` and `data/visits.csv` in a hidden cell, then builds value boxes and Plotly charts. `london.qmd` uses matplotlib for its figure; the dashboard uses Plotly.
- Build outputs go to `_site/`; edit the `.qmd`, not the outputs. Older `*.html/.docx/.pdf/.pptx/.epub` and `*_files/` copies in the project root are stale leftovers from before the website setup.
- Do not run two `quarto render`/`preview` processes at once in this folder: they race on moving files into `_site/` and fail.

## Data (`data/`)

Real ALVA visitor counts; `data/README.md` has the column definitions and the source-page IDs. Gotchas when charting or comparing:

- `attractions.csv` is the top ten London ALVA members only, not every London sight. `admission` is the label for `charge_code` (1 Paid, 2 Free, 3 Free (pay parking), 4 Free (paid exhibitions), 5 Other). "Free" counts, such as the dashboard's value box, use `admission.startswith("Free")`.
- `visits.csv` is long format (one row per attraction per year, 2015–2025, five attractions only). 2020–2021 are COVID years and are not comparable. From 2024 the Natural History Museum count includes its gardens, so 2024–2025 are not like-for-like with earlier years. `alva_name` varies by year; join on `attraction`.
- Every data row carries `source`, `url` and `retrieved`. Keep these when adding rows.

## Images

Every file in `images/` needs a matching entry in `images/ATTRIBUTION.md` (photographer, licence, Commons file page) and a visible credit line where it is used in `london.qmd`.
