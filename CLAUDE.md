# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

PPT Processor is a CLI tool that converts PPTX files into Markdown speeches with slide thumbnails. It auto-manages dependencies (LibreOffice, Poppler) and optionally uses an OpenAI-compatible API to generate coherent speech content.

## Running the Tool

```bash
# Basic usage
python scripts/extractor_ppt.py <input.pptx>

# With explicit output path
python scripts/extractor_ppt.py <input.pptx> --output <output.md>

# Test with included sample
python scripts/extractor_ppt.py test/OpenClaw与SKILLS技能系统.pptx
```

## Dependencies

```bash
pip install python-pptx>=0.6.21 pdf2image>=1.16.0 Pillow>=9.0.0 requests
pip install openai  # optional, for AI speech generation
```

Binaries managed automatically at runtime:
- **LibreOffice 25.8.5** — downloaded from Aliyun mirrors to `~/.cache/ppt_skill/libreoffice/`
- **Poppler v24.08.0** — downloaded from GitHub (Windows) or installed via package manager (Linux) to `scripts/bin/`

## Environment Variables

| Variable | Purpose | Default |
|---|---|---|
| `OPENAI_API_KEY` | Required for AI generation | — |
| `OPENAI_MODEL` | Model selection | `gpt-4o-mini` |
| `OPENAI_BASE_URL` | Custom API endpoint | OpenAI default |

Without `OPENAI_API_KEY`, the tool falls back to `generate_simple_markdown()` — direct concatenation of extracted content with no AI processing.

## Architecture

Single-file pipeline in `scripts/extractor_ppt.py`:

```
PPTX input
  → setup_poppler() / setup_libreoffice()   # auto-install binaries
  → extract_ppt_content()                   # parse titles, body, notes via python-pptx
  → ppt_to_pdf()                            # LibreOffice headless CLI
  → generate_thumbnails()                   # pdf2image → PNG at 800px width
  → build_prompt()                          # assemble content + image refs
  → generate_speech_with_ai()              # OpenAI API call → Markdown
Output: <name>.md + <name>_thumbnails/ folder
```

Key constants:
- `SCRIPT_DIR` / `BIN_DIR` — relative to script location (`scripts/bin/`)
- `CACHE_DIR` — `~/.cache/ppt_skill`
- `IMAGE_WIDTH` — 800px

## No Formal Test Suite

Testing is manual via the sample file in `test/`. There are no pytest/unittest files. The expected output is `test/OpenClaw与SKILLS技能系统.md` and `test/OpenClaw与SKILLS技能系统_thumbnails/` (13 PNGs).
