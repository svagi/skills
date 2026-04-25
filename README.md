# Agent Skills

A collection of agent skills that extend capabilities across planning, development, and tooling.

- **grill-me** — Get relentlessly interviewed about a plan or design until every branch of the decision tree is resolved.

  ```
  npx skills@latest add svagi/skills/grill-me
  ```

- **design-an-interface** — Generate multiple radically different interface designs for a module using parallel sub-agents.

  ```
  npx skills@latest add svagi/skills/design-an-interface
  ```

- **improve-codebase-architecture** — Find deepening opportunities in a codebase. Surface architectural friction and propose refactors that turn shallow modules into deep ones.

  ```
  npx skills@latest add svagi/skills/improve-codebase-architecture
  ```

- **simplify** — Simplify and refine recently modified code for clarity, consistency, and maintainability while preserving functionality.

  ```
  npx skills@latest add svagi/skills/simplify
  ```

- **tdd** — Test-driven development with a red-green-refactor loop. Builds features or fixes bugs one vertical slice at a time.

  ```
  npx skills@latest add svagi/skills/tdd
  ```

- **to-prd** — Turn the current conversation context into a PRD saved as a Markdown file in `./plans/`. No interview — just synthesizes what you've already discussed.

  ```
  npx skills@latest add svagi/skills/to-prd
  ```

- **to-issues** — Break a plan, spec, or PRD into independently-grabbable tasks saved as Markdown files in `./plans/`, using tracer-bullet vertical slices.

  ```
  npx skills@latest add svagi/skills/to-issues
  ```