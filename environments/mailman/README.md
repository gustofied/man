# Mailman

Email recipient routing: assign To, CC, and BCC across evolving threads.
Existing work consolidated from terminal2F, with the Python environment and
experimental record preserved. This is not a Go rewrite or a fresh training run.

## Contents

- `email_to_cc_bcc.py`, `pyproject.toml`: original Verifiers environment package.
- `datasets/`: v1, v2, and held-out test Parquet files, generation/check scripts,
  and generation artifacts. Virtual environments and caches are excluded.
- `configs/phase-a.toml`: original training configuration; inspect its remote
  model publishing settings before running it.
- `results/`: training summary and saved evaluations. Identical imported files
  share one copy; `imports.json` maps every source to its destination.
- [Research diary](research.html): detailed experiments and result analysis.
- [Early notes](research-early.md): older Markdown, not the source of the newer
  HTML diary. Rendering it over the diary would lose later work.
- [Article](article.html): local copy of the public Mailman write-up. Its site
  assets and navigation resolve against adamsioud.com and require internet.
- [Environment reference](environment-reference.md): original usage notes.

## Browse

With the MAN docs server running, open
http://127.0.0.1:8093/environments/mailman/research.html.
The docs directory links to this folder; it does not contain duplicate Mailman
content. The public article is https://www.adamsioud.com/projects/mailman.html.

Historical commands inside the write-ups retain their original paths.
The environment still uses its configured Hugging Face dataset by default;
copying the local datasets does not change runtime loading behavior.
