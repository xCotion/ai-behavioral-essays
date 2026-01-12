# Claude Code: Operational Manual

**Version:** January 2026
**Last Updated:** January 11, 2026
**Author's Note:** This is a working reference—I wrote it for my own AI to consume when building systems, and for anyone else who wants to use Claude Code effectively. This document reflects how I actually use the tool running Ashton Dot Services, not theoretical best practices.

---

## Introduction: What This Document Is

This is not documentation. The official docs cover installation and basic usage. This document captures what I've learned through actual use: building $800 websites for local businesses, maintaining client sites, writing Pine Script for algorithmic trading, and automating repetitive tasks that used to eat my days.

**How to use this document:**

For AI consumption: This document is structured for extraction. When Claude is building systems, relevant sections can be included as context. YAML blocks and code examples are formatted for direct parsing. Each section stands alone—you can include Part 4 (Context Management) without needing Part 3.

For human learning: Read Part 1 (Mental Models) first. The rest is reference material—scan headers, dive into what's relevant. Come back when you hit a problem.

**What's NOT covered:** Basic installation, getting started tutorials, subscription comparisons. For that, see the [official documentation](https://docs.anthropic.com/en/docs/claude-code/). This document assumes you have Claude Code installed and have run at least a few sessions.

---

## Part 1: Core Mental Models

### Claude Code as "Agentic Faucet"

In [[Intelligence as a Commodity]], I introduced the water/faucet analogy: intelligence is the water (commoditizing rapidly), and your workflow is the faucet (the differentiator). Claude Code is the most direct faucet for turning AI capability into working code.

The mental shift: you're not "using a coding assistant." You're directing an agent that can read your codebase, execute commands, manage files, and iterate on problems. The constraint isn't intelligence—it's context and direction.

This has practical implications. When something goes wrong, the question isn't "why is Claude dumb?" but "what context is Claude missing?" or "what direction was unclear?" The model is capable. Your job is to provide the **high-fidelity context** (see [[Maximizing AI Utility as a Business Owner]]) that enables that capability.

I've found this reframe changes everything. When a client website build goes sideways, I don't blame the tool—I look at what context I failed to provide. Usually it's something obvious in hindsight: I forgot to mention we're using a specific Tailwind version, or that the client's hosting has PHP limitations.

### The Conversation + Tools + Filesystem Triad

Claude Code operates through three interconnected systems:

**Conversation**: Your prompts and Claude's responses. This is the steering mechanism. Unlike chat interfaces, your conversation here has persistent effects—Claude remembers context within a session and can reference files it has read or created. A well-crafted prompt early in a session pays dividends throughout.

**Tools**: The actions Claude can take. Core tools include Read, Write, Edit, Bash, Grep, Glob, and Task (for subagents). MCP extends this with external integrations—I use Supabase MCP for database operations on my portfolio site and client projects. Understanding which tools exist shapes what you can ask Claude to accomplish.

**Filesystem**: The actual state of your project. Claude operates on real files. Changes persist. This isn't a sandbox—errors have consequences. When Claude reads a file, it sees the current state. When it writes, that becomes the new reality.

These three form a feedback loop: conversation directs tools, tools modify filesystem, filesystem contents inform future conversation.

### What Claude Code Can and Cannot Do

**Can do:**

