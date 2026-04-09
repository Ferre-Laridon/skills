---
name: write-a-prd
description: Create a PRD through user interview, codebase exploration, and module design, then submit as a GitHub issue. Use when user wants to write a PRD, create a product requirements document, or plan a new feature.
---
trig: write-PRD·create-PRD·product-requirements·plan-new-feature
in: problem-description·codebase

Go through the steps below. You may skip steps if not necessary.

1. Ask the user for a long, detailed description of the problem and any potential solution ideas.
2. Explore the repo to verify their assertions and understand the current state of the codebase.
3. Interview the user relentlessly about every aspect of the plan until you reach a shared understanding. Walk down each branch of the design tree, resolving dependencies one-by-one.
4. Sketch out the major modules to build or modify. Look for opportunities to extract **deep modules** — ones with a simple, testable interface hiding significant complexity. Confirm with the user which modules they want tests written for.
5. Once you have a complete understanding, create a GitHub issue using `gh issue create` with the PRD. Use the template in [templates/prd-issue.md](templates/prd-issue.md).
