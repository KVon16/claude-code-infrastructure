# Super Feynman - Resource Integration Guide

## Overview

This folder contains customized development resources extracted and adapted from the claude-code-infrastructure showcase specifically for the **Super Feynman** hackathon project.

## What's Inside

This package uses the **Claude Code skills format** for progressive disclosure and automatic activation:

### 1. **backend-dev skill**
Located in `.claude/skills/backend-dev/`
- Express.js with JavaScript patterns (adapted from TypeScript version)
- SQLite database integration (adapted from Prisma patterns)
- Clean architecture: Routes → Controllers → Services → Data Access
- Error handling patterns
- API route organization
- Resources: complete-backend-guide.md, architecture-patterns.md, database-sqlite.md, error-handling-middleware.md, routes-examples.md

### 2. **frontend-dev skill**
Located in `.claude/skills/frontend-dev/`
- React 18+ patterns with Hooks
- Tailwind CSS styling best practices (adapted from MUI v7 version)
- Component organization and structure
- State management patterns
- Loading states and error handling
- Lazy loading and code splitting
- Resources: complete-frontend-guide.md, audio-recording.md, component-patterns.md, data-fetching-hooks.md, routing-navigation.md, tailwind-styling.md

### 3. **api-integration skill**
Located in `.claude/skills/api-integration/`
- **Anthropic API (Claude)** integration patterns:
  - Concept generation from lecture notes
  - Conversational review sessions
  - Feedback analysis
- **OpenAI Whisper API** integration for speech-to-text
- Rate limiting and error handling
- Environment variable management
- Resources: complete-api-guide.md, anthropic-integration.md, error-handling-retries.md, whisper-integration.md

### 4. **Slash Commands**
Located in `.claude/commands/`
- **/quick-start** - Step-by-step implementation guide, priority order, testing checkpoints
- **/project-structure** - Recommended folder structure, file naming conventions, module organization

---

## How to Use These Resources

### Phase 1: Project Setup (30 minutes)

1. **Run `/project-structure` slash command first**
   - Set up your folder structure
   - Initialize npm packages
   - Configure environment variables

2. **Set up the backend** using the `backend-dev` skill
   - Reference `.claude/skills/backend-dev/resources/complete-backend-guide.md`
   - Create the Express server
   - Set up SQLite database with schema
   - Create base controller and service patterns

3. **Set up the frontend** using the `frontend-dev` skill
   - Reference `.claude/skills/frontend-dev/resources/complete-frontend-guide.md`
   - Initialize React app with Vite
   - Configure Tailwind CSS
   - Set up routing structure

### Phase 2: Core Development (Main hackathon work)

4. **Use the `api-integration` skill** for:
   - Reference `.claude/skills/api-integration/resources/complete-api-guide.md`
   - Anthropic API integration for concept generation
   - Chat interface with Claude
   - Whisper API for speech-to-text

5. **Error handling patterns** throughout development:
   - See backend-dev and api-integration skill resources
   - Wrap all API calls with proper error handling
   - Implement user-friendly error messages
   - Set up logging

### Phase 3: Polish & Testing

6. **Run `/quick-start` slash command**
   - Verify all features work end-to-end
   - Test error scenarios
   - Polish UI/UX

---

## Integration into Your New Repository

### Option 1: Claude Code Skills (Recommended)

Copy the entire `.claude/` directory structure into your Super Feynman project:

```bash
# In your new Super Feynman repo
cp -r /path/to/super-feynman-resources/.claude .
```

**What you get:**
- Skills auto-activate based on keywords, file patterns, and intent
- Progressive disclosure (SKILL.md loads first, detailed resources on demand)
- Slash commands available: `/quick-start` and `/project-structure`
- Consistent development patterns enforced

### Option 2: Reference Documentation

Keep this folder separate and reference it during development:

```bash
# Keep super-feynman-resources/ alongside your project
/hackathon-workspace/
├── super-feynman-resources/     # This package
└── super-feynman/                # Your project
```

Consult skills and resources when needed for specific patterns.

### Option 3: Extract Resources Only

Copy just the resource markdown files if you want documentation without Claude Code integration:

