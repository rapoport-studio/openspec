# Chat redesign — adjacent fields & emerging trends (source stream 4 of 4)

> 2026-05-14 · feeds [synthesis](chat-redesign-synthesis.md) · sister streams: [patterns](chat-redesign-patterns.md), [a11y](chat-redesign-a11y.md), [voice](chat-redesign-voice.md).
>
> Coverage: patterns borrowable from email clients, comment threads, terminal/REPL, messaging apps; 2026 trends (memory UX, branching, citations, multi-author, agent supervision).

---

## 1. Adjacent fields — what's borrowable

**Email clients.** Superhuman's grammar is the gold standard for keyboard-first authoring: `Cmd-K` is the universal escape hatch ("if you don't know it, hit `Cmd-K` and type what you want"), `c` composes, `/` searches, `j/k` traverses, single letters (`e` archive, `h` reply-all) make triage almost percussive. Reported gains: ~2x throughput, ~4 hours/week. Transferable rule: **every visible action has a letter; `Cmd-K` is the safety net for everything else.** Shortwave (AI-native) shows the inverse trap — when *every* action goes through AI, users lose proprioception. HEY's "Imbox/Feed/Paper Trail" split inbox: a single stream is wrong for high-throughput; classify and split.

**Comment threads.** Linear/Figma/Google Docs converge on the same primitives: (a) **resolved vs unresolved as a first-class filter**, (b) reply-in-place but no recursive branching (Atwood's "flat by design" is the consensus — even threaded forums disable trees by default), (c) `@mention` notifies + assigns context, (d) inline-anchored to an object, not floating. Figma users keep asking to *hide resolved by default* — borrow that as a setting, not a button.

**Terminal & REPL.** Warp's "blocks" (each command + output is an addressable, copyable, shareable unit) is the most underrated chat pattern of 2025 — turns the log into navigable items, not a wall of text. `Ctrl-Shift-P` palette + inline AI completions + a separate Agent Mode is the **four-tier interaction model**: keyboard → completion → command → agent. Claude Code and Aider show the input-grammar pattern worth stealing: bash mode toggle, side-questions mid-response, vim editor inside the input box.

**Messaging.** Telegram's quote-reply (highlight-to-quote, not whole-message reply) plus per-message reactions with animations is the throughput unlock — replies stay precise without thread explosion. iMessage's edit-with-history (15-min window, "Edited" tap-to-see-original) is the trust-preserving pattern.

## 2. Slash vs Cmd-K

The split is now clear: **`/` is for insertion-into-this-message** (Notion blocks, Slack apps, Discord); **`Cmd-K` is for navigation/global action** (Linear, Superhuman, Raycast, Notion's other palette). Hybrid fails when products conflate them. Practical rule for B2B chat: `/` mentions an inline tool ("send file," "summarize thread"), `Cmd-K` jumps anywhere in the workspace. Don't put model-picker or thread-switch behind `/` — that's a `Cmd-K` job.

## 3. Memory / context UX

State of the art is **visible, source-attributed, scoped**. ChatGPT (May 2025) added a "Sources" pill under each response listing which saved memories / past chats / custom instructions shaped it, with inline edit/delete. Simon Willison's critique remains the canonical UX brief: invisible global memory bleeds personal context into work; the fix is **project-scoped memory** (which OpenAI then shipped). Claude Projects nails the visible-sidebar model — files, instructions, chat list, all on the left. Cursor's `.cursorrules` (file-as-config) is the developer-friendly extreme: memory you can `git diff`. For B2B workspace: surface memory the way Claude does — a sidebar pane, scoped to the workspace, editable as text, with per-response "what shaped this" affordance.

## 4. Branching / forking

Three competing mental models, none decisively winning:

- **Linear with edit-reroll** (Claude.ai): edit a past user message → previous branch becomes a hidden sibling, accessible via `←/→` chevrons. Lowest cognitive load; users barely notice it's a tree.
- **Explicit tabs in parallel** (Cursor 2.0, `Cmd-T`): each branch is a tab, runs in parallel, shows in the Agents panel. Power-user pattern.
- **Tree view** (OpenAI Playground, forum requests): visible graph. Consistently requested, consistently bounces in usability — users get lost above ~3 splits.

Converging best practice for 2026: **linear-with-checkpoints as default, tabs for parallel exploration, tree view never as default.** Cursor's "Fork Chat" in the right-click menu is the cleanest discovery surface.

## 5. Citations / source attribution

Convergence in 2026: **inline numbered chips + a Sources panel as the second look**. Perplexity (inline `[1]` with favicon-on-hover, expandable Sources panel) is the de facto standard. Pattern winning: **(a) inline chip at the claim, (b) favicon+title preview on hover, (c) Sources panel for full list, (d) clickable filter by domain.** The "footnote-only at bottom" approach (Copy.ai) is losing — users won't scroll.

## 6. Multi-author (human + human + AI)

The pattern that works: **AI is a labeled participant, not a separate UI region.** Slack's design guidance is explicit — agent avatar must be visually non-human, name carries the agent's purpose, and the agent uses message reactions (✅, 👀) to show progress. Linear-in-Slack shows the canonical move: agent reacts ✅ when it creates the issue, posts a confirmation message, then is silent. Discord's Clyde quietly died because it was an undifferentiated peer in the stream. **Convergent rules:** (1) non-human avatar shape (square, not round), (2) distinct color/accent, (3) "AI" or agent name as prefix in dense views, (4) reactions for status, messages only for substance, (5) AI replies thread-by-default in busy channels.

## 7. Mode / model / persona switching

ChatGPT's August 2025 model-picker resurrection (forced unification → user backlash → picker returns with personalities: Default/Cynic/Robot/Listener/Nerd) is the cautionary tale: **hide the picker and power users revolt; show it and casuals get paralyzed.** Cursor's mode toggle (Ask/Agent) lives inline in the composer; Claude's selector sits at the top of the chat. Convergent placement: **inline in the composer for "what is this message," top-of-chat or sidebar for "what is this thread."** Persona toggles work when they're a property of the thread, not a per-message dropdown.

## 8. Notable 2025–2026 launches

- **Cursor 2.0 (Nov 2025)** — multi-agent tabs, fork-chat, split-view diff. Positive for tabs, mixed for "panels fighting itself."
- **ChatGPT memory dossier + sources (May 2025)** — initially negative (Willison), then corrected with project scoping and visible sources.
- **Claude Artifacts side-by-side update (late 2025)** — strongly positive; "no more scrolling a mile-long history."
- **Granola Series C ($1.5B valuation, 2025)** — Recipes (prompt shortcuts) + collaborative chat-over-meetings.
- **Dia browser (June 2025)** — contextual sidebar chat replacing Arc; lukewarm — Arc users feel abandoned.
- **Devin 2.0 (April 2025) / Replit Agent 4 (March 2026)** — parallel task forking with ~90% auto-merge; mixed on supervision UX.
- **ChatGPT model-picker U-turn (Aug 2025)** — negative for the removal, positive for the return.

## 9. Trends to watch (2026)

- **Multi-agent panels with explicit supervision UI** (Devin's "managed Devins," Replit Agent 4 forking, Cursor tabs) — the supervision/verification surface is the unsolved UX problem; "2025 was the year of agents, 2026 is the year of harnesses."
- **Canvas/side-by-side as default**, not opt-in (Claude Artifacts, ChatGPT Canvas, v0). Chat-only window becoming minority case.
- **Ambient/background agents** (Lindy, GitHub Agents, Microsoft "change-driven architecture") — request/response giving way to monitor/react; the chat becomes an inbox of agent activity.
- **Voice-first inline** (Wispr Flow's hotkey-anywhere model, OpenAI realtime) — voice becoming an input channel layered onto existing UIs.
- **"AI HUD" over "AI copilot"** (Geoffrey Litt) — squigglies, inline suggestions, no dialog box. Strongest minority position in 2026 design discourse.

## 10. What "minimal" means in 2026

The most-cited essays converge on a thesis: **chat is a bad default for knowledge work; the affordance-free textbox is the original sin.** Amelia Wattenberger's *Why Chatbots Are Not the Future* names four faults — no affordances, buried context, isolated responses, no comparison view — and recommends: contextual controls (dropdowns/sliders for parameters), side-by-side comparison, low latency, explicit "wrong-use" cues, human-in-control. Geoffrey Litt's *Enough AI Copilots! We need AI HUDs* (July 2025) is the most-shared minimal-AI piece of the year: the spellcheck red-squiggle is the ideal — no dialog, no asking, just ambient enhancement. Linus Lee (Notion) frames it as "what is chat *good* for?" — concluding: not synthesis, not understanding, not most knowledge work; chat is good for ambiguous, exploratory, conversational tasks. Simon Willison's memory-dossier post is the canonical critique of invisible context.

**Operating definition for "minimal" in B2B 2026:** single composer, keyboard-first grammar, visible scoped memory, inline citations, one optional side panel for artifacts, AI participants labeled but not isolated, no recursive threading, branching as linear-with-checkpoints. Strip everything else.

## Sources

- [Superhuman Keyboard Shortcuts](https://superhuman.com/products/mail/shortcuts)
- [Coding Horror — Flat by Design (threading)](https://blog.codinghorror.com/web-discussions-flat-by-design/)
- [Warp Agentic Development Environment](https://www.warp.dev/)
- [Cursor 2.0 changelog — multi-agent + fork-chat](https://cursor.com/changelog/2-0)
- [Figma — View and manage comments](https://help.figma.com/hc/en-us/articles/360041547593-View-and-manage-comments)
- [Slack — Implementing slash commands](https://docs.slack.dev/interactivity/implementing-slash-commands/)
- [Notion — Using slash commands](https://www.notion.com/help/guides/using-slash-commands)
- [OpenAI — Memory and new controls](https://openai.com/index/memory-and-new-controls-for-chatgpt/)
- [Simon Willison — I really don't like ChatGPT's new memory dossier](https://simonwillison.net/2025/May/21/chatgpt-new-memory/)
- [Anthropic — Claude Projects](https://support.claude.com/en/articles/9519177-how-can-i-create-and-manage-projects)
- [ShapeOfAI — Citation patterns](https://www.shapeof.ai/patterns/citations)
- [Slack — Agent design guidelines](https://docs.slack.dev/concepts/agent-design/)
- [VentureBeat — OpenAI Canvas vs Claude Artifacts](https://venturebeat.com/ai/openai-launches-chatgpt-canvas-challenging-claude-artifacts)
- [Cognition — Devin 2025 performance review](https://cognition.ai/blog/devin-annual-performance-review-2025)
- [Replit 2025 in review](https://blog.replit.com/2025-replit-in-review)
- [Amelia Wattenberger — Why Chatbots Are Not the Future](https://wattenberger.com/thoughts/boo-chatbots/)
- [Geoffrey Litt — Enough AI Copilots! We need AI HUDs](https://www.geoffreylitt.com/2025/07/27/enough-ai-copilots-we-need-ai-huds)
- [Microsoft — Beyond the Chat Window: Ambient AI Agents](https://techcommunity.microsoft.com/blog/linuxandopensourceblog/beyond-the-chat-window-how-change-driven-architecture-enables-ambient-ai-agents/4475026)
- [TechCrunch — ChatGPT's model picker is back](https://techcrunch.com/2025/08/12/chatgpts-model-picker-is-back-and-its-complicated/)
- [Telegram — Replies 2.0 (quote-replies)](https://telegram.org/blog/reply-revolution)
- [Latent Space — Building AI×UX with Linus Lee](https://www.latent.space/p/ai-interfaces-and-notion)
- [Prototypr — Claude Code: the terminal is the new chatbox](https://prototypr.io/post/claude-code-cli-ui)
- [TidBITS — Dia Browser debuts](https://tidbits.com/2025/06/20/dia-browser-debuts-with-contextual-ai-chat-but-arc-users-feel-left-behind/)
- [TechCrunch — Granola raises $43M, adds collab](https://techcrunch.com/2025/05/14/ai-note-taking-app-granola-raises-43m-at-250m-valuation-launches-collaborative-features/)
