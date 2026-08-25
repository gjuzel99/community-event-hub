# Community Event Hub

Community Event Hub is an open-source Power Platform project for exploring how to build maintainable and scalable event-management solutions with Microsoft Dataverse and Power Platform.

The project is being built publicly through a series of live streams.

## Current Status

🚧 Business logic architecture

Episode 1 established the initial project foundation:

- project scope and non-goals;
- initial architecture;
- engineering guidelines;
- project roadmap.

Episode 2 created the first Dataverse foundation:

- Community Event Hub solution and publisher;
- initial solution and repository structure;
- reuse of standard Dataverse tables such as Account and Contact;
- reuse of Activities such as Appointments, Emails, Tasks and Phone Calls;
- Connections and Connection Roles for flexible relationships;
- initial Event and Session custom tables;
- first version of the Community Event Hub data model;
- unpacked solution source stored in GitHub.

Episode 3 will focus on deciding where business logic should live.

We will compare:

- Dataverse configuration;
- Business Rules;
- TypeScript and JavaScript;
- C# plugins;
- Custom APIs;
- Power Automate.

The goal is to create a practical business-logic strategy before implementing more custom logic.

## First Episodes

1. Project vision, architecture, scope, and engineering principles
2. Designing a maintainable Dataverse foundation
3. Plugin, JavaScript or Power Automate? Designing the business logic architecture

More episodes will follow as Community Event Hub grows.

## Scope

Community Event Hub will explore event-management scenarios such as:

- events and venues;
- speakers and community contacts;
- session submissions;
- session reviews and approvals;
- rooms and scheduling;
- notifications and feedback.

The project will grow incrementally as the live series continues.

## Non-Goals

The initial project will not:

- replace platforms such as Sessionize or run.events;
- include ticketing or payment functionality;
- build a complete public attendee portal;
- become a complex multi-tenant SaaS platform;
- introduce custom code when standard Power Platform capabilities are enough.

## Live Series

### Episode 1

**How to Start a Maintainable Power Platform Project | Community Event Hub #1**

📅 13 August 2026  
🔴 YouTube: https://youtube.com/live/1Hh4_qY7PuM

### Episode 2

**Designing a Maintainable Dataverse Foundation | Community Event Hub #2**

📅 20 August 2026  
🔴 YouTube: https://www.youtube.com/live/H071iJ2TZLs

### Episode 3

**Plugin, JavaScript or Power Automate? Designing Business Logic That Scales | Community Event Hub #3**

📅 27 August 2026  
🔴 YouTube: https://www.youtube.com/live/U3nxJQlnH8A

## Project Documentation

- [Architecture](docs/architecture.md)
- [Engineering](docs/engineering.md)
- [Data Model](docs/data-model.md)
- [Business Logic](docs/business-logic.md)
- [Roadmap](ROADMAP.md)

## License

MIT
