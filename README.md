# learn

[![video](assets/thumbnail.png)](https://www.youtube.com/watch?v=kzcI5F4tGiU)

My AI learning system from this video: [How I Use AI to Learn Things](https://www.youtube.com/watch?v=kzcI5F4tGiU).

This is a personal system I built for myself, shared as-is. Built as a pi configuration: the teaching philosophy encoded in a skill, a few small extensions, and agent definitions.

## What's in it

- `skills/teach/` — the philosophy and the process
- `skills/visualize/` — adds a correct, minimal diagram to a lesson when an idea is clearer as a picture
- `extensions/ask-user-question/` — the agent asks you questions through a UI popup
- `extensions/quiz/` — graded questions with instant feedback (✓/✗, correct answer, explanation)
- `extensions/md-log/` — link a markdown file to the session
- `extensions/visual-tools/` — tools for visualization subagents
- `agents/` — `researcher`, `svg-maker`, `mermaid-maker`: the subagents the system delegates to

## Install

This repo **is** a `.pi` directory. From your learning project's root:

```bash
git clone https://github.com/NathanDai5287/learn .pi
```

Then open pi in that directory. (Or copy the pieces you want into your existing project config.)

### Windows with an existing checkout

Keep one long-lived learning directory and expose this checkout inside it as
`.pi`. A directory junction lets you update the checkout normally without
duplicating it:

```powershell
New-Item -ItemType Directory C:\Learning
Set-Location C:\Learning
New-Item -ItemType Junction -Path .pi -Target C:\path\to\this\repo
pi
```

Pi requires a bash implementation on Windows. Git for Windows supplies Git
Bash and is sufficient. Accept Pi's project-trust prompt when it first sees the
project-local `.pi` configuration.

## Use it

Start Pi from the learning directory and ask naturally:

```text
Teach me eigenvectors from first principles. I know basic matrix multiplication,
but I do not yet understand linear transformations geometrically.
```

The `teach` skill should activate automatically. You can invoke it explicitly
with `/skill:teach` if desired. The system first probes your current edge with
graded questions, asks what outcome you want, presents a dependency-map plan,
waits for your approval, and then teaches one checked reasoning step at a time.

Name a lesson so it is easy to find later:

```text
/name Eigenvectors introduction
```

Pi automatically saves terminal sessions. From the same learning directory:

```powershell
pi -c    # continue the most recent session
pi -r    # choose an older session
```

`/resume` does the same selection from inside Pi. Sessions are grouped by
working directory, so consistently launching Pi from the learning directory is
important. Separate sessions do not automatically share a durable model of
what you know: resume the same topic session, or explicitly attach/read an old
lesson note when beginning a new one.

## Notes, files, and Obsidian

Obsidian is optional. Pi itself renders Markdown, Mermaid, and LaTeX in the
terminal. Obsidian is useful when you want a comfortable, searchable notebook
and persistent rendered lesson artifacts.

A practical layout is:

```text
C:\Learning\
  .pi\                  # this repository (or a junction to it)
  notes\                # lesson notes you keep
  resources\            # PDFs, exercises, and source material
  viz\                  # generated automatically when visuals are used
```

The Markdown log must already exist before it can be linked:

```powershell
New-Item -ItemType Directory notes
New-Item -ItemType File notes\eigenvectors.md
```

Then, inside Pi:

```text
/md-log notes/eigenvectors.md
```

The command backfills the active conversation and mirrors future teaching,
quiz results, math, and image embeds. Stop mirroring with `/md-unlog`. You can
open `C:\Learning` as an Obsidian vault, but the Markdown files remain ordinary
portable files and can be read in any editor.

To start a fresh session using an old lesson as context, include it explicitly:

```text
@notes/eigenvectors.md Continue teaching me from this lesson. First check which
parts I still retain.
```

## Optional subagents and visuals

The main tutor, quiz UI, and Markdown logging work without subagents. For web
verification and generated diagrams, install the recommended extension:

```powershell
pi install git:github.com/HazAT/pi-interactive-subagents
```

Run Pi inside a supported terminal multiplexer. On Windows, WezTerm is the
simplest supported option: open a WezTerm tab, change to the learning directory,
and run `pi`.

Install the Mermaid renderer's local dependencies once:

```powershell
Set-Location C:\Learning\.pi\extensions\visual-tools
$env:PUPPETEER_SKIP_DOWNLOAD = '1'
npm install
Set-Location C:\Learning
```

The visual tools look for Chrome or Edge in their standard Windows locations.
SVG rendering uses `rsvg-convert` when available and otherwise falls back to
ImageMagick's `magick`. The bundled agents inherit the main Pi session's model,
so select a capable model once with `/model` rather than configuring separate
provider credentials in every agent definition.

## Requirements

- [pi](https://github.com/earendil-works/pi)
- A subagent implementation, so the system can spawn the researcher and the visual makers. Recommended: [pi-interactive-subagents](https://github.com/amosblomqvist/pi-interactive-subagents) (tmux only). With it, everything works out of the box. Any other implementation works too, but expect to adapt the agent definitions, e.g. `agents/researcher.md` lists `safe_bash` in its tools, which is specific to that extension.
- `ask-user-question` — use the copy bundled here. If your setup already has an `ask-user-question` extension, use **this** one in its place. Popups from different extensions serialize through a shared UI lock, which only works when it's the same implementation.

## Notes

You can run the system without subagents. The main session does the teaching. You just lose the researcher (truth verification) and the generated visuals.

The teaching skill is written for one learner (me). Edit the skill to fit how you learn best.
