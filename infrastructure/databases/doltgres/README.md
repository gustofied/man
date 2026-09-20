# Doltgres experiments

Shared database infrastructure imported from `doltgres-rl`, originally
`terminal2F/nuggets/dbenv`. This is independent of environment names.

- `dolt/base/`: setup, access control, branch permissions, and commit experiments.
- `dolt/tooling/`: lab, orchestration, server helpers, and saved-run viewer.
- `dolt/tooling/results/`: all ten saved runs.
- `dolt/dashboard.py`, `dashboard.html`, `rollout_viewer.py`: inspection tools.
- `PROPOSAL.md`: preserved proposal, not a selected environment specification.
- `provenance/`: original extraction metadata and README.
- Repository `docs/rl/`: inherited database and RL documentation.

The Python experiments are preserved; they have not been rewritten in Go or
validated as production environment infrastructure. A passing row-count check
is not a complete task verifier.

From this directory, `uv sync` installs the original dependencies and
`make viewer` serves saved runs on port 8092. Live metrics require a database.
Run `python3 docs/serve.py` from the repository root for documentation on port 8093.

Historical startup helpers can delete configured data directories and terminate
Doltgres processes by name. They are not invoked by the docs or saved-run viewer
commands. Inspect and replace their lifecycle before using them for new runs.

The earliest saved run uses a legacy list format; the original viewer expects
the newer format used by the other nine. Source repositories remain unchanged.
