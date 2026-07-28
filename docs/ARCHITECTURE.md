# Eventra Architecture

## Component View

```mermaid
flowchart TB
    subgraph Clients
        Operator["Operational user"]
        Administrator["Administrator"]
    end

    subgraph Application["Next.js application"]
        Pages["Responsive pages and calendar"]
        Routes["API routes"]
        Access["Authentication and RBAC"]
        Services["Domain services"]
        Exports["PDF and Excel generation"]
        Adapters["Communication adapters"]
    end

    subgraph Data
        ORM["Prisma ORM"]
        MySQL[("MySQL")]
        Backups["Backup / restore scripts"]
    end

    Operator --> Pages
    Administrator --> Pages
    Pages --> Routes
    Routes --> Access
    Routes --> Services
    Services --> ORM
    ORM --> MySQL
    Services --> Exports
    Services --> Adapters
    MySQL --> Backups
```

## Main Data Domains

| Domain | Representative responsibilities |
|---|---|
| Reservations | Events, participants, dynamic values, conflict checks |
| Customers | Active and prospective customer records |
| Finance | Payments, cash boxes, transactions, pricing, tax |
| Organization | Offices, branches, venues, packages, products |
| Access | Users, roles, permissions |
| Documents | Contract and receipt templates |
| Configuration | Form fields, visibility, calendar, theme, communication |
| Reporting | Revenue, reservation, customer, and date-based summaries |

## Dynamic Reservation Form Flow

```mermaid
sequenceDiagram
    participant A as Administrator
    participant C as Configuration
    participant O as Operator
    participant F as Reservation form
    participant R as Reservation API
    participant D as MySQL

    A->>C: Configure fields, visibility and options
    C->>D: Persist field definitions
    O->>F: Open reservation form
    F->>R: Request active configuration
    R->>D: Load definitions and business rules
    D-->>R: Current form schema
    R-->>F: Renderable field configuration
    O->>F: Enter reservation data
    F->>R: Submit fixed and dynamic values
    R->>R: Validate conflicts and configured rules
    R->>D: Save reservation and dynamic values
```

## Design Decisions

### Configuration separated from submitted values

Field definitions and reservation values are modeled separately. This keeps
historical records usable while allowing the active form configuration to
change.

### Business modules share one operational database

Reservations, finance, reporting, and customer operations use the same source
of truth. Cross-module actions are validated in the backend rather than being
coordinated only in the interface.

### Internal-first security model

The application is designed as a protected internal panel with authenticated
access and role-based permissions. Public indexing is intentionally disabled.

### Payment tracking is not payment processing

The finance module records and reports operational collections. It does not
represent an online card-processing or virtual-POS integration.
