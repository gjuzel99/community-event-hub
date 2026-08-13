# Community Event Hub Engineering Guidelines

This document defines the initial engineering guidelines for Community Event Hub.

The goal is to keep the project maintainable, understandable, and easy to extend without introducing unnecessary complexity.

These guidelines will evolve as the project grows.

## 1. Repository Structure

Code and project artifacts should be organised by responsibility.

The initial target structure is:

```text
community-event-hub/
│
├── docs/
│   ├── architecture.md
│   └── engineering.md
│
├── solutions/
│   └── CommunityEventHub/
│
├── src/
│   ├── plugins/
│   │   ├── CommunityEventHub.Plugins/
│   │   ├── CommunityEventHub.BusinessLogic/
│   │   └── CommunityEventHub.Plugins.Tests/
│   │
│   ├── webresources/
│   │   ├── forms/
│   │   ├── shared/
│   │   ├── services/
│   │   └── utils/
│   │
│   └── pcf/
│       ├── shared/
│       └── controls/
│
├── README.md
└── ROADMAP.md
```

Folders should be created only when the project actually needs them.

The repository should not contain large amounts of empty structure only to make the project look more advanced.

## 2. Solution and Publisher

Community Event Hub should use an intentional Power Platform solution and publisher.

Guidelines:

- create a dedicated publisher for the project;
- use one stable publisher prefix;
- do not use the default publisher for project components;
- create project components inside the Community Event Hub solution;
- use unmanaged solutions during development;
- use managed solutions for downstream environments when deployment is introduced;
- avoid manually modifying managed components in downstream environments;
- publish required changes before creating deployment artifacts.

Example:

```text
Publisher: Community Event Hub
Prefix: ceh
```

Example schema names:

```text
ceh_event
ceh_session
ceh_room
```

The exact solution structure may evolve as the project grows.

## 3. Naming Conventions

Names should be consistent and should describe business purpose rather than implementation details.

Examples:

### Tables

```text
Event
Session
Session Review
Room
Track
```

### Schema Names

```text
ceh_event
ceh_session
ceh_sessionreview
ceh_room
```

### Cloud Flows

```text
CEH - Session - Send Approval Notification
CEH - Event - Send Organizer Reminder
```

### Connection References

```text
ceh_Dataverse
ceh_Outlook
```

### Environment Variables

```text
ceh_BaseUrl
ceh_NotificationMailbox
```

Naming conventions should remain simple and predictable.

## 4. Dataverse Table and Relationship Design

Before creating a custom Dataverse table, first check whether a standard table or platform capability already represents the required concept.

Examples of standard tables that may be reused include:

- Account;
- Contact;
- Appointment;
- Email;
- Task;
- Phone Call.

Custom tables should be created for real Community Event Hub domain concepts such as:

- Event;
- Session;
- Room;
- Track;
- Session Review.

Standard tables should be reused when they fit the domain, but they should not be forced to represent concepts they were not designed for.

### Relationship Guidelines

Before creating a relationship:

1. Check whether a suitable relationship already exists.
2. Understand the business meaning of the relationship.
3. Decide the correct cardinality.
4. Avoid duplicate relationships that represent the same business concept.
5. Give each relationship a clear purpose.

Example questions:

```text
One Event → many Sessions?

One Session → many Speakers?

One Speaker → many Sessions?

Does this require 1:N or N:N?
```

Relationships should be designed from the business domain first, not only from what is easiest to configure.

Avoid creating multiple similar relationships unless they represent genuinely different concepts.

## 5. Connection References and Configuration

Connection references should be created intentionally inside the solution.

Do not rely on automatically generated connection references when an existing project connection reference should be reused.

Guideline:

> Reuse the same connection reference when the connector, identity, and purpose are the same.

Examples:

```text
ceh_Dataverse
ceh_Outlook
```

Create another connection reference only when a different identity, connection, or technical purpose requires it.

Environment-specific values should not be hard-coded.

Use appropriate configuration mechanisms such as:

- environment variables;
- connection references;
- configuration records where required.

## 6. C# Plugins

C# plugins should be kept small and focused.

The plugin entry point should mainly be responsible for:

- reading the execution context;
- validating execution conditions;
- accessing required Dataverse services;
- calling the required business logic;
- returning or throwing the appropriate result.

Business logic should not be placed directly into very large plugin `Execute` methods.

Preferred direction:

