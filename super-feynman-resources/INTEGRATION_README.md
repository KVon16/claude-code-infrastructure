# Super Feynman - Resource Integration Guide

## Overview

This folder contains customized development resources extracted and adapted from the claude-code-infrastructure showcase specifically for the **Super Feynman** hackathon project.

## What's Inside

### 1. **backend-development-guide.md**
- Express.js with JavaScript patterns (adapted from TypeScript version)
- SQLite database integration (adapted from Prisma patterns)
- Clean architecture: Routes → Controllers → Services → Data Access
- Error handling patterns
- API route organization

### 2. **frontend-development-guide.md**
- React 18+ patterns with Hooks
- Tailwind CSS styling best practices (adapted from MUI v7 version)
- Component organization and structure
- State management patterns
- Loading states and error handling
- Lazy loading and code splitting

### 3. **error-handling-guide.md**
- Error handling patterns for Node.js/Express
- Consistent error responses
- Logging strategies (console-based for hackathon, Sentry-ready for production)
- Async/await error patterns

### 4. **api-integration-guide.md**
- **Anthropic API (Claude)** integration patterns:
  - Concept generation from lecture notes
  - Conversational review sessions
  - Feedback analysis
- **OpenAI Whisper API** integration for speech-to-text
- Rate limiting and error handling
- Environment variable management

### 5. **project-structure-guide.md**
- Recommended folder structure for Super Feynman
- Frontend and backend organization
- File naming conventions
- Module organization

### 6. **quick-start-checklist.md**
- Step-by-step implementation guide
- Priority order for development
- Testing checkpoints

---

## How to Use These Resources

### Phase 1: Project Setup (30 minutes)

1. **Read `project-structure-guide.md` first**
   - Set up your folder structure
   - Initialize npm packages
   - Configure environment variables

2. **Set up the backend** using `backend-development-guide.md`
   - Create the Express server
   - Set up SQLite database with schema
   - Create base controller and service patterns

3. **Set up the frontend** using `frontend-development-guide.md`
   - Initialize React app with Vite
   - Configure Tailwind CSS
   - Set up routing structure

### Phase 2: Core Development (Main hackathon work)

4. **Follow `api-integration-guide.md`** for:
   - Anthropic API integration for concept generation
   - Chat interface with Claude
   - Whisper API for speech-to-text

5. **Use `error-handling-guide.md`** throughout development:
   - Wrap all API calls with proper error handling
   - Implement user-friendly error messages
   - Set up logging

### Phase 3: Polish & Testing

6. **Use `quick-start-checklist.md`**
   - Verify all features work end-to-end
   - Test error scenarios
   - Polish UI/UX

---

## Integration into Your New Repository

### Option 1: Direct Copy (Recommended for Hackathon)

Copy the relevant guides into your new repo:

```bash
# In your new Super Feynman repo
mkdir -p docs/development-guides
cp /path/to/super-feynman-resources/*.md docs/development-guides/
```

Then reference them during development.

### Option 2: Claude Code Skills (For Post-Hackathon)

If you want to integrate these as Claude Code skills for ongoing development:

1. Create `.claude/skills/super-feynman-backend/` directory
2. Copy `backend-development-guide.md` as `SKILL.md`
3. Add YAML frontmatter with triggers
4. Create `.claude/skills/skill-rules.json` for auto-activation

See the original repo's `.claude/skills/README.md` for detailed instructions.

### Option 3: Reference Material

Keep this folder as reference material and consult it when:
- Setting up new routes/controllers
- Creating new React components
- Integrating API calls
- Handling errors
- Structuring your project

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
│   │   ├── controllers/        # Follow backend-development-guide.md
│   │   ├── services/           # Business logic
│   │   ├── routes/             # API routes
│   │   ├── db/                 # SQLite database
│   │   ├── utils/              # Helpers
│   │   └── app.js              # Express setup
│   ├── .env                    # API keys
│   └── package.json
├── frontend/
│   ├── src/
│   │   ├── components/         # Follow frontend-development-guide.md
│   │   ├── pages/              # Screen components
│   │   ├── services/           # API client
│   │   ├── hooks/              # Custom React hooks
│   │   └── App.jsx
│   ├── tailwind.config.js
│   └── package.json
└── docs/
    └── development-guides/     # These resource files
```

---

## Support During Hackathon

### When to Reference Each Guide

- **Creating a new API route?** → `backend-development-guide.md` sections on Routes & Controllers
- **Calling Anthropic API?** → `api-integration-guide.md` sections on Claude integration
- **Creating a React component?** → `frontend-development-guide.md` sections on Component Patterns
- **Styling with Tailwind?** → `frontend-development-guide.md` sections on Styling
- **Handling errors?** → `error-handling-guide.md` for patterns
- **Unsure about structure?** → `project-structure-guide.md`

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
