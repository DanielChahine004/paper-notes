# CLAUDE.md

The thesis's paper notes (`src/paper-notes/main.tex`), built with Tectonic via
pixi, and the map drawn from them (notesmap, `tools/notesmap`). `README.md`
covers setup and the map; this file is the conventions. The thesis itself is in
the phd-thesis repo.

## Build

`pixi run build` → `build/main.pdf`. `src/shared/` holds `units.tex` (a copy of
the thesis's unit shorthands; keep the two in step by hand) and
`references.bib` (a copy of thesis-papers' bibliography that `pixi run sync`
refreshes), found through `-Z search-path=src/shared`.

`pixi run notes-check` lints the notes (missing images, dangling refs,
unplaced programmes, every `\doi{}` against the bibliography, every `\claim`
against its annotation); CI runs it with `--gaps --strict`.

## Source formatting

- **Hard tabs**, one per nesting level. **One sentence per line, no hard wrap.**
- **UTF-8 without a BOM, LF endings** for `src/paper-notes/**.tex`: they are
  edited live in JupyterLab (`pixi run notes-edit`), which always writes LF.
- **Units:** `\keV{511}`, `\mm{20}`, `\ps{200}` — not raw `\qty`.
- British spelling (-ise, -isation).

## Paper PDFs

`papers/` is a git-ignored clone of the private `thesis-papers` repo, a
[paperstore](https://github.com/DanielChahine004/paperstore) store:
`references.bib` (the bibliography), `<BibKey>.pdf`, `records.json` (each
DOI's doi.org record) and `annotations/<BibKey>.json`. `pixi run papers-pull`
(and every `pixi run sync`) pulls it and refreshes `src/shared/references.bib`
from its bib. Add a paper with `pixi run papers add`, never by hand here.
Never copy a PDF into this repo, which is public.

`pixi run papers <command>` runs the CLI on `papers/`:

- `add <doi> --pdf <file>` makes the key and entry from the record and files the
  PDF, refusing one whose first pages show neither that DOI nor that title.
- `check` must pass before a push (CI runs it too): keys and fields agree with
  their records, every entry has its PDF, every quote is still on its page. A
  field the record gets wrong for a reason is excused on the entry,
  `nocheck = {field: reason}`; books and `nopdf = {reason}` need no PDF.
- `search <regex>`, `text <Key> [-p N]`: read a paper there before asking for it.
- `annotate <Key> "<exact quote>" [-p N] [--value 214 --unit ps --what CTR]
  [-t tag] -n "<note>" --by claude`: **every number or claim taken from a paper
  goes in as an annotation**, with the quote that says it. The quote must be
  found on its page or the command fails, so check facts by searching the PDF
  before writing them, and never paraphrase into the quote. Notes say what the
  paper says; a reading of it is flagged as one. `--figure N -p P` boxes a figure.
- `notes [-k Key] [-t tag] [--status unchecked] --json` lists them; `link <Key> <id>`
  gives the viewer URL that opens the PDF at that quote.
- In the notes, a sentence stating a number or finding from a paper ends
  with `\claim{Key/id}`, naming the annotation that quotes it: the map shows a
  page chip that opens the PDF at the quote, and `notes-check` fails on a claim
  with no annotation. Annotate first, then cite it.

The user reads and annotates in the viewer (danielchahine004.github.io/paperstore),
which shows first-page thumbnails from `papers/thumbs/` (`papers check --fix` makes them);
their annotations arrive as commits, so `papers-pull` before relying on them.

For a paywalled paper, use the UTS Library through Claude in Chrome. Open each
paper in its own tab through the EZproxy prefix, e.g.
`https://ieeexplore-ieee-org.ezproxy.lib.uts.edu.au/document/<id>`, or
`https://login.ezproxy.lib.uts.edu.au/login?url=<publisher URL>`. The user,
already signed in, presses PDF and drops the files back here. Chrome blocks a
page from starting several downloads at once, and the PDFs land on the
user's machine, not this server, so pre-opened tabs beat scripted downloads.
Resolve each DOI to its publisher URL first (`curl -sI https://doi.org/<doi>`):
a bare doi.org link leaves the proxy. IOP sometimes shows a CAPTCHA, which is
the user's to tick; J Nucl Med is not on the proxy but is free on its own site.
`paperstore add` checks each returned PDF against its DOI as it files it.