```bash
# In your new Super Feynman repo
mkdir -p docs/development-guides
find /path/to/super-feynman-resources/.claude/skills -name "*.md" -exec cp {} docs/development-guides/ \;
```

Then reference them manually during development.

---

## Key Differences from Original Infrastructure

### What Was Adapted

| Original | Super Feynman Version |
|----------|----------------------|
| TypeScript | JavaScript |
| Prisma ORM | SQLite directly or better-sqlite3 |
| MUI v7 styling | Tailwind CSS |
| Microservices architecture | Monolithic Express server |
| Production Sentry tracking | Console logging (Sentry-ready) |
| Complex dependency injection | Simple module exports |

### What Was Kept

- Layered architecture (Routes → Controllers → Services → Data)
- Clean separation of concerns
- Error handling patterns
- Component organization principles
- Best practices for async/await
- API service layer patterns

---

## Recommended Development Order

Based on your hackathon timeline, follow this sequence:

1. ✅ **Backend Setup** (2-3 hours)
   - Database schema
   - Basic Express routes
   - File upload handling
   - Anthropic API test integration

2. ✅ **Frontend Basics** (2-3 hours)
   - React routing setup
   - Home screen
   - Course/Lecture/Concept list screens
   - Basic styling with Tailwind

3. ✅ **Core Features** (4-6 hours)
   - Concept generation from notes (Anthropic API)
   - Chat interface
   - Conversation flow with Claude
   - Feedback generation

4. ✅ **Enhancement** (2-4 hours)
   - Speech-to-text (Whisper API)
   - Polish UI/UX
   - Loading states
   - Error handling

5. ✅ **Testing & Deployment** (2-3 hours)
   - End-to-end testing
   - Bug fixes
   - Deployment prep

---

## Quick Reference: File Locations in Your New Project

```
super-feynman/
├── backend/
│   ├── src/
│   │   ├── controllers/        # Use backend-dev skill
│   │   ├── services/           # Business logic
│   │   ├── routes/             # API routes
│   │   ├── db/                 # SQLite database
│   │   ├── utils/              # Helpers
│   │   └── app.js              # Express setup
│   ├── .env                    # API keys
│   └── package.json
├── frontend/
│   ├── src/
│   │   ├── components/         # Use frontend-dev skill
│   │   ├── pages/              # Screen components
│   │   ├── services/           # API client
│   │   ├── hooks/              # Custom React hooks
│   │   └── App.jsx
│   ├── tailwind.config.js
│   └── package.json
└── .claude/                    # Claude Code skills (optional, copy from super-feynman-resources)
    ├── skills/
    │   ├── backend-dev/
    │   ├── frontend-dev/
    │   └── api-integration/
    └── commands/
```

---

## Support During Hackathon

### When to Use Each Skill

- **Creating a new API route?** → `backend-dev` skill activates automatically
  - See `.claude/skills/backend-dev/resources/routes-examples.md`
- **Calling Anthropic API?** → `api-integration` skill activates automatically
  - See `.claude/skills/api-integration/resources/anthropic-integration.md`
- **Creating a React component?** → `frontend-dev` skill activates automatically
  - See `.claude/skills/frontend-dev/resources/component-patterns.md`
- **Styling with Tailwind?** → `frontend-dev` skill
  - See `.claude/skills/frontend-dev/resources/tailwind-styling.md`
- **Handling errors?** → Both `backend-dev` and `api-integration` skills have error handling resources
- **Unsure about structure?** → Run `/project-structure` slash command

---

## Post-Hackathon: Evolving to Production

If you want to take Super Feynman to production after the hackathon:

1. **Migrate to TypeScript** (use original backend-dev-guidelines as reference)
2. **Add Sentry error tracking** (use error-tracking skill from original repo)
3. **Set up proper testing** (use testing-guide.md from original repo)
4. **Add authentication** (implement proper auth service)
5. **Database migration** (consider PostgreSQL instead of SQLite)
6. **Implement Claude Code skills** for ongoing development efficiency

The patterns you're using now are production-ready; you're just using simpler tools for rapid hackathon development.

---

## Questions?

If you have questions about any pattern or best practice:

1. Check the relevant guide in this folder
2. Search the original claude-code-infrastructure repo
3. Ask Claude Code for clarification while referencing these files

Good luck with your hackathon! 🚀
