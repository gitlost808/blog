# Repository Guide

## Overview

This repository contains the source for `blog.0x29a.me`.

- The site is built with Hugo (extended edition); configuration is in `hugo.toml`.
- Published pages and posts live in `content/`; posts normally live in
  `content/posts/<slug>.md` and use TOML front matter delimited by `+++`.
- Site customizations are in `layouts/`, translations in `i18n/`, and static assets in
  `static/`.
- The PaperMod theme is managed as a Hugo module through `go.mod`/`go.sum`; do not
  vendor or edit theme files unless that is an intentional project-level change.
- `bin/generator/` is a separate Python CLI that uses Ollama to create and edit posts.
  It writes to the site workspace through `WORKSPACE` (defaulting to
  `/app/mnt-workspace` in its container setup).

## Working Conventions

- Keep content changes focused. Preserve existing front matter fields, Markdown style,
  internal links, and post slugs unless the task requires otherwise.
- Use TOML front matter (`+++`), not YAML, for new Hugo content.
- Keep generated Hugo output out of version control: `public/`, `resources/_gen/`,
  `hugo_stats.json`, and `.hugo_build.lock` are ignored.
- Python dependencies are pinned in `bin/generator/requirements.txt`. Do not hand-edit
  the generator's `.venv/`; it is local and ignored.
- Do not run `deploy.sh` unless deployment is explicitly requested: it deletes the
  local `public/` directory and synchronizes to `/srv/http/blog/`.

## Common Commands

Run these from the repository root unless noted otherwise.

```bash
# Render the site locally (writes ignored generated output to public/)
hugo

# Serve the site during content/layout work
hugo server

# Generator syntax check (installs its pinned dependencies into bin/generator/.venv)
make -C bin/generator check

# Run the interactive local generator; requires Hugo and a running local Ollama service
make -C bin/generator run POST=<slug> MODEL=llama3.1

# Run the generator in Docker
./bin/generate.sh <slug> --model llama3.1
```

`make -C bin/generator check` requires access to install the pinned Python packages if
they are not already present. The Docker workflow uses host networking and mounts the
repository into the container.

## Validation Expectations

- For Hugo content, layout, configuration, or asset changes, run `hugo` and resolve
  build errors.
- For changes below `bin/generator/src/`, run `make -C bin/generator check`.
- Avoid using the interactive generator as an automated verification step: it requires
  a reachable Ollama server and can create or overwrite post content.
