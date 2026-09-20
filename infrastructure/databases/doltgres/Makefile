.PHONY: setup docs viewer
setup:
	uv sync

docs:
	python3 ../../../docs/serve.py

viewer:
	uv run uvicorn lab_viewer:app --app-dir dolt/tooling --host 127.0.0.1 --port 8092
