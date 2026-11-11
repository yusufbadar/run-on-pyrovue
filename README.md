# Run on Pyrovue

[Pyrovue](https://pyrovue.com) runs a GitHub or GitLab repository in an isolated box in the cloud. You paste a link, it works out how to install and start the project, and you get the running app with a live preview, a terminal, the files and a map of the code. Nothing is installed on your computer.

This repository holds the public pieces you can use from your own project: the run links, the README badge, the `pyrovue.json` format with a JSON Schema and examples, and a GitHub Actions workflow that adds a run link to every pull request.

## Run a repository from a link

Put `pyrovue.com/` in front of a repository address:

| What | Link |
| --- | --- |
| A GitHub repository | `https://pyrovue.com/github.com/OWNER/REPO` |
| The short form | `https://pyrovue.com/gh/OWNER/REPO` |
| A branch | `https://pyrovue.com/github.com/OWNER/REPO/tree/BRANCH` |
| A pull request, forks included | `https://pyrovue.com/github.com/OWNER/REPO/pull/NUMBER` |
| A gitlab.com project | `https://pyrovue.com/gitlab.com/GROUP/PROJECT` |
| A GitLab branch | `https://pyrovue.com/gitlab.com/GROUP/PROJECT/-/tree/BRANCH` |

Opening a link shows the repository and asks the visitor to sign in, then starts a box for it. The free plan includes 20 box-hours a month on a box with 2 vCPU and 4 GB of memory. [Pricing](https://pyrovue.com/pricing) has every limit.

## Add the badge to your README

[![Run on Pyrovue](https://pyrovue.com/badge.svg)](https://pyrovue.com/button)

Replace `OWNER/REPO` with your repository, or let the [button page](https://pyrovue.com/button) write it for you.

Markdown:

```markdown
[![Run on Pyrovue](https://pyrovue.com/badge.svg)](https://pyrovue.com/github.com/OWNER/REPO?ref=badge)
```

HTML:

```html
<a href="https://pyrovue.com/github.com/OWNER/REPO?ref=badge"><img src="https://pyrovue.com/badge.svg" alt="Run on Pyrovue" width="132" height="28"></a>
```

reStructuredText:

```rst
.. image:: https://pyrovue.com/badge.svg
   :target: https://pyrovue.com/github.com/OWNER/REPO?ref=badge
   :alt: Run on Pyrovue
```

## pyrovue.json

Most repositories need no configuration. Pyrovue reads what the repository already says about itself: package files, lockfiles, the README, a Dockerfile, a Procfile, a Compose file and more.

When your project needs something those files don't show, such as a migration before the first start, put a `pyrovue.json` at the root of the repository. Pyrovue uses it instead of guessing. Every field is optional.

```json
{
  "setup": ["pnpm install", "pnpm db:migrate"],
  "start": "pnpm dev",
  "port": 3000,
  "env": { "DATABASE_URL": "file:./dev.db" }
}
```

| Field | Type | What it does |
| --- | --- | --- |
| `setup` | array of strings | Shell commands, run in order from `cwd`. |
| `start` | string | The command that serves the app. Leave it out for a library or a command-line tool. |
| `port` | number | The port the app listens on. |
| `env` | object | Environment variables. Values can be strings, numbers or booleans. |
| `cwd` | string | A folder inside the repository to run from. |
| `previewPath` | string | The path or query the preview opens first, such as `/docs` or `?demo=1`. |
| `stack` | array of strings | Labels shown on the box, such as `["Node", "Vite"]`. |
| `services` | array, up to 5 | Background programs that start before `start`. Each one is `{ "cwd", "start", "port" }`, with `cwd` given from the repository's root. |

### A frontend with a backend

Declare the backend as a service. It starts first, in the background, and the preview shows the frontend.

```json
{
  "setup": ["cd /workspace/backend && pip install -r requirements.txt", "npm install"],
  "services": [{ "cwd": "backend", "start": "uvicorn main:app --port 8000", "port": 8000 }],
  "cwd": "frontend",
  "start": "npm run dev",
  "port": 5173
}
```

The repository is cloned to `/workspace` inside the box.

### Editor completion

[`pyrovue.schema.json`](pyrovue.schema.json) is a JSON Schema for the file. Add a `$schema` line and editors such as VS Code will complete the fields and flag mistakes. Pyrovue ignores fields it doesn't know, so the line is safe to keep.

```json
{
  "$schema": "https://raw.githubusercontent.com/yusufbadar/run-on-pyrovue/main/pyrovue.schema.json",
  "start": "npm run dev",
  "port": 5173
}
```

More in [`examples/`](examples).

## A run link on every pull request

[`workflows/run-on-pyrovue.yml`](workflows/run-on-pyrovue.yml) comments a run link on each new pull request, so a reviewer can open the change running before reading the diff. Copy it to `.github/workflows/run-on-pyrovue.yml` on your default branch.

It only comments. It never checks out or runs the pull request's code, which is what keeps `pull_request_target` safe, so don't add a checkout step to it.

## What runs, and what doesn't

Detected with no configuration:

- **JavaScript and TypeScript** with npm, pnpm, Yarn, Bun or Deno: Next.js, Vite, Remix, SvelteKit, Nuxt, Astro, Express and plain Node scripts, among 20 frameworks.
- **Python**: Django, FastAPI, Flask, Streamlit and Gradio, from `requirements.txt`, `pyproject.toml`, `uv.lock` or a `Pipfile`.
- **Go, Rust, Ruby (Rails included) and PHP (Laravel included)**, plus static HTML sites.
- **Postgres, MySQL, Redis, Memcached and RabbitMQ**, when a Compose file asks for one. They run inside the box and the app is pointed at them.
- **Libraries and command-line tools**, which are installed and left ready in the terminal.

Not today:

- Docker inside a box, and GPUs.
- MongoDB and Elasticsearch as services.
- Java, .NET and Elixir projects without a README, Makefile, Dockerfile or `pyrovue.json` that says how to run them.
- A single commit, a folder of a repository, or a self-hosted GitLab.

## Links

- [pyrovue.com](https://pyrovue.com)
- [Run a GitHub repo online](https://pyrovue.com/run-github-repo-online)
- [The Code Map](https://pyrovue.com/code-map)
- [Security](https://pyrovue.com/security)
- [Guides](https://pyrovue.com/blog)
- [How Pyrovue compares](https://pyrovue.com/alternatives) with GitHub Codespaces, CodeSandbox, Replit, Binder and others
- [llms.txt](https://pyrovue.com/llms.txt), the same links for coding agents

Questions and bug reports are welcome in this repository's issues, or at hello@pyrovue.com.