- Read and understand codebases of significant complexity (I've used it on repos with 50k+ lines without issues)
- Write, edit, and refactor code across multiple files in a single operation
- Execute bash commands, interpret output, and respond appropriately
- Run tests, interpret results, fix failures, and verify the fix worked
- Manage git operations (commits, branches, PRs) with proper messages
- Search files using grep and glob patterns, narrow down to relevant code
- Work with MCP servers for external integrations (Supabase, GitHub, etc.)
- Create and execute plans for complex multi-step tasks
- Spawn subagents for parallelizable work (see [[Multi-Agent Network Orchestration]])

**Cannot do:**

- Maintain state between sessions (context resets when you start fresh)
- Access files outside the project directory without explicit `--add-dir`
- Make network requests without MCP servers or bash tools (curl, wget)
- Run indefinitely—sessions have practical limits (context window fills up)
- Know what changed since you last spoke unless you tell it or it reads files
- See your screen or access your clipboard (it can read image files you point it to)
- Undo changes once accepted (though git handles this)

### When to Use Claude Code vs Alternatives

| Need | Best Tool | Why |
|------|-----------|-----|
| Code changes to a project | Claude Code | Direct filesystem access, session persistence |
| Quick questions, explanations | Chat (claude.ai) | Faster, no setup, no token accumulation |
| Bulk processing, automation | API with SDK | Programmatic control, parallelization |
| Rapid prototyping with preview | Cursor or Windsurf | IDE integration, visual feedback |
| Document creation, analysis | Chat with file upload | Better handling of non-code files |
| Complex multi-step coding | Claude Code with planning | Plan + execute + verify in one flow |
| Research before coding | Chat, then bring context to Claude Code | Cheaper to explore in chat |

The decision tree: If you need changes to files in a project and want full control, use Claude Code. If you want faster visual feedback and don't mind less control, Cursor or Windsurf work well. If you're gathering information or thinking through approaches, chat is faster and cheaper.

For my client website work ($800 builds, $120/month maintenance), I use Claude Code for the actual development and Cursor when I want to see live previews quickly. The tools complement each other.

---

## Part 2: Command Reference with Usage Notes

### Core Slash Commands

| Command | Purpose | When to Use |
|---------|---------|-------------|
| `/help` | List all commands | When you forget what's available |
| `/init` | Generate CLAUDE.md | First thing in a new project |
| `/clear` | Reset conversation context | Switching tasks, clearing confusion |
| `/compact` | Compress context manually | Before hitting token limits |
| `/model` | Switch models mid-session | Sonnet to Opus based on task complexity |
| `/permissions` | Manage tool permissions | Security tuning, troubleshooting |
| `/mcp` | View MCP server status | When integrations aren't working |
| `/doctor` | Diagnose configuration issues | First stop for any problems |
| `/context` | View context window as colored grid | Every 30-60 mins on long sessions |
| `/bashes` | List and manage background tasks | Managing parallel operations |
| `/usage` | Check plan limits (subscription only) | Subscription users monitoring limits |
| `/memory` | Edit CLAUDE.md files | Adding project context mid-session |

### CLI Flags Worth Knowing

```bash
# Start with specific model
claude --model claude-sonnet-4-5-20250929

# Continue most recent session (critical for multi-day work)
claude -c

# Resume specific session by ID or name
claude --resume <session-id>
claude -r auth-refactor "continue where we left off"

# Non-interactive mode (for scripting, CI/CD)
claude -p "your query here"

# Add additional directories to context
claude --add-dir ../shared-lib ../common-types

# Output as JSON (for automation pipelines)
claude -p "query" --output-format json

# Restrict tools (for security-sensitive operations)
claude --allowedTools "Bash(git log:*)" "Bash(git diff:*)" "Read"
claude --disallowedTools "Bash(rm:*)" "Edit"
```

### Keyboard Shortcuts and Tricks

**Escape key behavior:**
- Single Escape: Interrupt Claude mid-thought (preserves context, lets you redirect)
- Double Escape: Go back in history, edit a previous prompt, try a different approach
- This is your "undo" mechanism—use it liberally

**Auto-accept mode**: Use `/permissions` to configure which tools auto-execute. Use sparingly—active collaboration usually produces better results. Good for: applying a pattern across many files. Bad for: complex logic changes.

**Extended thinking**: Toggle with `Option+T` (macOS) or `Alt+T` (Windows/Linux). You can also include `ultrathink` in your message for deeper reasoning on complex problems. Claude 4.5 Sonnet and Opus have thinking enabled by default. Press `Ctrl+O` to see the thinking process in verbose mode.

**@ mentions**: Use `@filename` to reference specific files without Claude searching. More efficient than "look at the auth module"—just say `@src/auth/login.ts`.

**Background tasks**: Press `Ctrl+B` to background a running command. Use `/bashes` to see all background tasks and their status. If you use Tmux, press `Ctrl+B` twice.

### Commands to Avoid (and Why)

**`/clear` when you need continuity**: This wipes context completely. If you're mid-feature, you'll lose everything Claude learned about your approach, your preferences from this session, and the mental model it built of the problem.

**`/compact` too aggressively**: Compaction summarizes and discards detail. Let it happen automatically unless you're actively hitting limits. Premature compaction loses nuance.

**Rapid `/model` switching**: Each switch has overhead. Pick the right model for the task upfront rather than switching mid-thought. I default to Sonnet for most work and switch to Opus when I hit something that requires deeper reasoning.

---

## Part 3: Skills and Customization

### What Are Skills?

Skills are reusable instruction sets that extend Claude Code's capabilities. They appear in the slash command menu and can be invoked with parameters. Think of them as saved workflows for tasks you do repeatedly.

### Creating Custom Skills

Skills live in directories with a `SKILL.md` file inside:
- `~/.claude/skills/skill-name/SKILL.md` — Personal skills (all projects)
- `.claude/skills/skill-name/SKILL.md` — Project skills (shared with team)

Example skill structure:

```markdown
---
name: new-component
description: Create a new React component with tests
arguments:
  - name: component-name
    description: Name of the component (PascalCase)
    required: true
---

Create a new React component named {{component-name}} following our project patterns:

1. Create `src/components/{{component-name}}/{{component-name}}.tsx`
2. Create `src/components/{{component-name}}/{{component-name}}.test.tsx`
3. Export from `src/components/{{component-name}}/index.ts`
4. Follow the pattern in @src/components/Button/Button.tsx
```

### Skills I Use

For my client work, I've built skills for:

- `/new-page` — Creates a new Next.js page with layout, SEO meta, and placeholder content
- `/client-deploy` — Runs build, tests, and deploys to client's hosting
- `/maintenance-check` — Reviews all client sites for updates needed

---

## Part 4: Context Management

### What Goes in CLAUDE.md

CLAUDE.md is the highest-leverage configuration point. It's loaded into every session automatically. Think of it as the constitution for your project's AI interactions.

**Essential sections:**

```markdown
# Project Overview
One-paragraph description of what this project does and its core architecture.

# Tech Stack
- Framework: Next.js 14 (App Router)
- Language: TypeScript 5.x (strict mode enabled)
- Styling: Tailwind CSS 3.x
- Database: PostgreSQL via Supabase
- Authentication: Supabase Auth
- Testing: Vitest + React Testing Library
- Deployment: Vercel

# Key Commands
- `pnpm dev` - Start development server
- `pnpm build` - Production build
- `pnpm test` - Run unit tests
- `pnpm lint` - ESLint check
- `pnpm typecheck` - TypeScript check

# Code Patterns
- Functional components with hooks (no class components)
- Named exports over default exports (except for pages)
- Colocate tests: `Component.tsx` > `Component.test.tsx`
- Server components by default, 'use client' only when needed

# Critical Rules (NEVER VIOLATE)
- NEVER modify files in supabase/migrations/ directly
- NEVER commit code with TypeScript errors
- NEVER push to main directly (always use PRs)
- NEVER store secrets in code (use environment variables)

# Current Focus
Working on: User preferences feature
Key files: src/features/preferences/*
```

**What NOT to put in CLAUDE.md:**

- Code style rules already enforced by linters (Prettier handles formatting, don't repeat it)
- Information Claude can infer from package.json, tsconfig.json, etc.
- Instructions that apply to only some tasks (use prompt context instead)
- Obvious things ("use TypeScript" when the project is clearly TS)

### File Hierarchy for CLAUDE.md

Claude loads CLAUDE.md files hierarchically:

1. `~/.claude/CLAUDE.md` - Global defaults (all projects)
2. `./CLAUDE.md` - Project root (committed, shared with team)
3. `./.claude/CLAUDE.md` - Project-specific (can be gitignored)
4. Nested `CLAUDE.md` files - Directory-specific context

Most specific wins. I use this for context that varies by directory:

```
project/
├── CLAUDE.md              # Project-wide rules
├── src/
│   ├── api/
│   │   └── CLAUDE.md      # API conventions, auth requirements
│   └── components/
│       └── CLAUDE.md      # Component patterns, accessibility
└── tests/
    └── CLAUDE.md          # Testing conventions, mocking patterns
```

### When to Reset Context

**Reset (`/clear`) when:**

- Switching to a completely unrelated task
- Context has accumulated errors or misunderstandings
- Claude keeps referencing outdated information
- Starting fresh after a major merge

**Don't reset when:**

- Iterating on the same feature
- Claude has built up useful understanding of your approach
- Debugging something Claude helped create
- Session is long but focused

### Managing Long Conversations

**Monitor proactively**: Run `/context` every 30-60 minutes on intensive sessions to see a visual grid of your context window usage.

**Proactive summarization**:

```
"We've been working on this for a while. Before we continue,
summarize our current state:
- What we've accomplished
- What's left to do
- Any decisions we've made about approach"
```

**Scope rigorously**: One feature per session. Don't context-switch. If you need to do something else, start a new session.

**Strategic compaction**: If you're at 70%+ context usage and have more work:

```
"Create a PROGRESS.md summarizing everything we've done,
our current approach, and remaining tasks. I'll use this
to seed a fresh session."
```

---

## Part 5: Best Practices by Task Type

### Code Generation Workflows

**Start with intent, not implementation:**

```
Bad:  "Create a React component with useState for count"
Good: "I need a counter component. Users should see the count,
       increment/decrement buttons, and the count should persist
       to localStorage. Match the styling in our other components."
```

The bad prompt tells Claude _how_. The good prompt tells Claude _what you need_, letting it choose appropriate implementation details.

**Provide architectural context:**

```
"This is a Next.js 14 app using the App Router. We use Zustand
for state management, Tailwind for styling, and TanStack Query
for data fetching. Create the component following these patterns."
```

**Let Claude explore first:**

```
"Before writing code, look at how we handle similar components
in src/components/. Match our existing patterns for error handling,
loading states, and prop interfaces."
```

### Refactoring and Debugging

**For refactoring—the explicit approach:**

```
"Refactor the auth module to use async/await instead of callbacks.

Requirements:
- Don't change the external API (function signatures stay the same)
- Preserve all existing behavior (tests should still pass)
- Add proper error handling where callbacks swallowed errors

Process:
1. Make a plan showing which files will change
2. Wait for my approval
3. Change one file at a time
4. Run tests after each file
5. Commit if passing"
```

**For debugging—hypothesis first:**

```
"After adding the new middleware, login fails with this error:
[paste full error with stack trace]

Context:
- The middleware is supposed to check JWT expiration
- Login worked before adding the middleware
- Other routes seem fine

Before fixing, I want you to:
1. Hypothesize three possible causes
2. Tell me how to verify which one is correct
3. Then investigate and propose a fix"
```

This prevents Claude from immediately jumping to a fix that addresses the symptom but not the root cause. I learned this the hard way—early on I let Claude just "fix" things and ended up with band-aids on top of band-aids.

### Multi-Step Automation

**Break large tasks into verifiable phases:**

```
Phase 1: "Analyze all API endpoints in src/api/. Create a document
listing each endpoint, its parameters, and return types."
[Verify output]

Phase 2: "Based on that analysis, generate OpenAPI spec for all endpoints."
[Verify spec is correct]

Phase 3: "Generate TypeScript types from the OpenAPI spec."
[Verify types compile]

Phase 4: "Update all client code to use the generated types."
[Run tests]
```

**Use git checkpoints for safety:**

```
"After each major step, commit with a descriptive message.
If something breaks, we can revert to the last working state.

Commit after:
- Schema changes
- API implementation complete
- Tests passing
- Client integration done"
```

---

## Part 6: Common Failure Patterns and Fixes

### Claude Getting Stuck in Loops

**Symptom**: Claude keeps trying the same approach, getting the same error, saying "let me try again."

**Cause**: Usually insufficient context about _why_ the approach fails, or Claude not recognizing the pattern.

**Fix**:

```
"Stop. That approach has failed 3 times. The error tells us [your insight].

Let's try something completely different: [describe alternative approach].

Don't try to fix the previous approach—abandon it."
```

I've had this happen when debugging Supabase RLS policies. Claude kept tweaking the policy syntax when the real issue was the JWT claims weren't being passed correctly. Had to explicitly say "the policy syntax is fine, the problem is upstream."

### Context Window Exhaustion

**Symptom**: Responses become confused, lose track of earlier decisions, contradict previous statements.

**Cause**: Context window full, automatic compaction losing important details.

**Fix**:

1. Run `/context` to confirm you're near limits
2. Summarize the current state explicitly in your next message
3. Consider starting fresh with a focused summary

### Model Misunderstanding Intent

**Symptom**: Claude does something technically correct but not what you wanted.

**Cause**: Ambiguous instructions, missing context about your preferences or constraints.

**Fix—be explicit about the outcome:**

```
Bad:  "Add error handling"
Good: "Add error handling that:
       - Logs errors to our existing logger (src/lib/logger.ts)
       - Shows user-friendly messages (no technical details)
       - Never exposes stack traces in production
       - Follows our error boundary pattern in ErrorBoundary.tsx"
```

### Troubleshooting Decision Tree

```
Problem occurring?
│
├─ Claude won't start
│  └─ Run: claude /doctor
│     └─ Follow its recommendations
│
├─ Commands failing
│  ├─ Permission errors → /permissions, check tool allowlist
│  ├─ Tool not found → Verify: node, git installed
│  └─ Timeout → Break into smaller operations
│
├─ MCP not connecting
│  ├─ Run: /mcp → check status
│  ├─ Check: ~/.claude/settings.json for config errors
│  └─ Verify: API keys, network access
│
├─ Poor code quality
│  ├─ Missing context → Enhance CLAUDE.md
│  ├─ Wrong patterns → Show correct examples
│  └─ Not reading files → Use @filename to force read
│
├─ Context issues
│  ├─ Confused responses → Check context usage, consider /clear
│  ├─ Forgot earlier work → Summarize state in your message
│  └─ Wrong assumptions → Explicitly correct
│
└─ Performance issues
   ├─ Slow responses → Switch to Sonnet for speed
   ├─ High cost → Be more specific, reduce scope
   └─ Long sessions dragging → Start fresh with summary
```

---

## Part 7: Integration Patterns

### MCP (Model Context Protocol) Setup

MCP extends Claude Code with external tool access. This is how Claude reaches beyond the filesystem.

**Adding servers via CLI:**

```bash
# HTTP transport (recommended for remote servers)
claude mcp add --transport http stripe https://mcp.stripe.com
claude mcp add --transport http api https://api.example.com/mcp \
  --header "Authorization: Bearer token"

# Stdio transport (for local/npx servers)
claude mcp add --transport stdio github \
  -- npx -y @modelcontextprotocol/server-github

# With environment variables
claude mcp add --transport stdio supabase \
  --env SUPABASE_ACCESS_TOKEN=your_token \
  -- npx -y @anthropic/supabase-mcp

# Managing servers
claude mcp list
claude mcp remove github
```

**Configuration in settings:**

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "ghp_..."
      }
    }
  }
}
```

Note: HTTP transport is recommended over SSE (deprecated). Check your specific MCP server's documentation for the correct package name and configuration.

**MCP servers I use:**

| Server | Purpose | Notes |
|--------|---------|-------|
| Supabase | Database queries, auth management | Essential for my client work |
| GitHub | PR management, issues | For open source contributions |

### Hooks Configuration

Hooks run shell commands at specific lifecycle points.

**Hook events:**

| Event | When | Use Case |
|-------|------|----------|
| PreToolUse | Before tool executes | Block dangerous commands |
| PostToolUse | After tool completes | Run formatters, linters |
| PermissionRequest | When permission is requested | Custom approval logic |
| UserPromptSubmit | When user submits prompt | Input validation |
| Notification | When notification occurs | External alerts |
| Stop | When Claude stops | Commit changes, cleanup |
| SubagentStop | When subagent completes | Subagent result handling |
| SessionStart | When session begins | Setup tasks |
| SessionEnd | When session ends | Cleanup, logging |

**My PostToolUse hook for formatting:**

```json
{
  "hooks": {
    "PostToolUse": [{
      "matcher": "Edit|Write",
      "hooks": [{
        "type": "command",
        "command": "pnpm exec prettier --write \"$CLAUDE_FILE_PATHS\""
      }]
    }]
  }
}
```

This means I never manually run Prettier. Every file Claude touches gets formatted automatically.

**Exit codes:**

- 0: Allow (or success for Post)
- 2: Block (PreToolUse only), message to stderr shown to Claude
- Other non-zero: Error shown to user, operation continues

### Git Workflow Integration

**In CLAUDE.md:**

```markdown
## Git Workflow
- Create feature branches from main: `feature/[description]`
- Commit messages: `[type]: [description]`
  - Types: feat, fix, docs, refactor, test, chore
