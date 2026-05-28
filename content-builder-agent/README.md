# Content Builder Agent

`content-builder-agent` is a filesystem-configured writing agent for long-form and social content. It combines prompt files, reusable skills, a research subagent, and a few built-in tools to produce blog posts, LinkedIn content, research notes, and companion images from a single natural-language prompt.

This project is set up as a developer-facing example rather than a polished package. The code is intentionally simple: most behavior is driven by files on disk, while `content_writer.py` wires the pieces together at runtime.

## Architecture Overview

The agent is composed of a few clear runtime pieces:

- `content_writer.py` is the main entrypoint and orchestration layer. It loads environment variables, registers tools, loads subagents, builds the deep agent, and streams terminal output.
- `AGENTS.md` defines the base writing voice, brand standards, formatting rules, and research expectations.
- `skills/blog-post/` and `skills/social-media/` define specialized workflows for blog and social content. These skill files tell the agent how to structure outputs and when to trigger research or image generation.
- `subagents.yaml` defines the `researcher` subagent, including its system prompt, model, and tool access.
- Built-in tools provide web research, file writing, and image generation:
  - `web_search()` uses Tavily for current information.
  - `write_file()` saves generated text into the project directory.
  - `generate_cover()` creates blog hero images.
  - `generate_social_image()` creates images for LinkedIn or tweet-style content.

At runtime, the main agent uses `AGENTS.md` as memory, loads all skills from `skills/`, and delegates research tasks to the `researcher` subagent defined in YAML.

## Repository Structure

The current repository structure is:

```text
content-builder-agent/
├── AGENTS.md
├── content_writer.py
├── pyproject.toml
├── subagents.yaml
├── uv.lock
├── blogs/
│   └── ai-agents-transforming-software-development.md
├── skills/
│   ├── blog-post/
│   │   └── SKILL.md
│   └── social-media/
│       └── SKILL.md
├── GenAI_UseCases/
│   └── README.md
├── content-builder-agent/
│   └── linkedin/
│       └── MCP.md
└── Users/
    └── sky/
        └── Documents/
            └── Deep_agents/
                └── content-builder-agent/
                    └── linkedin_post.txt
```

Notes on the current layout:

- `blogs/` already contains an example generated post.
- `skills/` contains the workflow instructions that shape agent behavior.
- `GenAI_UseCases/` appears to be reference material rather than active runtime configuration.
- There are some path inconsistencies in the current repo, including nested content/output paths such as `content-builder-agent/linkedin/...` and the accidental `Users/...` tree inside the repo.

## How The Agent Works

The content flow is designed around research-first generation:

1. You give the agent a content prompt from the command line.
2. The main agent interprets the request and loads the relevant skill.
3. For blog or social tasks, the skill instructs the agent to delegate research to the `researcher` subagent first.
4. The researcher uses `web_search()` and `write_file()` to gather current information and save findings.
5. The main agent reads the research, writes the final content, and generates an image when the selected skill requires one.
6. Output files are written into project-relative paths such as `blogs/`, `linkedin/`, `tweets/`, or `research/`.

The terminal UI in `content_writer.py` streams progress so you can see when the agent is researching, writing, or generating images.

## Setup

### Requirements

- Python 3.11 or newer
- `uv`
- External API access for Tavily and Google GenAI features

### Install Dependencies

From the project root:

```bash
uv sync
```

### Create a `.env` File

`content_writer.py` loads environment variables from a `.env` file in the project root using `python-dotenv`.

Example:

```env
TAVILY_API_KEY=your_tavily_api_key
GOOGLE_API_KEY=your_google_genai_key
```

`TAVILY_API_KEY` is read directly by the code. Google image generation also requires valid Google GenAI credentials or API key configuration in your environment.

## Environment Variables

- `TAVILY_API_KEY`
  - Required for the `web_search()` tool.
  - If it is missing, research calls will return `"TAVILY_API_KEY not set"`.
- Google GenAI credentials
  - Required for `generate_cover()` and `generate_social_image()`.
  - The code initializes `google.genai.Client()` directly and does not enforce a single credential variable name in code, so use the credential setup expected by your local Google GenAI SDK configuration.

## Running The Agent

Run the agent from the project root:

```bash
uv run python content_writer.py "Write a blog post about AI agents"
```

Another example:

```bash
uv run python content_writer.py "Create a LinkedIn post about prompt engineering"
```

If no prompt is passed, `content_writer.py` falls back to a hardcoded default task.

## Output Conventions

The skill files define the intended output layout:

- Research findings:
  - `research/<slug>.md`
- Blog posts:
  - `blogs/<slug>/post.md`
  - `blogs/<slug>/hero.png`
- LinkedIn posts:
  - `linkedin/<slug>/post.md`
  - `linkedin/<slug>/image.png`
- Tweet/X threads:
  - `tweets/<slug>/thread.md`
  - `tweets/<slug>/image.png`

Current note: the repository does not fully follow those conventions yet. There are existing flat files and nested paths in the repo, so treat the skill-defined layout as the intended convention and the checked-in files as a mix of examples and in-progress structure.

## Example Prompts

### Blog Content

```bash
uv run python content_writer.py "Write a blog post about how AI agents are transforming software development"
```

```bash
uv run python content_writer.py "Create a technical article about the tradeoffs between agent orchestration and workflow automation"
```

### LinkedIn / Social Content

```bash
uv run python content_writer.py "Create a LinkedIn post about prompt engineering for developer teams"
```

```bash
uv run python content_writer.py "Write a short social post about MCP in agentic AI"
```

## Current Limitations

- The app currently lives in a single Python file, so orchestration, tools, config loading, and terminal rendering are all coupled together.
- Path layout is not fully normalized yet, and some checked-in files do not match the intended skill output structure.
- Research and image generation depend on external APIs and working credentials.
- The repository does not yet include formal automated tests or broader project documentation beyond the in-repo prompt/config files.
