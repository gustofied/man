# MAN

A family of specialist agent training environments across legal, finance,
markets, negotiation, and business operations. Aim for one environment or
evaluation per week. Mailman is the first; subsequent tasks remain open.

## Structure

- `docs/`: terminal2F documentation structure, theme, and RL notes.
- `environments/mailman/`: consolidated Mailman code, datasets, results, and write-ups.
- `environments/ledgerman/`: provisional finance environment stub.
- `infrastructure/databases/doltgres/`: full imported database experiments,
  saved runs, dependency lockfile, tooling, and research proposal.
- `tools/md_to_image.py`: inherited documentation renderer.

New environment logic should be Go. Python adapters may connect to training
frameworks. Imported Python experiments remain reference implementations.

## Documentation

Run `python3 docs/serve.py`, then open http://127.0.0.1:8093.
The inherited pages describe prior work and may include stale versions or
original repository paths. See `docs/imports.json` for import provenance.

Mailman retains its existing Python environment; Ledgerman remains a stub.