- Run all tests before committing
- Never force push to shared branches
```

---

## Part 8: Performance and Cost

### Token Usage Patterns

**What consumes tokens:**

- Your messages (usually small)
- Claude's responses (can be large)
- File contents when read (can be very large)
- Tool inputs and outputs
- CLAUDE.md and context files

**Approximate token counts:**

- 1000 words = ~750 tokens
- Average TypeScript file = 500-2000 tokens
- CLAUDE.md = 200-500 tokens

**Cost relationships (ratios stay stable even as absolute prices change):**

- Opus costs roughly 10× more than Sonnet per token
- Haiku costs roughly 1/10th of Sonnet per token
- Output tokens cost 3-5× more than input tokens
- Check [Anthropic pricing](https://www.anthropic.com/pricing) for current rates

### Cost Optimization Strategies

1. **Model selection**: Start with Sonnet. Upgrade to Opus for complex reasoning, architecture decisions, or when Sonnet fails repeatedly.
2. **Scope sessions**: One feature = one session. Don't accumulate unrelated context.
3. **Be specific**: "Look at src/auth/login.ts" costs less than "search for authentication code."
4. **Use @ mentions**: `@src/utils/helpers.ts` forces reading that specific file without search overhead.
5. **Limit file reads**: "Read only the function handleLogin, not the entire file."

### Model Selection Guide

| Task | Model | Reasoning |
|------|-------|-----------|
| Quick edits, simple features | Claude 4.5 Sonnet | Fast, cheap, capable |
| Complex architecture decisions | Claude 4.5 Opus | Superior reasoning |
| Debugging tricky issues | Claude 4.5 Opus | Better hypothesis generation |
| Bulk file operations | Claude 4.5 Sonnet | Speed matters more |
| Planning major refactors | Claude 4.5 Opus | Worth the investment |
| Writing tests | Claude 4.5 Sonnet | Pattern matching is sufficient |
| Security analysis | Claude 4.5 Opus | Needs deeper thinking |

For my client website work, 90% of tasks use Sonnet. I switch to Opus when I'm doing something complex like designing a new data model or debugging a tricky authentication flow.

---

## Part 9: Security Considerations

### Default Access Scope

By default, Claude Code can access:

- All files in the project directory (and subdirectories)
- Environment variables in the shell session
- Network via bash tools (curl, wget, etc.)
- Any configured MCP servers
- Any directory added via `--add-dir`

### Permissions Configuration

```json
{
  "permissions": {
    "allow": [
      "Read(*)",
      "Write(src/**/*)",
      "Bash(pnpm *)",
      "Bash(git *)"
    ],
    "deny": [
      "Read(.env*)",
      "Read(**/secrets/**)",
      "Write(supabase/migrations/*)",
      "Bash(rm -rf *)",
      "Bash(sudo *)"
    ]
  }
}
```

**Principle**: Start restrictive, expand as needed. It's easier to allow more than to recover from accidental damage.

### Sensitive Data Handling

1. Keep secrets in `.env` files
2. Add `Read(.env*)` to deny list
3. Use environment variable references in code, never inline values
4. Never commit `.claude/settings.local.json` if it contains tokens
5. Review bash commands before approval when working with production data

For client work, I'm extra careful here. A leaked API key could compromise a client's Supabase instance. The permissions deny list is non-negotiable.

---

## Part 10: How I Actually Use Claude Code

### My Daily Workflow

**Morning start:**

```bash
cd ~/projects/current-client
claude -c  # Continue yesterday's session if still relevant
```

If starting fresh:

```bash
claude
> "Good morning. Continuing work on [feature]. Yesterday we
   completed [x]. Today I need to [y]. Let's review where we
   left off before continuing."
