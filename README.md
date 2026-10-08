<div align="center">

# 🛠️ Agent Skills

**A curated collection of reusable skills for Claude Code and other AI coding agents.**

[![Tested on Claude Code](https://img.shields.io/badge/tested%20on-Claude%20Code-D97757?style=for-the-badge)](https://claude.com/claude-code)
[![Created by Valhalla Technologies Team](https://img.shields.io/badge/created%20by-Valhalla%20Technologies%20Team-1f2937?style=for-the-badge)](#-about)

</div>

---

## ✨ What is this?

This repository contains ready-to-use **skills**: small, self-contained folders of instructions that teach an AI agent how to perform a specific task well and consistently. Drop a skill in, and your agent picks it up automatically when the task matches.

- 📦 **Plug and play**: copy a folder, and it works
- 🔁 **Reusable**: use the same skill across projects and teammates
- 🧩 **Modular**: each skill is independent, so take only what you need
- 🧪 **Tested on Claude Code**

---

## 📁 Repository structure

```
.
├── skills/
│   ├── <skill-name>/
│   │   ├── SKILL.md          # Required: frontmatter + instructions
│   │   ├── scripts/          # Optional: helper scripts
│   │   └── resources/        # Optional: templates, examples, reference docs
│   └── ...
├── LICENSE
└── README.md
```

---

## 🚀 Installation

### Claude Code

**Option 1: Personal skills** (available in all your projects)

```bash
git clone https://github.com/<your-org>/<your-repo>.git
cp -r <your-repo>/skills/<skill-name> ~/.claude/skills/
```

**Option 2: Project skills** (scoped to one repo, can be committed and shared with your team)

```bash
cp -r <your-repo>/skills/<skill-name> <your-project>/.claude/skills/
```

Restart Claude Code (or start a new session), then run `/skills` to confirm the skill is loaded.

### Other agents

The skills use a plain-Markdown `SKILL.md` format, so they are easy to adapt to other agents and tools that support instruction files or custom prompts. Copy the content of a skill's `SKILL.md` into your agent's equivalent (for example a rules file, a custom instructions field, or an `AGENTS.md`).

> ⚠️ Compatibility with agents other than Claude Code is best-effort and has not been formally tested.

---

## 🧰 Available skills

| Skill | Description |
|-------|-------------|
| `skill-name` | Short description of what it does and when it triggers |
| `skill-name` | Short description of what it does and when it triggers |
| `skill-name` | Short description of what it does and when it triggers |

*Add your skills to this table as the collection grows.*

---

## 💡 Usage

Skills are triggered automatically based on their `description`, or you can invoke one explicitly by name in your prompt:

```text
Use the <skill-name> skill to review the changes in this branch.
```

---

## ✍️ Creating your own skill

1. Create a new folder under `skills/`:

   ```bash
   mkdir -p skills/my-skill
   ```

2. Add a `SKILL.md` with frontmatter and instructions:

   ```markdown
   ---
   name: my-skill
   description: One clear sentence explaining what this skill does and when to use it.
   ---

   # My Skill

   ## When to use
   Describe the situations where this skill applies.

   ## Instructions
   1. Step one
   2. Step two
   3. Step three
   ```

3. Test it in Claude Code, then add it to the table above.

**Tips for good skills**

- Write a precise `description`, because it decides when the skill gets triggered
- Keep instructions short, concrete, and actionable
- Put long reference material in separate files and link to it from `SKILL.md`
- Include examples of expected input and output

---

## 🤝 Contributing

Contributions are welcome!

1. Fork the repository
2. Create a branch: `git checkout -b feat/my-new-skill`
3. Add or improve a skill
4. Open a pull request with a short description of what the skill does and how you tested it

---

## 🏢 About

**Created by Valhalla Technologies Team.**
🌐 [valhalla-technologies.com](https://valhalla-technologies.com)

**Tested on Claude Code.**

---

## 📄 License

Distributed under the terms of the license in the [LICENSE](LICENSE) file.
