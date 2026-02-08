# Domain Model

## Core Concepts

### Action
- Atomic, immediately doable step
- Lifecycle: available, completed, postponed, rejected
- Has context and estimated effort

### Routine
- Recurring definition of behavior
- Contains action templates
- Has cadence (daily, weekly, monthly)
- Generates actions when triggered, once per cadence

### Task
- One-off obligation
- Must be fulfilled via actions
- Can be broken down incrementally

### Project
- Groups tasks
- Provides context
- Does not generate actions