```

**For client website builds ($800 projects):**

1. Start with requirements from client call notes
2. "Plan this feature, don't code yet"
3. Review plan, adjust for client constraints (hosting limits, budget)
4. "Execute phase 1, stop after it works"
5. Review, push to staging, get client feedback
6. Iterate until approved

**For maintenance work ($120/month clients):**

1. Check client tickets/requests
2. Quick fixes go straight to Claude Code
3. Larger changes get planned first
4. Always test on staging before production

### My Custom Configurations

**Global CLAUDE.md** (`~/.claude/CLAUDE.md`):

```markdown
# Global Preferences

## Communication
- Be direct and concise
- Skip pleasantries and preamble
- If my approach seems wrong, say so clearly
- Ask clarifying questions before making assumptions
- Prefer "no" over hedging if something won't work

## Code Style
- Prefer explicit over clever
- TypeScript strict mode always
- Functional > class-based when equivalent complexity

## Tools
- pnpm over npm
- Prettier for formatting (don't argue about style)
- ESLint errors only (not warnings)

## Workflow
- Plan before coding for anything > 30 minutes
- Commit after each working increment
- Run tests frequently
```

### What's Working Well

- **Plan mode for anything beyond trivial**: The 5 minutes spent planning saves 30 minutes of rework
- **Session continuity with `-c`**: Essential for client projects spanning multiple days
- **Hooks for formatting**: I never manually run Prettier anymore
- **Explicit constraints in CLAUDE.md**: "NEVER modify migrations directly" has prevented disasters
- **Hypothesis-first debugging**: Forces Claude to think before acting
- **Supabase MCP**: Direct database access without leaving the terminal

### Current Limitations I'm Working Around

- **Long sessions degrade**: After ~2 hours of intensive use, quality noticeably drops. I take breaks or start fresh with a summary.
- **Complex multi-file refactors**: Still need careful human review. Claude sometimes doesn't see cascading effects.
- **Cross-repository work**: `--add-dir` helps but doesn't share CLAUDE.md context. I maintain separate sessions for different repos.
- **Pine Script**: Claude knows it but sometimes hallucinates functions. I verify against TradingView docs.

---

## Appendix: Prompt Templates

### Task Initiation (General)

```markdown
I need to [describe desired outcome].

Context:
- This is part of [feature/system/project area]
- It should integrate with [existing components]
- The user will [describe key interaction]

Constraints:
- Must work with [existing code/patterns]
- Don't modify [protected files/systems]
- Follow our [specific conventions—or see CLAUDE.md]

Start by making a plan. Don't write code until I approve the plan.
```

### Debugging (Systematic)

```markdown
I'm getting this error after [what I changed]:

```
[paste full error with stack trace]
```

Context:
- I was trying to [goal]
- What I changed: [specific changes]
- This worked before when [previous state]

Before fixing:
1. Give me 3 hypotheses for the root cause
2. Rank them by likelihood
3. Tell me how to verify each one

Then I'll tell you which to pursue.
```

### Refactoring (Safe)

```markdown
Refactor [component/module/system] to [describe goal].

Requirements:
- All existing tests must pass (no behavior changes)
- Maintain backward compatibility with [specific interfaces]
- Follow patterns in [example file or description]
- Commit after each working increment

Process:
1. Make a plan showing files that will change
2. Wait for my approval
3. Execute one file at a time
4. Run tests after each change
5. Stop if tests fail
```

### Feature Implementation

```markdown
Implement [feature name].

User story: As a [user type], I want to [action] so that [benefit].

Acceptance criteria:
- [ ] [criterion 1]
- [ ] [criterion 2]
- [ ] [criterion 3]

Technical requirements:
- Use [specific technologies/patterns]
- Include [tests/docs/migrations as needed]

Start with a technical plan, then implement phase by phase.
```

---

## Related Essays

- [[Intelligence as a Commodity]] - The water/faucet mental model for AI capability
- [[Maximizing AI Utility as a Business Owner]] - High-fidelity context, failure modes, authority decay
- [[Multi-Agent Network Orchestration]] - Using subagents effectively, blind ensemble pattern
- [[Data Monitoring and Warehousing for AI]] - What context to maintain long-term
- [[AI for Local Service Businesses]] - Practical application for web development business

---

## Verification Status (January 2026)

### Verified
- [x] `/context` - Shows context window as colored grid
- [x] `/bashes` - Manages background tasks (not `/tasks`)
- [x] `/usage` - Subscription plan limits only
- [x] `/memory` - Edits CLAUDE.md files
- [x] Background tasks: `Ctrl+B` to background, `/bashes` to manage
- [x] Extended thinking: `Option+T` / `Alt+T`, or `ultrathink` keyword
- [x] `--add-dir` syntax correct
- [x] `--output-format json` exists
- [x] `--allowedTools` and `--disallowedTools` exist
- [x] Model ID format: `claude-sonnet-4-5-20250929`
- [x] Skills: directories with `SKILL.md` files in `~/.claude/skills/`
- [x] MCP: HTTP transport recommended, SSE deprecated
- [x] Hook events: 9 events documented
- [x] Official docs: https://docs.anthropic.com/en/docs/claude-code/

### Not Verified (Author Should Confirm)
- [ ] Specific Supabase MCP package name (check your `/mcp` output)
- [ ] Hook environment variable names (`$CLAUDE_FILE_PATHS`, etc.)

---

## Resources

- [Official Documentation](https://docs.anthropic.com/en/docs/claude-code/)
- [MCP Server Directory](https://github.com/modelcontextprotocol)
- [Anthropic Engineering Blog](https://www.anthropic.com/engineering)

---

_This document is a working reference, last verified January 2026. As Claude Code evolves, some details may change. Run `/help` to see current commands, and check the official documentation for the latest features._
