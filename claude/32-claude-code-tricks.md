# 32 Tricks to Level Up Claude Code

> Summary of Nate Herk's video: [32 Tricks to Level Up Claude Code in 16 Mins](https://www.youtube.com/watch?v=jqoFP9QapXI)

---

## Beginner Hacks (1-10)

### 1. Run `/init` on every project
Claude scans your entire codebase and generates a `CLAUDE.md` file. It maps your architecture, conventions, and key files automatically so you don't need to re-explain your project each session.

### 2. '/statusline' Set up a status line 
Type `/status line` in the terminal to get a mini dashboard showing your model, context %, and cost in real time. Helps you spot context rot before it silently kills your session.

### 3. Use voice input
Claude Code ships a native `/voice` command. Talk to your terminal and it codes. A massive unlock for hands-free workflow.

### 4. Keep your context small
Don't dump your entire codebase into one conversation. Give Claude only what it needs for the current task. Less noise = dramatically better outputs.

### 5. `/context` to find your token bloat
Type `/context` to see exactly what's eating your tokens -- system prompts, file contents, MCP servers. Diagnose it, fix it, stay lean.

### 6. Compact at 60%, clear between tasks
When context hits ~60%, run `/compact`. You can tell it what to preserve: e.g. "Compact, but keep all API integration decisions and database schema." Use `/clear` when switching to a completely different task.

### 7. Always start in Plan Mode
Hit `Shift + Tab` to enter plan mode. Claude reads, researches, and outlines steps but won't touch a single file. Once you like the plan, switch out of plan mode and say "execute." Cuts revision rounds in half.

### 8. Treat Claude like a junior developer
Don't say "Write a function that does X." Say "How should we handle growth tracking?" When Claude reasons through the problem first and makes its own decisions, output quality jumps noticeably.

### 9. Make Claude ask YOU questions
Tell it: "Keep asking me questions until you're 95% confident you understand exactly what I need." Front-loading alignment saves 3-4 rounds of painful back-and-forth revisions.

### 10. Build self-checking into to-do lists
Tell Claude to build the page, screenshot it, check Chrome DevTools for errors. Add the rule: "Don't move to the next task until you're 95% confident this one is complete."

---

## Intermediate Hacks (11-22)

### 11. Deploy sub-agents for parallel work
Tell the main session to spin up sub-agents. Each gets its own context window and model. They work in parallel and report back. Like having a full team of developers.

### 12. Build custom skills
Create reusable `.md` files in your `.claude/skills/` directory (e.g. `techdebt.md`, `codereview.md`). Invoke them in plain English. Commit to GitHub so your whole team has access.

### 13. Use Haiku for sub-agents
Main thread on Opus, sub-agents on Haiku. When a sub-agent needs to scrape articles or read hundreds of thousands of tokens and return a summary, Haiku is cheap and fast enough. Save the big spend for where quality matters.

### 14. Constantly refresh your CLAUDE.md
After every session, log new patterns, gotchas, and conventions. But keep it under 150-200 lines max -- every line loads into your context window every single time.

### 15. Route CLAUDE.md to other files
Keep `CLAUDE.md` lean. Link it out to style guides, business context docs, and reference files. Claude knows where to look without loading everything upfront.

### 16. Exit early and re-ask
If Claude starts going the wrong direction, hit Escape immediately. Every token spent going the wrong way is wasted context. Steer tight, steer early.

### 17. Challenge outputs aggressively
Say things like "Scrap that. Do a more elegant version" or "This isn't good enough -- try a completely different approach." Claude almost always comes back stronger on the second try. Then tell it to update the skill or CLAUDE.md so it doesn't repeat the mistake.

### 18. `/rewind` for quick undos
Wrong turn? Type `/rewind` and Claude rolls back to a previous point in the conversation. No restart, no lost work.

### 19. Hooks for notifications '/hooks'
Set up a notification hook so Claude sends you an actual sound notification when a session finishes. Run 15 parallel sessions, do other work, and listen for the ping.

### 20. Use screenshots
Claude can see. Feed it error messages, inspiration websites, competitor UIs. Build a loop: screenshot, analyse, fix, screenshot again. By V1 handoff, Claude has already done 3 passes.

### 21. Use Chrome DevTools
Claude can open a browser, click buttons, fill forms, and check functionality. Huge for front-end work. If there's no explicit API, Claude can navigate manually.

### 22. Clone inspiration sites
Screenshot any site you love, feed it to Claude: "Make it look like this." Claude recreates the design patterns without generic AI output. You can even feed the actual HTML styling as a starting template.

---

## Pro Hacks (23-32)

### 23. Parallel sessions with Git Worktrees
Run 3-5 Claude sessions on the same project with zero interference. Each works on its own isolated branch. When done, merge them back like any normal Git branch. Command: `claude --worktree [feature-name]`

### 24. API endpoints vs. MCP servers
MCP servers load their entire tool list into your context window. If you only need to read one Notion database, hardcode that single API endpoint instead. Saves a massive number of tokens on focused tasks.

### 25. `/loop` for recurring tasks
"Every 5 minutes, check on the deployment." Claude reruns the prompt in the background and only interrupts you when something needs attention. Can also set one-time natural language reminders. Loops last up to 3 days.

### 26. Host on a VPS for always-on sessions
Run Claude Code on a remote server. It stays alive even when your laptop is closed. SSH in whenever you need to steer it, or control it through Telegram from anywhere. Perfect for long-running tasks.

### 27. Remote control from your phone
Start a task at your desk, walk away, keep steering from your phone via browser. Your code never leaves your local machine -- only the remote connection travels with you.

### 28. No-SQL data analytics
Connect CLI tools like BigQuery's BQ tool to Claude Code. Ask in plain English: "What were our top 10 revenue sources last quarter?" Claude writes the query, runs it, and returns the answer. No SQL required.

### 29. Ultrathink for hard problems
Type `ultrathink` and Claude goes all-out, allocating up to 32K thinking tokens before responding. Don't use it for simple fixes. Use it for architecture decisions, complex debugging, or when after two tries it's still not right.

### 30. Edit permissions for safe autonomy
Don't use `--dangerously-skip-permissions` blindly. Instead, explicitly ALLOW commands you know are safe and DENY destructive ones like deletes and removes. Same speed, actually safe. Deny list always takes priority over allow list.

### 31. Agent teams
Unlike sub-agents (parallel but isolated), agent teams can talk to each other, share a task list, communicate freely, and assign each other work. You can talk to each individual agent directly. More expensive and longer-running, but output quality is in a different league for big complex projects.

### 32. Context7 MCP
Claude's training data has a cutoff, so it sometimes suggests deprecated or renamed functions. Context7 MCP injects current, version-specific documentation from thousands of popular libraries (Next.js, React, MongoDB, etc.) before Claude writes a single line of code. One command to install, instant quality upgrade.

---

## Quick Recap by Level

**Beginner:** `/init` > Status line > Plan Mode > Keep context small > Self-check to-dos

**Intermediate:** Sub-agents > Custom skills > CLAUDE.md routing > Challenge outputs > Screenshot loops

**Pro:** Git Worktrees > Ultrathink > Agent teams > Context7 MCP > VPS always-on
