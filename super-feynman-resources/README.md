# Super Feynman - Hackathon Resource Package

**Production-tested development patterns adapted for your Super Feynman hackathon project.**

Born from 6 months of real-world enterprise TypeScript development, customized for JavaScript + Express + SQLite + React + Tailwind CSS.

---

## 📦 What's Inside

This resource package contains **3 specialized skills**, **2 slash commands**, and comprehensive guides extracted from production code and specifically adapted for the Super Feynman learning application.

### Structure

\`\`\`
super-feynman-resources/
├── .claude/
│   ├── skills/
│   │   ├── backend-dev/           # Express + JavaScript + SQLite patterns
│   │   │   ├── SKILL.md
│   │   │   └── resources/
│   │   │       ├── complete-backend-guide.md
│   │   │       ├── architecture-patterns.md
│   │   │       ├── database-sqlite.md
│   │   │       ├── error-handling-middleware.md
│   │   │       └── routes-examples.md
│   │   ├── frontend-dev/          # React + Tailwind CSS patterns
│   │   │   ├── SKILL.md
│   │   │   └── resources/
│   │   │       ├── complete-frontend-guide.md
│   │   │       ├── audio-recording.md
│   │   │       ├── component-patterns.md
│   │   │       ├── data-fetching-hooks.md
│   │   │       ├── routing-navigation.md
│   │   │       └── tailwind-styling.md
│   │   ├── api-integration/       # Anthropic Claude & OpenAI Whisper
│   │   │   ├── SKILL.md
│   │   │   └── resources/
│   │   │       ├── complete-api-guide.md
│   │   │       ├── anthropic-integration.md
│   │   │       ├── error-handling-retries.md
│   │   │       └── whisper-integration.md
│   │   └── skill-rules.json       # Auto-activation configuration
│   ├── commands/
│   │   ├── quick-start.md         # /quick-start command
│   │   └── project-structure.md   # /project-structure command
│   ├── agents/                    # (Empty - can add custom agents)
│   └── hooks/                     # (Empty - can add custom hooks)
├── INTEGRATION_README.md          # How to integrate into your project
├── SUMMARY.md                     # Overview and learning path
└── README.md                      # This file
\`\`\`

---

## 🚀 Quick Start

### Option 1: Use as Claude Code Skills (Recommended)

If you're using Claude Code for development:

\`\`\`bash
# In your Super Feynman project repository
cp -r /path/to/super-feynman-resources/.claude .

# Skills will auto-activate based on file patterns and keywords!
\`\`\`

**What you get:**
- Skills automatically suggest themselves when relevant
- Progressive disclosure (SKILL.md loads first, resources/ on demand)
- Consistent patterns across your codebase
- Inline guidance while coding

### Option 2: Reference Documentation

Keep this folder separate and reference during development.

### Option 3: Copy to Docs

Copy guides into your project documentation folder.

---

## 📚 Available Skills

### 1. backend-dev
Express + JavaScript + SQLite patterns

### 2. frontend-dev  
React + Tailwind CSS patterns

### 3. api-integration
Anthropic Claude & OpenAI Whisper integration

See `.claude/skills/` for details.

---

## 💬 Available Commands

- `/quick-start` - Implementation checklist
- `/project-structure` - Folder structure guide

---

## 📖 Documentation

- **INTEGRATION_README.md** - How to integrate
- **SUMMARY.md** - Overview and learning path

---

**Good luck with Super Feynman! 🧠✨**
