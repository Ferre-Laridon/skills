---
name: prd-to-plan
description: Turn a PRD into a multi-phase implementation plan using tracer-bullet vertical slices, saved as a local Markdown file in ./plans/. Use when user wants to break down a PRD, create an implementation plan, plan phases from a PRD, or mentions "tracer bullets".
---
trig: break-down-PRD·create-implementation-plan·plan-phases·tracer-bullets
in: PRD-in-context·codebase

## Process

### 1. Confirm the PRD is in context
PRD should be in the conversation. If not, ask the user to paste it or point to the file.

### 2. Explore the codebase
Understand current architecture, existing patterns, and integration layers.

### 3. Identify durable architectural decisions
Before slicing, note decisions unlikely to change: route structures, DB schema shape, key models, auth approach, third-party boundaries. These go in the plan header.

### 4. Draft vertical slices
Break the PRD into **tracer bullet** phases — each a thin vertical slice through ALL layers (schema, API, UI, tests). See [REFERENCE.md](REFERENCE.md) for slice rules and the plan file template.

### 5. Quiz the user
Present phases as a numbered list (title + user stories covered). Ask:
- Does granularity feel right? Should any phases merge or split?

Iterate until approved.

### 6. Write the plan file
Create `./plans/` if needed. Write `./plans/{feature-name}.md` using the template in [REFERENCE.md](REFERENCE.md).
