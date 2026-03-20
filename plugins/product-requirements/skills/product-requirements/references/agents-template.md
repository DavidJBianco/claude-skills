# AGENTS.md Template

This is the reference structure for the AGENTS.md file. The software engineering agent should use this as a guide, adapting sections to the specific project. The document should be practical and opinionated — not generic boilerplate.

## Required Sections

- **Project overview** — Brief architecture summary referencing the PRD
- **Tech stack** — Complete list with versions where relevant
- **Project structure** — Directory layout with descriptions
- **Dependency management** — How to add/manage dependencies
- **Code style & standards** — Type hints, formatting, linting, general principles
- **Database conventions** — ORM style, migrations, computed vs. stored fields (if applicable)
- **Configuration & secrets** — What goes where, how to access config
- **Key patterns** — How to add new features following existing architecture (e.g., plugin patterns, adding new endpoints, adding new commands)
- **Testing** — How to run tests, what to test, mocking requirements
- **Common pitfalls** — Explicit do/don't list based on project-specific gotchas

## Required: Task Tracking Section

Every AGENTS.md must include this section on persistent task tracking via a `TODO.md` file:

```markdown
## Task Tracking with TODO.md

AI agents working on this project must use a `TODO.md` file for persistent task tracking:

- **Read `TODO.md` at the start of every session** to understand current project state
- **Write tasks to `TODO.md` before starting work** as a durable record alongside any in-session tracking
- **Update `TODO.md` immediately** when tasks are added, changed, completed, or removed
- Use markdown checkboxes for task status:
  - `- [ ]` for pending tasks
  - `- [ ] **IN PROGRESS**` for actively worked tasks
  - `- [x]` for completed tasks
- **Never delete completed items during a session** — keep them checked off for visibility
- Treat `TODO.md` as the **recovery mechanism for interrupted sessions** — the next agent must be able to pick up exactly where the last one left off
- Group tasks by feature area or phase
```

## Merging with Existing AGENTS.md

If an AGENTS.md already exists in the project root:
1. Read it first
2. Preserve all existing items — they represent deliberate choices
3. Build new sections around existing content, filling gaps from the PRD
4. If there's a conflict between the existing AGENTS.md and the PRD, flag it for the user rather than silently overriding either one
