# Claude Code Skills

A personal collection of custom skills for [Claude Code](https://claude.ai/code) — the AI coding assistant by Anthropic.

Skills extend Claude Code's capabilities with domain-specific workflows, automations, and best-practice guides that can be invoked with a `/` command directly in the chat.

---

## What Are Claude Code Skills?

Skills are instruction files stored under `~/.claude/skills/`. Each skill lives in its own folder with a `SKILL.md` file that tells Claude exactly how to behave when the skill is invoked.

To use a skill, type `/skill-name` in Claude Code's chat.

---

## Skills in This Repo

### `/build-aab` — Build & Upload to Google Play
**Path:** `build-aab/SKILL.md`

Automates the full release build flow for a React Native Android app:

- Reads current `versionCode` from `app.json` and `build.gradle`
- Analyzes recent git commits to suggest a semantic version bump (patch / minor / major)
- Updates version numbers in all relevant files
- Runs pre-build checks: stops Gradle daemons, kills Java processes, cleans stale build folders, adds Windows Defender exclusions, checks network connectivity
- Builds the AAB with `./gradlew bundleRelease --no-daemon`
- Handles common build errors with specific recovery steps
- Displays the output path and upload instructions for Google Play Console

> Responds entirely in Hebrew. Technical values (paths, commands, version numbers) stay in English.

---

### `/tailwindcss` — Tailwind CSS Implementation Guide
**Path:** `tailwindcss/SKILL.md`

A comprehensive reference for implementing Tailwind CSS across any web project:

- Installation & setup for Vite, Next.js, React, Vue, Svelte, Astro
- Core utility classes: layout, spacing, typography, colors, borders, shadows
- Responsive design with mobile-first breakpoints
- Dark mode (media query & class-based)
- Interactive states: hover, focus, active, disabled, group, peer
- Transitions, transforms, and built-in animations
- Arbitrary values and CSS variable integration
- Custom theme tokens via `@theme`
- Component examples: buttons, cards, forms, navigation, grids
- Best practices and common pitfalls
- Troubleshooting guide

---

### `/react-native-best-practices` — React Native Performance Optimization
**Path:** `react-native-best-practices/SKILL.md`

Based on Callstack's *Ultimate Guide to React Native Optimization*. Covers the full performance stack:

| Category | Topics |
|----------|--------|
| **FPS & Re-renders** | FlashList, React Compiler, atomic state, deferred values |
| **Bundle Size** | Barrel imports, tree shaking, Hermes mmap, R8 shrinking |
| **TTI (Startup Time)** | Cold start measurement, native navigation, screen preloading |
| **Native Performance** | Turbo Modules, threading model, C++ modules |
| **Memory Management** | JS & native leak detection, heap snapshots |
| **Animations** | Reanimated worklets, UI thread animations |

Includes 25+ reference files with **Quick Pattern** (before/after code) and **Deep Dive** sections for each optimization.

---

## Installation

Clone this repo directly into your Claude Code skills folder:

**Windows:**
```powershell
cd $env:USERPROFILE\.claude
git clone https://github.com/IsraelBenZeev/claude-code-skills.git skills
```

**macOS / Linux:**
```bash
cd ~/.claude
git clone https://github.com/IsraelBenZeev/claude-code-skills.git skills
```

After cloning, the skills are immediately available in Claude Code — no restart needed.

---

## Updating

To pull the latest skills on any machine:

```bash
cd ~/.claude/skills
git pull
```

---

## Adding New Skills

1. Create a new folder: `~/.claude/skills/my-skill-name/`
2. Add a `SKILL.md` file with the skill instructions
3. Commit and push to keep all machines in sync

---

## License

MIT