```text
Dataverse
    ↓
Plugin Entry Point
    ↓
Business Logic / Service
    ↓
Dataverse Operations
```

SOLID principles should be applied where they improve maintainability.

The project should not introduce unnecessary layers, abstractions, or patterns only to follow SOLID mechanically.

Plugins should remain stateless.

Shared business logic should be extracted into reusable classes or services when it is used by multiple plugins or when the logic becomes complex enough to justify separation.

## 7. Plugin Unit Testing

Meaningful plugin business logic should have unit tests.

The goal is to test business behaviour and important edge cases, not to chase an arbitrary code-coverage percentage.

Examples of valuable tests:

```text
Session overlaps another session
→ validation fails

Session does not overlap another session
→ validation succeeds

Invalid status transition
→ rejected

Valid status transition
→ accepted
```

Focus unit tests on:

- business rules;
- validation;
- important branching logic;
- edge cases;
- logic that could easily break during future changes.

Do not create tests only to prove that framework plumbing works.

Not every line of plugin registration or Dataverse boilerplate needs its own unit test.

## 8. Client-Side TypeScript and JavaScript

New client-side logic should prefer TypeScript.

Form scripts should remain focused on form-specific orchestration.

Avoid duplicating the same logic across multiple forms.

Preferred structure:

```text
webresources/
│
├── forms/
│   ├── event/
│   │   └── EventForm.ts
│   └── session/
│       └── SessionForm.ts
│
├── services/
│   └── DataverseService.ts
│
├── shared/
│   ├── validation.ts
│   └── formHelpers.ts
│
└── utils/
```

If multiple forms use the same validation, Dataverse operation, formatter, or helper function, move that logic into a shared module.

Form files should not become large files containing unrelated reusable logic.

Critical business rules should not be enforced only in client-side code because data can enter Dataverse without using the form.

## 9. PCF Components

PCF controls should avoid duplicating reusable React components, hooks, services, types, and utilities.

If multiple PCF controls use the same functionality, extract it into a shared project.

Example structure:

```text
pcf/
│
├── shared/
│   ├── components/
│   ├── hooks/
│   ├── services/
│   ├── types/
│   └── utils/
│
├── SessionPicker/
└── SpeakerPicker/
```

Reference implementation:

https://github.com/gjuzel99/reusable-pcf-controls-react

The goal is to keep reusable code in one place instead of copying the same implementation between controls.

Shared code should only be extracted when it is genuinely reusable.

Do not create a shared abstraction for code that is used only once.

## 10. Power Automate

Cloud flows should be used mainly for orchestration, automation, notifications, scheduled work, and appropriate asynchronous processes.

Guidelines:

- create flows inside the solution;
- use intentional connection references;
- use environment variables for environment-specific values;
- avoid duplicating the same logic across multiple flows;
- keep flows focused on one clear process;
- do not move core transactional business rules into Power Automate only because it is easy to build there.

Example:

```text
Session Approved
       ↓
Power Automate
       ↓
Send Speaker Notification
```

A flow should not automatically become the central business-logic layer of the application.

The correct implementation should depend on the requirement.

## 11. Avoid Overengineering

Maintainable does not mean complicated.

Do not introduce unnecessary abstractions, frameworks, layers, or shared libraries before the project needs them.

For example, a simple plugin does not automatically require:

```text
Domain/
Application/
Infrastructure/
Repositories/
Factories/
Builders/
Commands/
Handlers/
Adapters/
```

Create abstractions when duplication, complexity, reuse, testing, or expected change makes them useful.

The project should remain easy to understand for someone opening the repository for the first time.

## Core Engineering Rules

Community Event Hub follows these initial rules:

1. Use Power Platform capabilities before writing custom code.
2. Build inside an intentional solution and publisher.
3. Keep naming consistent.
4. Reuse standard Dataverse tables when they fit the domain.
5. Design relationships before creating them.
6. Avoid duplicate relationships and unnecessary custom tables.
7. Reuse connection references when connector, identity, and purpose are the same.
8. Keep environment-specific configuration outside custom code.
9. Keep plugin entry points small and business logic testable.
10. Test meaningful business behaviour instead of chasing full code coverage.
11. Reuse shared client-side logic instead of copying it between forms.
12. Reuse shared PCF components when multiple controls need them.
13. Use Power Automate for the processes it fits best.
14. Avoid unnecessary complexity.
15. Let the engineering structure evolve with real project requirements.
