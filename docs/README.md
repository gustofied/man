# MAN documentation

Inherited from terminal2F: project folders, editorial HTML, and renderer. Imported pages are historical references, not newly verified documentation.

Run `python3 docs/serve.py` from the repository root, then visit http://127.0.0.1:8093. Use `PORT=8094 python3 docs/serve.py` to choose another port.

Render markdown with `python3 tools/md_to_image.py <project> <file.md>`. Optional SVG/PNG rendering requires Rich/CairoSVG; HTML uses the inherited renderer.

## How It Works

1. Write markdown files and drop them in `docs/<project>/`
2. Run the renderer to produce styled HTML
3. Open the HTML in a browser, copy-paste into Google Docs for sharing

The renderer uses Rich to convert markdown into styled terminal output, then exports that as HTML with a clean white background and Solarized color accents. Each project can override colors, fonts, and layout via `style.toml`.

## Usage

```bash
# Render a single file
uv run --with rich tools/md_to_image.py rl grpo_example.md

# Render a specific section
uv run --with rich tools/md_to_image.py rl myfile.md --section "## Training"

# Render all .md files in a project
uv run --with rich tools/md_to_image.py rl --all

# Rebuild all index pages
uv run --with rich tools/md_to_image.py --index

# SVG output
uv run --with rich tools/md_to_image.py rl myfile.md --format svg

# PNG output (needs cairosvg)
uv run --with rich,cairosvg tools/md_to_image.py rl myfile.md --format png

# Custom width
uv run --with rich tools/md_to_image.py rl myfile.md --width 120
```

## Adding a New Project

```bash
mkdir assets/docs/myproject
# drop .md files in there
uv run --with rich tools/md_to_image.py myproject --all
```

Optionally add a `style.toml` for custom colors - see STYLE.md.
