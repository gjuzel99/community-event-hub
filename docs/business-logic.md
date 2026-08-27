# Community Event Hub Business Logic Strategy

This document defines the initial strategy for deciding where business logic should live in Community Event Hub.

The goal is to use the simplest Power Platform capability that can correctly satisfy the requirement while keeping important business rules maintainable, reusable, and consistent.

The strategy will evolve as real project requirements are implemented.

## Decision Principle

When a new requirement appears, do not start by choosing a technology.

Start by understanding the requirement.

```text
Requirement
    ↓
Can Dataverse handle it natively?
    ├─ Yes → Dataverse configuration
    └─ No
         ↓
Is it simple declarative form behaviour?
    ├─ Yes → Business Rule
    └─ No
         ↓
Is it client-side user experience logic?
    ├─ Yes → TypeScript / JavaScript
    └─ No
         ↓
Must it always be enforced server-side?
    ├─ Yes → C# Plugin
    └─ No
         ↓
Is it an explicit reusable business operation?
    ├─ Yes → Custom API
    └─ No
         ↓
Is it asynchronous, process-oriented, or a notification?
    └─ Yes → Power Automate
```

This is a guideline, not a strict algorithm.

The final decision should consider:

- data integrity;
- transaction boundaries;
- user experience;
- performance;
- reuse;
- maintainability;
- where the data can enter Dataverse;
- whether the operation must be synchronous;
- whether multiple callers need to reuse the same logic.

## 1. Dataverse Configuration

Use Dataverse capabilities before introducing custom code.

Examples include:

- required columns;
- relationships;
- Choice columns;
- formula or calculated columns;
- duplicate detection;
- auditing;
- security;
- other platform configuration.

### Example

A simple derived value should not automatically require a plug-in.

```text
Simple derived value
→ Dataverse formula or calculated capability
```

The first question should always be:

> Can Dataverse already solve this requirement clearly and maintainably?

---

## 2. Business Rules

Business Rules are useful for simple declarative validation and form behaviour.

Good scenarios include:

- making a field required;
- showing or hiding a field;
- enabling or disabling a field;
- setting a value;
- simple validation.

### Example

```text
Event End Date must not be before Event Start Date
→ Business Rule
```

Business Rules should remain the first option when the requirement is simple and easy to understand without custom code.

### Maintainability Boundary

Business Rules can become difficult to maintain when a table accumulates many interacting rules.

Problems may appear when:

- many Business Rules affect the same fields;
- multiple rules depend on each other;
- similar behaviour is repeated across multiple forms;
- it becomes difficult to understand the execution order;
- reusable behaviour must be copied between rules;
- debugging the complete form behaviour becomes difficult.

At that point, client scripting can provide a clearer structure through:

- reusable functions;
- shared services;
- explicit source control;
- clearer separation of responsibilities;
- unit testing;
- reusable validation and helper logic.

Guideline:

```text
Simple declarative behaviour
→ Business Rule

Complex, reusable, or highly coordinated client behaviour
→ Client-Scripting
```

This does not mean every Business Rule should be replaced with JavaScript.

The goal is to use the simplest approach that remains understandable as the project grows.

---

## 3. Client Scripting

Use Client Scripting when the requirement is primarily related to the Model-Driven App user experience and is too complex for a Business Rule.

Possible responsibilities include:

- complex form behaviour;
- lookup filtering;
- contextual user feedback;
- reusable form validation;
- querying related records;
- shared form services;
- command behaviour;
- dynamic user-interface interactions.

New client-side code should be authored in TypeScript and compiled or bundled into JavaScript web resources.

Preferred structure:

```text
webresources/
├── forms/
├── services/
├── shared/
└── utils/
```

Form-specific files should orchestrate form behaviour rather than contain every piece of reusable logic.

If multiple forms require the same validation, Dataverse query, formatter, or helper, that logic should be extracted and reused.

### Example

```text
Filter available records in a lookup based on related Event data
→ Client Scripting
```

Client-side logic should improve the user experience, but critical business rules should not depend only on the browser.

Data can enter Dataverse through:

- APIs;
- imports;
- Power Automate;
- integrations;
- other applications;
- future external platforms.

---

## 4. Client-Side and Server-Side Validation

Some important requirements intentionally need validation in both the client and the server.

This is not necessarily bad duplication because the two layers have different responsibilities.

### Example

```text
Session must be inside Event dates
```

Client-side validation can:

- show the problem immediately;
- provide a better user experience;
- prevent the user from reaching Save with obviously invalid data.

Server-side validation can:

- protect the rule when records are created through an API;
- protect the rule during imports;
- protect the rule when Power Automate creates or updates data;
- protect the rule for future integrations;
- guarantee that the business invariant is always enforced.

Guideline:

```text
Client-side
= immediate user feedback

Server-side
= authoritative business rule
```

The business condition may exist at both layers, but the implementation responsibilities are different.

Avoid copying large blocks of identical implementation logic between TypeScript and C#.

Keep the server-side validation authoritative and keep the client-side check focused on user experience.

---

