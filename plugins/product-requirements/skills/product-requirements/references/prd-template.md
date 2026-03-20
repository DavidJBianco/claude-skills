# PRD Template

Use this skeleton as the starting structure for every PRD. Adapt it to the project — skip sections that don't apply, add sections that do. The goal is a consistent, navigable document that an AI coding agent can implement from without asking clarifying questions.

```markdown
# PRD: [Project Name]

## 1. Overview
Brief description of what the application does and why it exists.

## 2. Goals & Non-Goals
### Goals
What this project aims to achieve for MVP.
### Non-Goals
What is explicitly out of scope (but may be revisited later).

## 3. Target Users
Who uses this and in what context.

## 4. Functional Requirements
### 4.1 Core Workflows
Step-by-step user workflows for key features.
### 4.2 Data Model
Tables with field names, types, constraints, and descriptions.
### 4.3 API / Interface Definitions
Endpoints, CLI commands, or other interfaces with exact signatures.

## 5. Non-Functional Requirements
Performance, security, accessibility, reliability expectations.

## 6. Technical Architecture
### 6.1 Tech Stack
Specific technologies with versions where relevant.
### 6.2 Project Structure
Directory layout with descriptions.
### 6.3 Configuration & Secrets
What's configurable, where config lives, how secrets are handled.

## 7. UI/UX
Key screens/views, interaction patterns, empty states, error states.
(Skip for non-UI projects.)

## 8. Error Handling & Edge Cases
What can go wrong and what happens when it does.

## 9. Testing Strategy
What to test, how to test it, what frameworks to use.

## 10. MVP Scope & Future Considerations
What's in v1, what's planned for later, and architectural decisions
that must not block future features.
```

## Formatting Guidelines

- Use numbered sections and clear subsections for navigability
- Use tables for data models, configuration items, and API endpoints
- Include code snippets for interfaces and data structures where they clarify the design
- Be specific: "SQLite" not "SQLite or PostgreSQL"; "15-second timeout" not "reasonable timeout"
- Define behavior for every edge case — if a developer would have to guess, it's not specific enough
