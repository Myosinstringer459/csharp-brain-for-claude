<h1 align="center">C# Brain for Claude</h1>

<p align="center">
  <b>Curated knowledge base to supercharge Claude as your C# / .NET copilot</b><br/>
  Drop into any project as context — get senior-level code from day one
</p>

<p align="center">
  <img src="https://img.shields.io/badge/C%23-14-512BD4?style=flat-square&logo=csharp" alt="C# 14"/>
  <img src="https://img.shields.io/badge/.NET-10-512BD4?style=flat-square&logo=dotnet" alt=".NET 10"/>
  <img src="https://img.shields.io/badge/EF_Core-10-003B57?style=flat-square" alt="EF Core 10"/>
  <img src="https://img.shields.io/badge/WPF-MVVM-7C5CFC?style=flat-square" alt="WPF"/>
  <img src="https://img.shields.io/badge/License-MIT-green?style=flat-square" alt="MIT"/>
</p>

---

## What is this?

A structured collection of **C# patterns, rules, snippets, and architecture decisions** designed to be fed into [Claude](https://claude.ai) (or any LLM) as context. Instead of repeating yourself every conversation, give Claude a brain.

**The problem:** Claude knows C# well, but defaults to generic patterns. It doesn't know *your* style — `Result<T>` over exceptions, `FrozenDictionary` in hot paths, `field` keyword, Vertical Slices, Minimal API.

**The solution:** This repo. Copy the `CLAUDE.md` into your project root, or reference individual files as context.

## Contents

| File | What's inside |
|------|--------------|
| [`CLAUDE.md`](CLAUDE.md) | Drop-in instructions for Claude — code style, architecture, naming, forbidden patterns |
| [`senior-tips.md`](senior-tips.md) | Performance patterns, async best practices, DI, source generators, security |
| [`architecture-decisions.md`](architecture-decisions.md) | ADRs: Modular Monolith, Result pattern, EF Core + Dapper, Vertical Slices |
| **snippets/** | |
| [`csharp14-features.md`](snippets/csharp14-features.md) | Extension members, `field` keyword, null-conditional assignment, implicit spans |
| [`result-pattern.md`](snippets/result-pattern.md) | `Result<T>` + `Error` implementation, HTTP mapping, railway chaining |
| [`ef-core.md`](snippets/ef-core.md) | AsNoTracking, pagination (offset + keyset), split queries, compiled queries, Dapper |
| [`handlers.md`](snippets/handlers.md) | Command/Query handlers, validators, pipeline behaviours |
| [`wpf-mvvm.md`](snippets/wpf-mvvm.md) | MVVM Toolkit ViewModels, DI setup, glassmorphism XAML, navigation |
| [`docker.md`](snippets/docker.md) | Multi-stage .NET Dockerfile, docker-compose with PostgreSQL + Redis + Seq + Jaeger |
| [`github-actions.md`](snippets/github-actions.md) | .NET CI/CD, Docker build & push, release workflow |

## Quick start

### Option 1: Copy CLAUDE.md into your project

```bash
# In your .NET project root
curl -o CLAUDE.md https://raw.githubusercontent.com/valinerosgordov/csharp-brain-for-claude/master/CLAUDE.md
```

Claude Code and Claude IDE extensions will automatically pick up `CLAUDE.md` from your project root.

### Option 2: Reference as context

Point Claude to specific files when you need them:

> "Use the patterns from `senior-tips.md` and `result-pattern.md` for this feature"

### Option 3: Fork and customize

```bash
git clone https://github.com/valinerosgordov/csharp-brain-for-claude.git
```

Edit the rules to match your team's conventions.

## Who is this for?

- **.NET developers** who use Claude Code, Cursor, Windsurf, or Claude API
- **Teams** who want consistent AI-generated code across projects
- **Solo devs** tired of re-explaining their architecture to AI every session

## Philosophy

1. **Modern C# only** — C# 14, .NET 10. No legacy patterns.
2. **Opinionated** — `Result<T>` over exceptions, Vertical Slices over layer folders, Minimal API over controllers.
3. **Production-grade** — every snippet compiles and follows real-world best practices.
4. **Copy-paste ready** — no filler, just working code.

## Contributing

Found a pattern that makes Claude write better C#? PRs welcome.

- Add snippets to `snippets/`
- Update rules in `CLAUDE.md`
- Keep everything concise — Claude has a context window

## License

[MIT](LICENSE)
