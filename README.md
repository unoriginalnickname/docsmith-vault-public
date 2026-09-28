# docsmith-vault

A system for producing and maintaining documentation where every claim can be traced
back to a source you can open.

You give it a subject. It gathers sources from the web and from video, saves each one
word for word, and writes documents where every statement points at the source and line
it came from. It also notices when a source has changed since it was saved.

> **The code isn't public yet.** This page describes what it is, how it works and where
> it's going. I'm happy to show it running.

## Why

Documentation drifts away from how things actually work, and the drift is invisible until
someone follows a stale instruction. The same problem sits under AI tools: a model answers
from its training data, which may be out of date. An answer is only as good as the
information behind it, so that information has to be current and checkable.

## What it does that others don't

Compared with four published deep-research systems (Stanford's STORM, GPT Researcher,
OpenAI Deep Research and Anthropic's Research). None of them documents a step that checks
its claims, and none groups sources by independence; GPT Researcher takes the most
frequent answer as the true one. Everything below is on purpose, in the rules and tools:

1. **Counts independent sources, not links.** Ten sites copying the same thing count as
   one source.
2. **Keeps the source's exact words.** Nothing summarises on the way in, so everything can
   be quoted.
3. **Follows the attribution to the origin.** When a page says "according to X", it reads
   X and checks that X really says it.
4. **Keeps contradictions.** When two sources disagree, both stay, and neither is quietly
   picked.
5. **Never invents.** A missing number becomes an open question, not a guess.
6. **Marks shaky claims where they stand**, so a reader sees at once what is uncertain.
7. **Can check whether a source has changed** since it was saved: it keeps the page's
   original bytes and compares. Built, but not yet run on a real subject.
8. **Runs in an isolated container**, so a web page that tries to trick the AI can't reach
   the machine.

In short: the others trust the majority. This checks whether the majority just copied
each other.

## How it works

Three stages, each feeding the next:

1. **Gather.** Each source (web page, PDF, video, podcast) is fetched and saved verbatim,
   one file per source. Nothing summarises on the way in, because a summary can't be
   quoted. Video and podcasts go through
   [docsmith-transcript](https://github.com/unoriginalnickname/docsmith-transcript).
2. **Corroborate.** An agent reads all sources side by side and writes an entry. It counts
   **independent** sources, not URLs: four sites reprinting the same wiki count as one.
   When sources disagree, both readings are kept.
3. **Write.** The document a reader actually uses is written from the entry. Shaky claims
   are marked where they stand. The sourcing lives in a separate file.

**The rules it rests on:**

- **Never invent.** A number no source gives becomes an open question, not a guess.
- **Follow the attribution.** When a page says where it got something, read the origin. It
  may not say what is attributed to it.
- **An absence is a claim too.** "The source doesn't say" gets checked as carefully as
  anything else.

**What it's made of:** a set of rules an AI coding session (Claude Code) reads at start-up,
five agent definitions, and 26 small Python command-line tools with their own test suite.
One agent gathers a source, one writes the entry, and three review a finished document
from different angles (readability, format, facts) without seeing each other's findings.

## Tech stack

| | |
|---|---|
| **Python 3.10+, uv** | The 26 command-line tools |
| **Claude Code** | The agents and the automated research loop |
| **Docker (dev container) with a firewall** | Isolation for research sessions |
| **Git hooks** | A leak check on every commit |
| **pytest, GitHub Actions** | 444 tests for the tools |
| **trafilatura, pypdf** | Exact text out of web pages and PDFs, no model involved |
| **Markdown with YAML frontmatter** | The corpus format; readable in Obsidian |

## Running it in Docker

An AI session that reads web pages can be tricked by text on a page. Run directly on a
machine, it can reach everything: files, SSH keys, the GitHub login. So research runs in
a dev container that sees only what it needs.

| Control | What it does |
|---|---|
| Isolation | The container sees only the rules, the corpus and a read-only transcript tool. No GitHub token, no SSH key, no home folder |
| Firewall | Web traffic out only (ports 80 and 443). No SSH, no local network |
| Read-only controls | Everything the host later runs is read-only inside: the tools, the hooks, the security settings, git's config. A tricked session can't plant code |
| Agent guard | Each agent may run only its own commands and write only where its job needs |
| Blocked commands | No session may make a repo public, delete repos or force-push |
| Leak check | A hook on every commit stops anything that looks like it came from the private corpus |

**The principle:** security lives in code and isolation, never in an instruction alone. A
control counts as working only once a live test has seen it refuse something.

**Accepted risk:** data can leave the container over the web. The firewall was opened to
the whole web because research keeps finding new sites. The container still protects the
host machine.

## Automated research

A loop runs research passes headless inside the container. Each pass has a spending cap,
logs everything it does, and commits its result. It stops when it can't get further
without a human. Because a headless pass can't ask questions, the goal is pinned down
first in an up-front interview.

## Where it's going

1. **Make the research loop cheaper.** It has run eight real passes on one subject, which
   isn't finished yet, all on the most expensive model. Next is comparing a cheaper model
   on cost per finished task.
2. **Maintenance, the other half of the goal.** Today it builds documentation. Next, it
   should check other people's documents: look up each claim in the corpus and mark it
   backed, contradicted or missing, with how many independent sources stand behind it.
3. **A language reviewer**, checking wording and terms across several documents.
4. **Close the remaining security gaps.** The container only protects the machine when the
   AI runs inside it. Started directly on the machine, it can still reach everything, so
   the aim is that research never runs outside the container.

## Related

Three projects are attempts at the same goal, with different architectures:

- [docsmith](https://github.com/unoriginalnickname/docsmith): the oldest. A Blazor app over
  a Python pipeline that audits a project's documentation and proposes fixes for a human to
  approve. Checking the docs against the source code was the goal; it didn't get that far.
  Runs on a local model (Ollama) or a hosted one.
- [fresh-eyes-reader](https://github.com/unoriginalnickname/fresh-eyes-reader): AI agents
  that write and review documentation on a subject, beginner to advanced.
- **docsmith-vault**: this one, and the one that works end to end today.