## 5. C# Plugins

Use C# plug-ins when a business rule must execute server-side and be enforced regardless of where the data comes from.

Good scenarios include:

- important validation;
- transactional business logic;
- state-transition rules;
- preventing invalid data;
- logic that must run during Create or Update operations.

### Example: Session must be inside Event dates

```text
Event
10 September → 12 September

Session
13 September
```

The Session should not be saved.

```text
Session Create / Update
        ↓
C# Plugin
        ↓
Read related Event
        ↓
Validate Session dates
        ↓
Allow or reject operation
```

### Example: Scheduling Conflict

```text
Session A
Main Stage
10:00 → 11:00

Session B
Main Stage
10:30 → 11:30
```

If overlapping Sessions in the same Room are not allowed, this rule should be enforced server-side.

```text
Scheduling conflict validation
→ C# Plugin
```

---

## 6. Plug-in Execution Pipeline

The plug-in execution stage should be chosen based on what the logic needs to do.

### PreValidation

Use PreValidation when:

- validation should happen before the main database operation;
- the operation may need to be cancelled early;
- unnecessary transaction rollback should be avoided.

Example:

```text
Session is outside Event dates
→ reject the operation early when the required information is available
```

PreValidation is usually a good place for validation that can reject an invalid request before the main database transaction.

### PreOperation

Use PreOperation when:

- logic must run before the main database operation;
- values on the Target record need to be changed before they are saved;
- the logic must participate in the transaction.

Example:

```text
Normalize or calculate a value before Save
→ PreOperation
```

When possible, modify the Target directly instead of issuing another Update request.

This avoids unnecessary additional Dataverse operations.

### PostOperation

Use PostOperation when:

- the main Dataverse operation must already have completed;
- the created record ID is required;
- final saved values are required;
- related processing needs to happen after the main operation.

Synchronous PostOperation still participates in the Dataverse transaction.

Avoid updating the same Target record in PostOperation when the value could have been set during PreOperation.

Doing another Update can trigger another execution pipeline and introduce unnecessary complexity or recursion.

---

## 7. Pre Images and Post Images

Entity Images should be used when the plug-in needs values from before or after the Dataverse operation.

### Pre Image

Use a Pre Image when the plug-in needs previous values.

Examples:

```text
Previous Status
Previous Start Date
Previous End Date
Previous Event
```

A common scenario is detecting a real change:

```text
Pre Image Status
        ↓
Compare with Target Status
        ↓
Run logic only when Status changed
```

### Post Image

Use a Post Image when the plug-in needs the final values after the operation.

Examples:

```text
Final Status
Final saved values
Final calculated values
```

### Image Guidelines

Use images instead of retrieving the same record again only to compare before and after values when the message supports entity images.

Only include the columns the plug-in actually needs.

Good:

```text
Pre Image
- Status
- Start Date
- End Date
```

Avoid:

```text
Pre Image
- All Columns
```

This keeps the plug-in registration intentional and reduces unnecessary data.

---

## 8. Plug-in Performance

Synchronous plug-ins should be short, predictable, and focused.

The user is waiting for the Dataverse operation to finish, so synchronous logic directly affects the response time of the application.

Dataverse also has a hard execution time limit for message processing, so long-running work should not be placed inside a synchronous plug-in.

Guideline:

```text
Synchronous Plugin
→ fast validation and transactional logic

Longer or non-critical processing
→ asynchronous processing
```

Avoid:

- unnecessary Dataverse queries;
- retrieving all columns;
- repeated updates;
- long loops;
- uncontrolled external HTTP calls;
- doing work that could happen asynchronously.

External HTTP calls should use explicit short timeouts and should never be allowed to wait indefinitely.

The two-minute platform limit should be treated as a hard boundary, not as a performance target.

A synchronous plug-in should normally complete much faster.

---

## 9. Asynchronous Plugins

Use asynchronous plug-ins when the work does not need to block the original Dataverse operation.

Possible scenarios include:

- processing that can happen after the original Save;
- heavier related processing;
- work where failure should not roll back the original record operation;
- developer-centric server-side processing that does not need to be synchronous.

Guideline:

```text
Must complete before Save returns
→ synchronous

Can happen after Save
→ consider asynchronous
```

Asynchronous plug-ins and Power Automate can sometimes both solve the same problem.

The final decision should depend on the responsibility.

A useful general distinction is:

```text
Developer-centric Dataverse logic
→ Asynchronous Plugin

Process / notification / orchestration
→ Power Automate
```

This is not an absolute rule.

---

## 10. Custom APIs

Use a Custom API when Community Event Hub needs an explicit reusable business operation instead of logic that only reacts to Create, Update, or Delete events.

Examples:

```text
Submit Session
Approve Session
Generate Event Schedule
```

A Custom API can:

- accept input parameters;
- execute server-side business logic;
- return output parameters;
- provide one reusable Dataverse operation;
- be called by different clients.

Possible callers include:

- Model-Driven App client scripting;
- Power Automate;
- external applications;
- integrations;
- other Dataverse code.

