# claude-hyperframes-manager

A [Claude Code](https://claude.com/claude-code) subagent that runs [HyperFrames](https://github.com/heygen-com/hyperframes) video projects for you. You describe what you want in plain words — "make a 15-second promo for our new product", "preview it", "make the title bigger", "render the final version" — and the agent does the HyperFrames work: scaffolding, authoring, linting, checking, previewing and rendering.

> **Unofficial community project.** Not affiliated with, endorsed by, or supported by HeyGen, the HyperFrames team, or Anthropic. HyperFrames is © HeyGen and licensed Apache-2.0.

## Install

### Option 1 — Claude Code plugin (recommended)

Two commands in your terminal. You get updates when this repo changes.

```bash
claude plugin marketplace add TamimMobarak/claude-hyperframes-manager
claude plugin install hyperframes-manager@tamimmobarak-plugins
```

Or inside a Claude Code session:

```text
/plugin marketplace add TamimMobarak/claude-hyperframes-manager
/plugin install hyperframes-manager@tamimmobarak-plugins
```

### Option 2 — download the agent file

The agent is one file. Put it in your user folder to use it in every project:

```bash
mkdir -p ~/.claude/agents
curl -fsSL https://raw.githubusercontent.com/TamimMobarak/claude-hyperframes-manager/main/plugins/hyperframes-manager/agents/hyperframes-manager.md -o ~/.claude/agents/hyperframes-manager.md
```

Windows (PowerShell):

```powershell
New-Item -ItemType Directory -Force "$HOME\.claude\agents" | Out-Null
Invoke-WebRequest https://raw.githubusercontent.com/TamimMobarak/claude-hyperframes-manager/main/plugins/hyperframes-manager/agents/hyperframes-manager.md -OutFile "$HOME\.claude\agents\hyperframes-manager.md"
```

For a single project, save it to that project's `.claude/agents/` folder instead, and commit it to share it with your team.

Claude Code picks up new agent files within a few seconds. Restart only if the `agents` folder did not exist when your session started.

## Requirements

- Node.js 22 or newer
- FFmpeg (includes `ffprobe`) — Windows: `winget install Gyan.FFmpeg` · macOS: `brew install ffmpeg` · Debian/Ubuntu: `sudo apt install ffmpeg`
- The HyperFrames agent skills. The agent reads them before every command rather than relying on memory:

  ```bash
  npx hyperframes skills update
  ```

  This installs the recommended core set (the `hyperframes` router, the `hyperframes-*` domain skills and `media-use`); creation workflows install on demand. Installing only `hyperframes-cli` is enough to run commands but leaves out the router and composition rules, so new-video requests will be weaker.

The agent checks all of this on its first run and tells you what is missing. It never installs these for you without asking.

## Use

### Talk to it directly (recommended)

Start a Claude Code session with the agent as the main assistant:

```bash
# installed as a plugin (Option 1)
claude --agent hyperframes-manager:hyperframes-manager

# installed as a file (Option 2)
claude --agent hyperframes-manager
```

The whole session becomes the HyperFrames manager. It greets you with the status of your projects, then you describe videos in your own words and go back and forth with it — it asks questions, shows you previews, and waits for your go-ahead before rendering.

To make it the default in a folder where you keep videos, add `.claude/settings.json` to that folder with `"agent": "hyperframes-manager:hyperframes-manager"` (plugin) or `"agent": "hyperframes-manager"` (file).

### Or delegate to it from a normal session

Mention HyperFrames or name the agent:

- "Use the hyperframes-manager agent to check my setup."
- "Make a HyperFrames video: a 20-second promo for our spring sale, 9:16 for Instagram."
- "Preview it." · "Make the headline bigger and slow the intro down." · "Undo that." · "Render the final version."

You can also type `@` and pick it from the list.

Delegated agents cannot talk to you while they run — that is how Claude Code subagents work. Your session passes the instruction on, and the agent comes back with a report ending in **Done**, **Result**, **Checks** and, when something is pending, **Needs your decision**. Your answer goes back on its next run. This works, but creative back-and-forth is smoother in direct mode.

Tell it where to keep your videos the first time. It uses a permanent folder and will not create projects in temporary ones.

## What it does

- Routes new videos through the HyperFrames `hyperframes` router skill, which picks the right workflow (product promo, explainer, captions, motion graphic, music video, slideshow and more).
- Edits existing projects, tracking its changes with HyperFrames project history so they can be undone.
- Searches the HyperFrames catalog for ready-made effects before hand-writing animation.
- Runs the quality gates (`lint`, `check`, snapshots) before calling anything done.
- Opens a preview and **waits for your approval before rendering**, then verifies the file with `ffprobe`.
- Keeps a `HYPERFRAMES_MANAGER.md` state file in your video folder, so it picks up where it left off.

## What it will not do without asking

Anything that costs money, uploads your content, publishes something, or changes your system:

- HeyGen cloud rendering (paid credits) and `auth login`
- `publish` (uploads your project; `--public` makes it viewable by anyone with the link)
- AWS Lambda or Google Cloud Run deploys and renders (billed to you)
- Sending `feedback` reports to HyperFrames' public channel — the HyperFrames skill tells agents to send one after every render; this agent offers instead
- Cloud text-to-speech, transcription, music or image generation (ElevenLabs, Gemini, HeyGen, OpenAI)
- The optional ~33 MB on-device catalog search model
- Starting extra headless agent sessions as workers
- Installing or updating FFmpeg, Chrome, npm packages or HyperFrames skills (it asks once whether on-demand workflow skill installs are OK)
- Deleting or overwriting projects, renders or media

On first run it also tells you that the HyperFrames CLI sends anonymous usage telemetry by default, and how to turn it off.

## Tools it has

`Read, Write, Edit, Bash, Glob, Grep, WebFetch, Skill, Agent, AskUserQuestion`. It runs commands and edits files because building and rendering video needs both. `Skill` loads the HyperFrames skills; `Agent` lets HyperFrames workflows hand frames to helper agents (without it, frames are built one at a time); `AskUserQuestion` is used in direct mode.

Always-on cost is about 70 tokens per session; a run loads about 3,200 tokens of instructions. Review [the agent file](plugins/hyperframes-manager/agents/hyperframes-manager.md) before installing, as you should with any agent.

## Repository layout

```text
.claude-plugin/marketplace.json                       marketplace listing (tamimmobarak-plugins)
plugins/hyperframes-manager/.claude-plugin/plugin.json plugin manifest
plugins/hyperframes-manager/agents/hyperframes-manager.md  the agent
```

## License

Apache-2.0 — see [LICENSE](LICENSE). HyperFrames is a separate project by HeyGen under its own Apache-2.0 license.
