<!--
Sync Impact Report:
- Version change: (none) → 1.0.0
- Added sections: All sections (initial constitution)
- Templates updated: ✅ plan-template.md, ✅ spec-template.md, ✅ tasks-template.md
- Follow-up TODOs: None
-->

# ADHD-Friendly Action-First Second Brain Constitution

## Core Principles

### I. Action-First Design
Every feature must prioritize immediate, actionable steps over planning overhead. Actions MUST be atomic, immediately doable, and context-aware. The system MUST generate engagement opportunities without requiring completion for value.

### II. ADHD-Centric UX (NON-NEGOTIABLE)
Interface MUST accommodate ADHD cognitive patterns: minimal context switching, immediate feedback, forgiving about missed days, and focus on engagement over completion. NO guilt-inducing language or punitive backfill mechanisms.

### III. Deterministic Core
System MUST be fully functional without AI dependencies. Core behaviors (actions, routines, tasks, projects) MUST operate deterministically and be independently testable. AI features MUST be optional enhancements only.

### IV. Cadence-Based Simplicity
Routines MUST generate actions once per cadence period without exception. Complex scheduling logic MUST be rejected in favor of simple, predictable patterns. Missing days MUST NOT create debt or complex catch-up mechanisms.

### V. Engagement Metrics
Success MUST be measured by engagement frequency and action initiation, NOT completion rates. Analytics MUST track user interaction patterns that encourage continued use without shame or pressure.

## ADHD-Specific Constraints

Technology and design MUST accommodate neurodivergent users:
- Minimal cognitive load per interaction
- Forgiving about inconsistent usage patterns
- No punishing language for missed routines
- Immediate value from minimal effort
- Offline-first capability where possible

## Development Workflow

Test-First Development mandatory for core domain logic:
- Unit tests for all action/routine/task behaviors
- Integration tests for user journeys
- Accessibility testing for ADHD compliance
- Performance testing for engagement responsiveness

## Governance

This constitution supersedes all other project practices. Amendments require:
- Documented impact analysis on ADHD user experience
- Updated test coverage for affected behaviors
- Migration plan preserving user engagement data
- Review compliance with existing ADHD accessibility standards

All code reviews MUST verify constitution compliance. Complexity increases MUST be explicitly justified against simplicity principles. Use README.md and architecture.md for runtime development guidance.

**Version**: 1.0.0 | **Ratified**: 2026-02-08 | **Last Amended**: 2026-02-08