### Example: Submit Session

```text
Submit Session
      ↓
Validate required information
      ↓
Change Session status
      ↓
Record submission data
      ↓
Return result
```

This creates an explicit business operation rather than spreading the same process across multiple callers.

---

## 11. Custom API as a Server-Side Integration Boundary

Custom APIs are also useful when client-side code needs to trigger communication with a third-party API.

Sensitive third-party authentication information should not be placed in browser JavaScript.

Client-side code should not contain:

- client secrets;
- API secrets;
- passwords;
- sensitive authentication material.

Instead, the client can call a Dataverse Custom API.

Concept:

```text
Model-Driven App
      ↓
TypeScript
      ↓
Dataverse Custom API
      ↓
Server-side Plugin
      ↓
Third-party API
```

Benefits:

- authentication logic remains server-side;
- sensitive credentials are not exposed to browser code;
- client-side code uses a stable Dataverse operation;
- third-party implementation details remain behind the server-side boundary;
- multiple callers can reuse the same operation.

Example:

```text
Model-Driven App
      ↓
Request external event information
      ↓
Custom API
      ↓
Server-side authentication
      ↓
Third-party service
```

A Custom API does not automatically solve secret management.

Credentials still need to be stored and accessed securely using an appropriate server-side configuration or secret-management approach.

### External API Performance

A slow or unreliable third-party call should not automatically be placed inside a synchronous Custom API.

For long-running integration work, consider:

- Power Automate;
- asynchronous plug-in processing;
- Azure services;
- webhooks;
- Azure Service Bus;
- another asynchronous integration architecture.

Use synchronous external calls only when the user truly needs the response immediately and the external service is expected to respond quickly and reliably.

---

## 12. Power Automate

Use Power Automate mainly for asynchronous, process-oriented, and integration work.

Good scenarios include:

- notifications;
- approvals;
- scheduled processes;
- asynchronous orchestration;
- external integrations;
- background processes.

### Example: Speaker Acceptance Notification

```text
Session becomes Accepted
        ↓
Power Automate
        ↓
Send notification to speaker
```

The core business state should remain in Dataverse.

Power Automate handles the asynchronous reaction to that state.

Do not use Power Automate as the default location for every business rule.

Critical transactional business invariants should normally be enforced closer to Dataverse.

---

## Business Rule vs Client Scripting

Use a Business Rule when the requirement is simple and declarative.

```text
Simple show/hide
Simple required field
Simple validation
→ Business Rule
```

Use Client Scripting when the client-side requirement becomes more complex or needs reuse.

```text
Complex lookup filtering
Reusable form services
Related-table queries
Dynamic UI behaviour
Complex command logic
Shared validation
→ Client Scripting
```

The goal is not to prefer code automatically.

The goal is to keep the client-side implementation understandable as complexity grows.

---

## Initial Requirement Matrix

| Requirement | Preferred Technology | Reason |
|---|---|---|
| Show Online Meeting URL for online events | Business Rule | Simple form behaviour |
| Complex reusable form behaviour | Client Scripting | Maintainable client-side UX |
| Event End Date must be after Start Date | Client Scripting + C# Plugin | Simple declarative validation |
| Session must be inside Event dates | Client Scripting + C# Plugin | Immediate feedback + authoritative validation |
| Prevent scheduling conflicts | Client Scripting + C# Plugin | UX feedback + server-side invariant |
| Simple derived value | Dataverse configuration | Platform capability |
| Send speaker acceptance notification | Power Automate | Asynchronous process |
| Submit Session operation | Custom API | Explicit reusable business operation |
| Call a third-party API from the Model-Driven App | Custom API + Server-side Plugin | Keeps sensitive integration logic off the client |

This matrix should evolve as Community Event Hub introduces real requirements.

---

## Core Rules

1. Understand the requirement before choosing the technology.
2. Use Dataverse capabilities before writing custom code.
3. Keep Business Rules for simple declarative behaviour.
4. Move complex or reusable client-side behaviour into Client Scripting.
5. Use client-side validation for immediate user feedback.
6. Keep important business invariants authoritative on the server.
7. Accept intentional client/server validation when both user experience and data integrity require it.
8. Choose the correct plug-in execution stage for the responsibility.
9. Use PreValidation for early validation when appropriate.
10. Use PreOperation when Target values need to be changed before Save.
11. Use PostOperation only when the completed operation or final data is required.
12. Use Pre Images and Post Images instead of unnecessary retrieves when appropriate.
13. Request only the image columns the plug-in actually needs.
14. Keep synchronous plug-ins fast and predictable.
15. Use asynchronous processing when work does not need to block the user.
16. Use Custom APIs for explicit reusable business operations.
17. Keep third-party credentials and sensitive authentication logic out of browser code.
18. Do not force slow third-party operations through synchronous Custom APIs.
19. Use Power Automate for appropriate asynchronous process, notification, and orchestration work.
20. Avoid unnecessary duplication while allowing deliberate UX + server validation where required.
21. Let the implementation follow the requirement rather than the developer's preferred technology.
