# paper-notes

**▶ [Open the map](https://danielchahine004.github.io/danvas/?repo=DanielChahine004/paper-notes)** (runs in your browser, no server)

Working notes on the papers behind a PhD thesis on monolithic-detector PET,
one LaTeX file, and the world map of research programmes drawn from them. The
thesis is in [phd-thesis](https://github.com/DanielChahine004/phd-thesis); the
papers themselves are in the private thesis-papers repo.

## Setup and build

Install [pixi](https://pixi.sh), then in the repo root:

```
pixi install
pixi run build            # -> build/main.pdf
pixi run sync             # commit, pull, refresh the bibliography, push
```

`src/paper-notes/main.tex` is the notes. `src/shared/` holds `units.tex` (a copy
of the thesis's unit shorthands) and `references.bib` (a copy of thesis-papers'
bibliography, refreshed by every `pixi run sync` and `pixi run papers-pull`;
add papers there, not here).

## The map

The paper notes also render as a live board, and as a world map pinned by
institution. The tool is `tools/notesmap`, configured by `notesmap.toml`, and
it follows edits to the notes without a restart:

```
pixi run notes-map        # world map, http://127.0.0.1:8000
pixi run notes-canvas     # the same notes as clustered rows
pixi run notes-check      # lint: missing images, dangling refs, unplaced programmes,
                          # and every \doi{} in the notes against src/shared/references.bib
pixi run notes-check --gaps --strict   # also unfinished notes; non-zero exit, for CI
pixi run notes-edit       # shared live editor, http://127.0.0.1:8891/lab
```

`notes-edit` serves `src/paper-notes` as JupyterLab with real-time
collaboration: several people type in the notes at once, images arrive by
dropping them in the file tree, and every keystroke is written to the file, so
the map follows without a save or a push. The map carries a link to it.
Its password is `NOTES_EDIT_TOKEN`, which falls back to the placeholder
`change-me` in `tools/jupyter-notes-config.py` when unset — change it before sharing
the link, because anyone holding it can edit the notes.

To share just the map, nothing needs to run: it is published from this
repository at
[danielchahine004.github.io/danvas/?repo=DanielChahine004/paper-notes](https://danielchahine004.github.io/danvas/?repo=DanielChahine004/paper-notes),
built in each reader's browser (`danvas.toml` says how) and redrawn within a
minute of a push.

To share both the map and the live editor with a lab, one command does the lot:

```
NOTES_EDIT_TOKEN=<a password> pixi run notes-lab
```

It opens a public HTTPS tunnel for each (danvas bundles the tunnel binary, so
there is nothing to install), starts the editor and the map, points the map's
button at the editor's public address, and prints the two links. The tunnels are
held by detached keeper processes, so **the addresses survive a restart** — stop
them deliberately with `python -m danvas.tunnel --stop --port 8000`. Set
`NOTES_MAP_URL` and `NOTES_EDIT_URL` to skip tunnelling, for a domain of your
own or to run the pair locally.

The map has no password: anyone with the link can read it, and can also scribble
on the canvas and drag panels, which a restart clears. The editor's password is
the only thing protecting the notes, so treat the two links differently.

`tools/notesmap/README.md` explains the note macros, the lab-sharing setup
(one file per programme, lock files, bucket or URL sources), and how another
lab adapts it.

## CI

`.github/workflows/build.yml` runs on every push: `notesmap check --gaps
--strict`, which fails on a defect in the notes (a missing image, a `\ref` to a
key nothing defines, a programme with no `\location`, a `\doi` that reaches no
bibliography entry), and a build whose PDF is uploaded as a run artefact.

## Paper PDFs

```
pixi run papers-pull       # clone the store into papers/, or update it
pixi run papers-push       # share PDFs you added to papers/
pixi run papers-status     # which bibliography keys still lack a PDF
pixi run papers-fetch      # download the open-access copies of those
```

The PDFs live in a private repo,
[thesis-papers](https://github.com/DanielChahine004/thesis-papers), cloned into
the git-ignored `papers/`: one file per key in `references.bib`
(`papers/Freire2026.pdf`). It is private, and kept out of this public repo and
Overleaf, because most of the papers are under publisher copyright. A
collaborator needs an invite (Settings → Collaborators on that repo), then can
pull and push here or upload through its web page, named by the key.
