# AMAALPROP

## Product Requirements Document (PRD)

**Document status:** Product definition baseline\
**Version:** 1.0\
**Date:** 13 November 2026\
**Product type:** Multi-organisation property management SaaS\
**Initial market consideration:** Uganda, with an architecture designed
for international expansion\
**Document purpose:** Define the product, users, workflows, data-entry
fields, business rules, functional requirements, non-functional
requirements and acceptance criteria that will guide UX design, database
modelling, API design, implementation and testing.

------------------------------------------------------------------------

# 1. Executive Summary

AMAALPROP is a multi-organisation property management platform for
landlords and property management organisations. It brings property
records, buildings, floors, rentable units, tenants, leases, rent
billing, payment verification, double-entry accounting, maintenance,
inspections, staff assignments, approvals, communications and reporting
into one controlled system.

A landlord uses AMAALPROP to manage one or many properties. A property
may be a standalone house, a shop, an apartment block, a shopping mall,
a commercial building, a mixed-use development or a multi-building
estate. Properties can contain buildings, buildings can contain floors
or levels, and floors can contain rentable units. The model must also
support standalone units without requiring artificial building or floor
records.

AMAALPROP serves two different meanings of tenancy:

1.  **SaaS organisation tenancy:** an independent landlord organisation
    uses the platform and must not see another organisation's data.
2.  **Real-estate tenancy:** a person, household or business rents a
    unit or multiple units from a landlord.

These concepts must never be confused in the data model or interface.

The platform supports organisation-specific staff structures,
configurable roles, permissions, reporting relationships and
property-level assignments. Tenants receive individual portal access
after invitation, contact verification and approval by the landlord or
an authorised manager.

The payment model does not require each landlord to connect a separate
banking or mobile-money API. The landlord configures payment
instructions in AMAALPROP. Tenants pay externally and submit transaction
details. Authorised staff verify actual receipt before the payment is
posted to the accounting ledger and allocated to charges.

AMAALPROP must provide a full double-entry accounting ledger. Every
posted journal entry must balance. Financial records must be auditable,
and posted transactions must not be silently edited or deleted.

Tenant move-out is a controlled workflow. The tenant submits a move-out
request, a caretaker conducts a documented inspection, a property
manager verifies the inspection, an accountant or authorised financial
officer prepares the financial settlement, and the landlord or delegated
authority gives final approval. The platform preserves the distinction
between financial settlement, physical condition and lawful lease
termination.

The product should begin as a modular monolith with strong domain
boundaries. This provides a maintainable foundation without prematurely
introducing distributed microservices.

------------------------------------------------------------------------

# 2. Product Definition

## 2.1 What AMAALPROP is

AMAALPROP is a property operations and financial management system that
gives a landlord or property management organisation a structured view
of:

-   What properties the organisation controls.
-   How each property is physically organised.
-   Which units are available, occupied, reserved, under maintenance or
    otherwise unavailable.
-   Who occupies each unit and under what agreement.
-   What rent and other charges are due.
-   Which tenant payment submissions have been verified.
-   What money has been received, allocated, refunded or reversed.
-   Which employees are responsible for each property or operational
    task.
-   What maintenance, inspection and clearance work is outstanding.
-   Which decisions require approval.
-   How each property and the organisation are performing financially
    and operationally.

AMAALPROP is not merely a tenant register, rent reminder application or
payment-reference collection tool. It is intended to provide a
controlled operating record for a property organisation.

## 2.2 Product vision

Provide landlords and property management organisations with one
reliable system for managing property assets, tenancy relationships,
financial activity and day-to-day operations, while giving each
authorised user an interface appropriate to their responsibilities.

## 2.3 Product objectives

1.  Centralise property, building, floor and unit information.
2.  Maintain reliable tenant and lease records.
3.  Support single-tenant, co-tenant, company-tenant and multiple-unit
    lease arrangements.
4.  Make rent billing and payment verification traceable.
5.  Provide complete double-entry accounting.
6.  Define clear staff responsibilities and scoped access.
7.  Standardise maintenance, inspection and move-out workflows.
8.  Give tenants a secure portal for relevant tenancy activities.
9.  Provide accurate property, portfolio and organisation reporting.
10. Support multiple independent landlord organisations on shared SaaS
    infrastructure without cross-organisation data exposure.
11. Support different countries, currencies, time zones and
    identification systems over time.

## 2.4 Product principles

-   **Organisation isolation:** one landlord organisation cannot access
    another organisation's private records.
-   **One organisation per email identity:** one email address may
    belong to only one property organisation under the confirmed product
    rule.
-   **Configurable operations:** landlords can configure property
    structures, occupancy limits, roles, payment policies, approval
    rules and inspection checklists.
-   **Traceability:** important actions record who did what, when and
    within which organisation.
-   **Financial integrity:** posted accounting entries are balanced and
    immutable.
-   **Least privilege:** users receive only the access required for
    their responsibilities.
-   **Separation of concerns:** tenant profile, login account, lease,
    unit occupancy, payment submission and accounting transaction are
    separate concepts.
-   **Real-world flexibility:** custom unit labels and property
    structures must not be constrained by assumptions about numbering or
    building design.
-   **Mobile usability:** core operational workflows must work on phones
    as well as desktop screens.
-   **International readiness:** country-specific settings must not be
    hard-coded into core entities.

------------------------------------------------------------------------

# 3. Confirmed Product Decisions

The following decisions are confirmed and form the baseline for this
PRD.

  -----------------------------------------------------------------------
  Area                                Confirmed decision
  ----------------------------------- -----------------------------------
  SaaS model                          Multiple independent
                                      landlord/property organisations use
                                      one platform with strict data
                                      isolation

  Property ownership                  Each property record belongs to one
                                      landlord organisation

  Property hierarchy                  A property may contain buildings,
                                      floors and units, or standalone
                                      units

  Unit labels                         Landlords can use custom display
                                      codes such as MN20, PA1 or PN400

  Tenant occupancy                    A unit/lease can be configured for
                                      one tenant or multiple co-tenants,
                                      with configurable limits

  Multiple units                      A tenant may rent more than two
                                      units in one building; one lease
                                      may cover multiple units

  Tenant identity                     Tenant profile and portal login
                                      account are separate records

  Tenant activation                   Landlord or authorised manager
                                      approves activation after
                                      invitation and contact verification

  Payment instructions                Landlord configures
                                      bank/mobile-money payment
                                      instructions; tenants pay
                                      externally

  Payment verification                Tenant submits reference and
                                      payment details; authorised staff
                                      verify actual receipt

  Accounting                          Full double-entry ledger is
                                      required

  Staff structure                     Customisable roles, permissions,
                                      assignments and reporting
                                      relationships

  Staff scope                         Staff access can be restricted by
                                      organisation, region, portfolio,
                                      property, building or other defined
                                      scope

  Tenant clearance                    Caretaker inspects, property
                                      manager verifies, landlord or
                                      delegated authority gives final
                                      approval

  Payment status                      Green, orange and red
                                      financial-health indicators are
                                      configurable

  Clearance status                    Physical condition and financial
                                      health are separate statuses

  Architecture                        Modular monolith initially, with
                                      clear domain boundaries
  -----------------------------------------------------------------------

## 3.1 One email address per organisation

An email address must not be registered as a member of more than one
property organisation. This rule must be enforced in registration,
invitation, identity management and database constraints.

A user working for two separate landlord organisations must use separate
email identities under this product rule. Organisation transfers or
changes of employment must use an explicit controlled process; the
system must not silently move historical activity from one organisation
to another.

This is a deliberate product restriction and must be explained in
onboarding and validation messages.

## 3.2 Property ownership scope

A property belongs to one organisation in AMAALPROP. The initial product
does not model several landlord organisations holding fractional
ownership shares in the same property record.

An organisation may record external ownership or management
documentation where required, but this does not create
cross-organisation ownership access.

------------------------------------------------------------------------

# 4. Scope

## 4.1 In scope

-   Organisation registration and configuration.
-   User identity, invitations and activation.
-   Roles, permissions, assignments and reporting relationships.
-   Properties, buildings, floors, units and unit categories.
-   Standalone units.
-   Residential, commercial and mixed-use properties.
-   Tenant profiles and tenant portal accounts.
-   Single and multiple tenant arrangements.
-   Multiple units under one lease.
-   Lease creation, renewal, variation and termination.
-   Configurable payment instructions.
-   Rent and other charge billing.
-   Payment submissions and staff verification.
-   Full double-entry accounting.
-   Receipts, statements, deposits, refunds, expenses and
    reconciliation.
-   Green/orange/red financial health rules.
-   Maintenance and work orders.
-   Property inspections and tenant move-out clearance.
-   Approval workflows and delegated authority.
-   Notifications and communication history.
-   Operational and financial reporting.
-   Audit logs and document management.
-   Internationalisation foundations.

## 4.2 Out of scope for the first release unless separately approved

-   Mandatory direct bank API integration for every landlord.
-   Mandatory mobile-money API integration for every landlord.
-   Fractional ownership of a property by multiple AMAALPROP
    organisations.
-   A public property marketplace.
-   Automated tenant credit scoring.
-   Automated legal interpretation of leases.
-   Automatic legal enforcement or eviction decisions.
-   Full payroll and human-resource management.
-   Full contractor marketplace.
-   AI-based property valuation as a source of official accounting
    values.
-   Autonomous financial verification based only on uploaded
    screenshots.
-   Microservices as an initial deployment requirement.

These may be considered as future extensions.

------------------------------------------------------------------------

# 5. Users, Roles and Portals

AMAALPROP must provide role-appropriate experiences. A portal is a
tailored interface and workflow set, not simply a shared dashboard with
buttons hidden.

## 5.1 User categories

### A. Landlord / Organisation Owner

Responsible for the organisation and its property portfolio.

Typical capabilities:

-   Configure organisation settings.
-   Register and manage properties.
-   Invite and activate employees.
-   Define roles and permissions.
-   Assign employees to scopes.
-   Configure payment instructions.
-   Configure rent policies and approval thresholds.
-   Review organisation-wide financial and operational reports.
-   Approve high-authority actions.
-   Review audit records.
-   Suspend access and revoke assignments.

### B. Director / General Manager

A senior manager with authority delegated by the landlord.

Possible capabilities:

-   View organisation-wide operations.
-   Supervise managers.
-   Approve transactions or operational requests within assigned
    authority.
-   Review property and staff performance.
-   Escalate matters to the landlord.

The landlord controls whether this role exists and what authority it
receives.

### C. Regional Manager

Oversees properties within one or more assigned geographical regions.

Possible capabilities:

-   View assigned regional portfolio.
-   Supervise property managers.
-   Review regional occupancy and arrears.
-   Approve defined operational requests.
-   Escalate matters.

### D. Portfolio Manager

Oversees a defined portfolio of properties.

### E. Property Manager

Manages assigned properties.

Possible capabilities:

-   Manage tenant and lease operations.
-   Review payment submissions if authorised.
-   Verify caretaker inspection reports.
-   Manage maintenance requests.
-   Prepare lease renewals.
-   Submit recommendations for final approvals.

### F. Site Manager

Manages an assigned building or site and coordinates day-to-day
activities.

### G. Caretaker

Handles assigned physical property tasks.

Possible capabilities:

-   View assigned units and relevant tenant information.
-   Record inspections.
-   Upload condition evidence.
-   Report maintenance issues.
-   Record meter readings and key handovers.
-   Submit work completion evidence.

Caretakers should not automatically receive access to full accounting
records or sensitive tenant identity documents.

### H. Accountant / Finance Officer

Handles financial operations.

Possible capabilities:

-   Review payment submissions.
-   Verify actual receipts against available financial evidence.
-   Create or review invoices according to permission.
-   Allocate verified payments.
-   Issue receipts.
-   Record expenses.
-   Prepare deposit settlements.
-   Reconcile bank transactions.
-   Review ledger and financial reports.

### I. Leasing Officer

Handles tenant onboarding and lease administration.

### J. Tenant

A tenant can:

-   View their authorised profile and lease information.
-   View applicable payment instructions.
-   Submit payment references.
-   View verified receipts and statements.
-   Submit maintenance requests.
-   Receive notices.
-   Submit move-out requests.
-   View relevant inspection and clearance updates.

A tenant must not access another tenant's records, even where they share
a unit, except for information explicitly shared by policy or agreement.

### K. Occupant / Representative

A person recorded as an authorised occupant or company representative
may receive limited portal access if the landlord enables it. This
access is not automatically equivalent to legal tenant status.

## 5.2 Roles are configurable

The roles above are suggested defaults, not a mandatory fixed hierarchy.
An organisation may create custom roles, rename roles and define its own
reporting structure.

## 5.3 Roles, permissions and scope

These are distinct:

-   **Role:** named responsibility.
-   **Permission:** an action, such as `payment.verify` or
    `lease.update`.
-   **Assignment:** the organisational or property scope where the
    permission applies.
-   **Reporting relationship:** who supervises or receives escalations
    from whom.
-   **Approval authority:** what decisions and limits a user can
    approve.

A property manager may have `lease.manage` for Property A but not
Property B. A finance officer may have `payment.verify` for several
assigned properties but no authority to change lease terms.

------------------------------------------------------------------------

# 6. Organisation Onboarding and Account Management

## 6.1 Organisation registration

### Data-entry fields

  -----------------------------------------------------------------------------------------
  Field          Type                   Required Entered by    Rules / purpose
  -------------- ------------- ----------------- ------------- ----------------------------
  Organisation   Text                        Yes Registrant    Registered or legal
  legal name                                                   organisation name

  Trading name   Text                         No Registrant    Name shown to tenants if
                                                               different

  Organisation   Select                      Yes Registrant    Individual landlord,
  type                                                         property company, property
                                                               manager or other configured
                                                               type

  Registration   Text                Conditional Registrant    Required where applicable
  number                                                       

  Country        Country                     Yes Registrant    Drives country-specific
                 selector                                      settings

  Region / state Select or           Conditional Registrant    Country-dependent
                 text                                          

  District /     Text or                     Yes Registrant    Main operating location
  city           selector                                      

  Physical       Multiline                   Yes Registrant    Organisation contact address
  address        text                                          

  Postal address Multiline                    No Registrant    Optional
                 text                                          

  Official email Email                       Yes Registrant    Must satisfy
                                                               one-email-per-organisation
                                                               rule

  Official phone Phone                       Yes Registrant    Include country calling code

  Website        URL                          No Registrant    Optional

  Logo           File                         No Registrant    Supported image format and
                                                               size limits

  Default        Currency                    Yes Registrant    Initial accounting currency;
  currency       selector                                      changes require controlled
                                                               handling

  Time zone      Time-zone                   Yes Registrant    Used for dates, schedules
                 selector                                      and audit timestamps

  Date format    Select                      Yes Registrant    Display preference

  Financial year Month/day                   Yes Registrant    Used for reporting periods
  start                                                        

  Primary        Text                        Yes Registrant    Organisation contact
  contact name                                                 

  Primary        Text                         No Registrant    Optional
  contact role                                                 

  Terms          Checkbox                    Yes Registrant    Record version and timestamp
  acceptance                                                   
  -----------------------------------------------------------------------------------------

### Registration flow

1.  User opens organisation registration.
2.  User enters organisation details.
3.  System validates required fields and email uniqueness.
4.  System creates an organisation in pending or onboarding state.
5.  Registrant verifies the email address.
6.  Registrant establishes secure credentials.
7.  Organisation owner completes any required verification.
8.  Organisation becomes active according to platform policy.
9.  Owner is assigned the organisation-owner role.
10. Owner enters the organisation setup workflow.

## 6.2 Organisation settings

Organisation settings should include:

-   Display name and branding.
-   Country, currency and time zone.
-   Financial year.
-   Default rent policies.
-   Default payment instructions.
-   Default inspection templates.
-   Default approval policies.
-   Notification preferences.
-   Document retention settings.
-   User invitation rules.
-   Accounting configuration.
-   Supported languages.
-   Tenant portal branding where available.

Changes to accounting currency, financial year or core accounting
configuration must be controlled and audited.

------------------------------------------------------------------------

# 7. Identity, Invitations and Access Control

## 7.1 User registration fields

  -------------------------------------------------------------------------------------
  Field         Type                   Required Entered by     Rules
  ------------- ------------- ----------------- -------------- ------------------------
  First name    Text                        Yes User / inviter Person's given name

  Middle name   Text                         No User           Optional

  Last name     Text                        Yes User / inviter Family name

  Email         Email                       Yes User / inviter Globally checked against
                                                               one-organisation rule

  Phone         Phone               Conditional User           Country code and
                                                               verification

  Password      Password                    Yes User           Must meet password
                                                               policy

  Preferred     Select                       No User           Organisation-supported
  language                                                     language

  Profile image File                         No User           Optional

  Account       System enum                 Yes System/admin   Invited, pending
  status                                                       verification, active,
                                                               suspended, revoked

  Created at    System                      Yes System         Immutable
                timestamp                                      

  Last login    System                       No System         Updated by
                timestamp                                      authentication process
  -------------------------------------------------------------------------------------

Passwords must never be stored in plain text. Store an appropriately
salted password hash using a modern password-hashing algorithm.

## 7.2 Employee invitation

### Invitation fields

  Field                Type                    Required Entered by
  -------------------- --------------- ---------------- ------------
  Invitee full name    Text                         Yes Inviter
  Invitee email        Email                        Yes Inviter
  Invitee phone        Phone                         No Inviter
  Proposed role        Role selector                Yes Inviter
  Initial scope type   Select                       Yes Inviter
  Initial scope        Search/select        Conditional Inviter
  Reporting manager    User selector                 No Inviter
  Start date           Date                          No Inviter
  Invitation expiry    Date/time         System/default System
  Invitation note      Text                          No Inviter

### Employee activation flow

1.  Landlord or authorised administrator selects Invite Employee.
2.  System validates that the email is not attached to another property
    organisation.
3.  Inviter selects role and initial scope.
4.  System creates an invitation with a secure, single-use token.
5.  Invitation is sent by email and optionally SMS.
6.  Employee opens invitation.
7.  Employee verifies contact and creates credentials.
8.  System validates invitation expiry and organisation association.
9.  Account becomes active after any required administrator approval.
10. User receives only permissions within the assigned role and scope.
11. The action is recorded in the audit log.

## 7.3 Employee assignment fields

  --------------------------------------------------------------------------
  Field            Type                          Required Purpose
  ---------------- ---------------- --------------------- ------------------
  User ID          Foreign key                        Yes Employee account

  Organisation ID  Foreign key                        Yes Organisation
                                                          membership

  Role ID          Foreign key                        Yes Assigned role

  Scope type       Enum                               Yes Organisation,
                                                          region, portfolio,
                                                          property,
                                                          building, floor or
                                                          unit

  Scope ID         Foreign                    Conditional Specific assigned
                   key/reference                          scope

  Valid from       Date/time                          Yes Assignment start

  Valid until      Date/time                           No Optional end

  Assignment       Enum                               Yes Pending, active,
  status                                                  suspended, ended

  Assigned by      User reference                     Yes Authorising user

  Approval         Foreign key                         No Related approval
  reference                                               

  Created at       Timestamp                          Yes System-generated
  --------------------------------------------------------------------------

Assignments must be independent of property records. Reassigning a
manager must not require recreating a building, tenant, lease or unit.

## 7.4 Reporting relationships

Fields:

-   Organisation ID.
-   Supervisor user ID.
-   Subordinate user ID.
-   Relationship type.
-   Effective start date.
-   Effective end date.
-   Status.
-   Created by.
-   Change reason.

The system must prevent circular reporting relationships and invalid
self-reporting. A user may have more than one reporting relationship
only where the organisation's policy allows it.

## 7.5 Permission model

Use role-based access control combined with scope-aware checks.

Examples of permission identifiers:

-   `organisation.view`
-   `organisation.update`
-   `employee.invite`
-   `employee.activate`
-   `employee.suspend`
-   `role.manage`
-   `assignment.manage`
-   `property.create`
-   `property.update`
-   `unit.manage`
-   `tenant.view`
-   `tenant.create`
-   `tenant.activate`
-   `lease.create`
-   `lease.update`
-   `payment_instruction.manage`
-   `payment_submission.view`
-   `payment.verify`
-   `payment.reject`
-   `payment.allocate`
-   `journal.view`
-   `journal.post`
-   `journal.reverse`
-   `expense.create`
-   `expense.approve`
-   `inspection.create`
-   `inspection.verify`
-   `clearance.final_approve`
-   `report.financial.view`
-   `audit.view`

Every protected API operation must enforce authentication, organisation
membership, permission and scope. Hiding a button in the user interface
is not sufficient security.

------------------------------------------------------------------------

# 8. Property, Building, Floor and Unit Management

## 8.1 Property classifications

A property must support:

-   Residential.
-   Commercial.
-   Mixed-use.

A property may be a single standalone rentable premise or a complex
structure with buildings, floors and many units.

## 8.2 Property registration fields

  --------------------------------------------------------------------------------------------
  Field           Type                   Required Entered by            Rules / purpose
  --------------- ------------- ----------------- --------------------- ----------------------
  Property ID     System UUID                 Yes System                Internal identifier

  Organisation ID Foreign key                 Yes System                Owning organisation

  Property name   Text                        Yes Landlord/authorised   Human-readable name
                                                  staff                 

  Property code   Text                        Yes Landlord/authorised   Organisation-defined
                                                  staff                 reference

  Property type   Select                      Yes User                  Residential,
                                                                        commercial, mixed-use

  Property        Select                      Yes User                  Standalone, single
  structure                                                             building,
                                                                        multi-building,
                                                                        complex

  Description     Multiline                    No User                  Additional information
                  text                                                  

  Country         Country                     Yes User                  Property location
                  selector                                              

  Region / state  Text/select         Conditional User                  Country-dependent

  District / city Text                        Yes User                  Location

  Area /          Text                         No User                  Local area
  neighbourhood                                                         

  Street / road   Text                         No User                  Street

  Plot / parcel   Text                         No User                  Land reference
  number                                                                

  GPS latitude    Decimal                      No User/map              Valid geographic
                                                                        coordinate

  GPS longitude   Decimal                      No User/map              Valid geographic
                                                                        coordinate

  Postal address  Text                         No User                  Optional

  Acquisition /   Text                         No User                  Optional documentation
  registration                                                          reference
  reference                                                             

  Property status Select                      Yes User/system           Active, inactive,
                                                                        archived, under
                                                                        development

  Main image      File                         No User                  Optional

  Documents       File                         No User                  Ownership, plans,
                  collection                                            permits or other
                                                                        documents

  Created by      User                        Yes System                Audit
                  reference                                             

  Created at      Timestamp                   Yes System                Audit
  --------------------------------------------------------------------------------------------

Property code uniqueness should be enforced within the organisation
unless a more specific policy is configured.

## 8.3 Building fields

  Field              Type                  Required Entered by
  ------------------ ------------------- ---------- -------------
  Building ID        System UUID                Yes System
  Property ID        Foreign key                Yes User/system
  Building name      Text                       Yes User
  Building code      Text                       Yes User
  Building type      Select                      No User
  Number of levels   Integer                     No User
  Description        Text                        No User
  Building status    Select                     Yes User
  Main image         File                        No User
  Created by / at    System references          Yes System

A property may contain multiple buildings. A building must belong to one
property.

## 8.4 Floor / level fields

  Field                Type            Required Entered by
  -------------------- ------------- ---------- -------------
  Floor ID             System UUID          Yes System
  Building ID          Foreign key          Yes User/system
  Floor display name   Text                 Yes User
  Floor code           Text                 Yes User
  Floor sequence       Integer              Yes User/system
  Floor category       Select                No User
  Description          Text                  No User
  Status               Select               Yes User

Floor categories should support basement, ground, mezzanine, upper floor
and custom labels. The system must not assume that every building uses
ordinary numeric floors. Examples include Basement, Ground, PZ, PN and
other landlord-defined labels.

## 8.5 Unit fields

  --------------------------------------------------------------------------------------------
  Field          Type                           Required Entered by    Rules / purpose
  -------------- --------------------- ----------------- ------------- -----------------------
  Unit ID        System UUID                         Yes System        Stable internal
                                                                       identity

  Organisation   Foreign key                         Yes System        Tenant isolation
  ID                                                                   

  Property ID    Foreign key                         Yes System        Parent property

  Building ID    Foreign key                          No User/system   Null for standalone
                                                                       units

  Floor ID       Foreign key                          No User/system   Null where no floor
                                                                       applies

  Unit display   Text                                Yes User          E.g. MN20, PA1, Shop 07
  code                                                                 

  Unit name      Text                                 No User          Optional friendly name

  Unit category  Select                              Yes User          Residential,
                                                                       commercial, other

  Unit subtype   Select/configurable                 Yes User          Apartment, shop,
                                                                       office, kiosk,
                                                                       warehouse, house, etc.

  Description    Multiline text                       No User          Details

  Bedrooms       Integer                     Conditional User          Residential only

  Bathrooms      Decimal/integer             Conditional User          Residential only

  Kitchens       Integer                              No User          Residential only

  Area           Decimal                              No User          Numeric area

  Area unit      Select                      Conditional User          Square metres, square
                                                                       feet or other

  Maximum        Integer                              No User          Physical/contractual
  occupants                                                            occupancy limit

  Permitted      Integer                              No User          Configurable co-tenant
  legal tenants                                                        limit

  Facilities     Multi-select                         No User          Parking, lift, water,
                                                                       power, security, etc.

  Furnishing     Select                               No User          Furnished,
  status                                                               semi-furnished,
                                                                       unfurnished, not
                                                                       applicable

  Default rent   Money                                No User          Suggested rent, not a
                                                                       posted charge

  Currency       Currency                    Conditional User          Defaults from
                                                                       property/organisation
                                                                       policy

  Unit status    Select                              Yes User/system   Available, reserved,
                                                                       occupied, maintenance,
                                                                       unavailable, archived

  Availability   Date                                 No User/system   When unit can be
  date                                                                 offered

  Images         File collection                      No User          Unit images

  Documents      File collection                      No User          Plans, inventory or
                                                                       other documents

  Created by /   System references                   Yes System        Audit
  at                                                                   
  --------------------------------------------------------------------------------------------

### Unit identifier rules

The system-generated Unit ID is the true internal identifier. The
user-defined display code is a label.

Display codes must be unique within a configured scope, normally a
building or property. Changing a display code must not break leases,
payments, inspections or historical reports.

### Standalone units

A standalone house or shop must be creatable directly under a property
without requiring a building or floor. The database must permit null
building and floor references where the property structure allows it.

## 8.6 Unit configuration and occupancy policy

The landlord should configure occupancy policy at unit or lease level.

Fields:

-   Permitted tenancy arrangement: single tenant, multiple co-tenants,
    company tenant, configurable.
-   Maximum legal tenants.
-   Maximum authorised occupants.
-   Whether occupants require approval.
-   Whether each legal tenant needs a portal account.
-   Whether one primary tenant must be designated.
-   Whether joint liability is allowed.
-   Whether rent shares can be assigned.
-   Whether all co-tenants may submit maintenance requests.
-   Whether all co-tenants may view payment and lease information.
-   Whether notices go to all co-tenants or selected contacts.

A lease-specific configuration may override a unit default, provided it
does not violate applicable property or legal constraints.

------------------------------------------------------------------------

# 9. Tenant Management

## 9.1 Tenant profile and portal account are separate

A tenant profile is the real-world person or organisation associated
with a tenancy. A portal account is an authentication identity.

A tenant may exist in AMAALPROP before portal activation. A portal
account must be explicitly linked to the correct tenant profile and
organisation.

## 9.2 Tenant type

Support:

-   Individual.
-   Joint/group tenancy participant.
-   Company or other legal entity.
-   Authorised occupant.
-   Company representative.

The exact available types may depend on country and organisation
settings.

## 9.3 Individual tenant fields

  ---------------------------------------------------------------------------------------------
  Field            Type                           Required Entered by     Rules
  ---------------- --------------------- ----------------- -------------- ---------------------
  Tenant ID        System UUID                         Yes System         Internal identity

  Organisation ID  Foreign key                         Yes System         Tenant data owner

  Tenant reference Text                                Yes System/user    Organisation-unique
                                                                          reference

  Tenant type      Select                              Yes Staff          Individual, company,
                                                                          occupant,
                                                                          representative

  First name       Text                        Conditional Staff/tenant   Required for
                                                                          individual

  Middle name      Text                                 No Staff/tenant   Optional

  Last name        Text                        Conditional Staff/tenant   Required for
                                                                          individual

  Date of birth    Date                     No/conditional Staff/tenant   Collect only where
                                                                          needed

  Gender           Select                               No Staff/tenant   Optional and subject
                                                                          to lawful need

  Email            Email                                No Staff/tenant   Contact address;
                                                                          uniqueness policy
                                                                          applies to portal
                                                                          identity

  Phone            Phone                               Yes Staff/tenant   Primary contact

  Alternative      Phone                                No Staff/tenant   Optional
  phone                                                                   

  Profile          File                                 No Staff/tenant   Optional
  photograph                                                              

  Nationality      Country selector                     No Staff/tenant   Optional where needed

  Identification   Select/configurable         Conditional Staff/tenant   NIN, passport,
  type                                                                    driving licence or
                                                                          country-specific type

  Identification   Encrypted text              Conditional Staff/tenant   Restrict access
  number                                                                  

  Identification   Date                                 No Staff/tenant   Where relevant
  expiry                                                                  

  Address          Multiline text                       No Staff/tenant   Contact address

  Emergency        Text                                 No Staff/tenant   Optional
  contact name                                                            

  Emergency        Phone                                No Staff/tenant   Optional
  contact phone                                                           

  Tenant status    Select                              Yes System/staff   Draft, pending,
                                                                          active, former,
                                                                          blocked

  Notes            Restricted text                      No Authorised     Avoid unnecessary
                                                           staff          sensitive notes

  Created by / at  System references                   Yes System         Audit
  ---------------------------------------------------------------------------------------------

Do not collect sensitive personal data without a defined operational or
legal purpose. Identification data and documents require encryption,
access restrictions and retention controls.

## 9.4 Company tenant fields

-   Legal company name.
-   Trading name.
-   Registration number.
-   Country of registration.
-   Registered address.
-   Company email.
-   Company phone.
-   Authorised representative name.
-   Representative role.
-   Representative email and phone.
-   Supporting registration documents.
-   Tenant status.

Company representatives must be separately recorded and authorised. A
representative is not automatically the legal tenant.

## 9.5 Next-of-kin and emergency contacts

Support multiple contacts rather than limiting a tenant to one.

Fields:

-   Contact full name.
-   Relationship.
-   Phone number.
-   Alternative phone.
-   Email, optional.
-   Contact type: next of kin, emergency contact or other.
-   Preferred contact flag.
-   Notes, optional.

## 9.6 Tenant registration workflow

1.  Staff selects Register Tenant.
2.  Staff chooses individual or company.
3.  Staff enters the required profile information.
4.  Staff records identification information where required.
5.  Staff records emergency contacts if available.
6.  Staff associates the tenant with a draft or active lease.
7.  Staff defines whether the person is a primary tenant, co-tenant,
    company tenant, representative or occupant.
8.  System validates unit occupancy rules and tenant limits.
9.  System saves the tenant profile.
10. Staff may send a portal invitation.
11. Tenant verifies contact details and creates credentials.
12. Landlord or authorised manager approves portal activation.
13. System activates access and records the approving user.

## 9.7 Tenant portal activation fields

-   Tenant profile reference.
-   Organisation ID.
-   Invitation email or phone.
-   Invitation channel.
-   Invitation token hash.
-   Expiry timestamp.
-   Contact verification status.
-   Credentials created timestamp.
-   Activation approval status.
-   Approved by.
-   Approval timestamp.
-   Rejection reason, where applicable.
-   Portal account status.

The invitation token must be unpredictable, single-use, time-limited and
stored securely as a hash where practical.

------------------------------------------------------------------------

# 10. Lease and Tenancy Management

## 10.1 Lease principles

A lease is the contractual record connecting one organisation, one or
more tenants and one or more rentable units for a defined period.

A tenant may rent more than two units in the same building. A lease can
cover multiple units. A unit may have different leases over time but
must not have overlapping active occupancy unless an explicitly
supported arrangement permits it.

The model must support:

-   One tenant, one unit.
-   Several co-tenants, one unit.
-   One tenant, several units.
-   Several co-tenants, several units.
-   Company tenant with representatives.
-   Occupants who are not contractual tenants.
-   Separate leases for units rented by the same tenant where
    appropriate.

## 10.2 Lease fields

  --------------------------------------------------------------------------------------
  Field          Type                      Required Entered by     Rules
  -------------- ---------------- ----------------- -------------- ---------------------
  Lease ID       System UUID                    Yes System         Internal identity

  Lease          Text                           Yes System/user    Organisation-unique
  reference                                                        

  Organisation   Foreign key                    Yes System         Tenant isolation
  ID                                                               

  Lease type     Select                         Yes Staff          Residential,
                                                                   commercial, other

  Lease status   Select                         Yes System/staff   Draft, pending
                                                                   approval, active,
                                                                   expired, terminated,
                                                                   cancelled

  Start date     Date                           Yes Staff          Contract start

  End date       Date                   Conditional Staff          Required for
                                                                   fixed-term leases

  Signing date   Date                            No Staff          Actual signing date

  Rent frequency Select                         Yes Staff          Weekly, monthly,
                                                                   quarterly, annual,
                                                                   custom

  Billing day    Integer/date           Conditional Staff          Due-date rule
                 rule                                              

  Currency       Currency                       Yes Staff          Must match permitted
                                                                   accounting policy

  Base rent      Money                          Yes Staff          Contracted rent

  Deposit amount Money                           No Staff          Security deposit

  Notice period  Integer + unit                  No Staff          Days/weeks/months

  Grace period   Integer + unit                  No Staff          Policy or lease
                                                                   override

  Late fee rule  Rule reference                  No Staff          Only where
                                                                   contractually
                                                                   permitted

  Renewal terms  Text/rule                       No Staff          Renewal conditions

  Service charge Text/rule                       No Staff          Contractual basis
  terms                                                            

  Utility terms  Text/rule                       No Staff          Responsibility and
                                                                   allocation

  Joint          Boolean/select         Conditional Staff          Defines tenant
  liability                                                        responsibility

  Lease notes    Text                            No Staff          Non-sensitive
                                                                   operational notes

  Lease document File                            No Staff          Signed agreement

  Created by /   System                         Yes System         Audit
  at             references                                        

  Approved by /  System                 Conditional System         Approval trail
  at             references                                        
  --------------------------------------------------------------------------------------

## 10.3 Lease-unit association fields

-   Lease ID.
-   Unit ID.
-   Unit-specific rent amount.
-   Unit-specific deposit amount, if applicable.
-   Unit start date.
-   Unit end date.
-   Allocation percentage or agreed amount.
-   Occupancy status.
-   Notes.

This allows one lease to cover several units with different rents.

## 10.4 Lease-tenant association fields

-   Lease ID.
-   Tenant ID.
-   Relationship: primary tenant, co-tenant, company tenant,
    representative or other permitted type.
-   Financial responsibility type.
-   Agreed rent share or percentage, if applicable.
-   Deposit responsibility, if applicable.
-   Notice recipient flag.
-   Portal access level.
-   Start date.
-   End date.
-   Status.

The sum of percentage-based allocations should be validated where the
agreement requires a complete allocation. The system must not assume all
co-tenants share rent equally.

## 10.5 Lease creation event flow

1.  Leasing officer or authorised manager selects a unit or several
    units.
2.  System checks that the units belong to the organisation and are
    eligible for leasing.
3.  Staff selects an existing tenant or creates a new tenant profile.
4.  Staff adds co-tenants, company representatives or occupants as
    appropriate.
5.  System checks permitted tenancy arrangement and maximum numbers.
6.  Staff defines primary tenant and financial responsibilities.
7.  Staff enters lease dates, rent, deposit, billing schedule and
    contractual terms.
8.  Staff attaches the lease document or marks it pending signature.
9.  System calculates proposed billing schedule.
10. Lease is submitted for approval if required.
11. Authorised approver reviews and approves or returns it.
12. Lease becomes active on the defined effective date.
13. System creates unit occupancy records.
14. System schedules invoices according to billing policy.
15. Tenant invitations are sent to people authorised for portal access.
16. Unit status changes to occupied when occupancy begins.
17. All events are recorded.

## 10.6 Lease renewal and variation

A renewal or variation must preserve the original agreement and create a
traceable new version or amendment.

Fields:

-   Lease ID.
-   Amendment reference.
-   Amendment type.
-   Proposed effective date.
-   Old value.
-   New value.
-   Reason.
-   Supporting document.
-   Requested by.
-   Approved by.
-   Approval timestamp.
-   Effective status.

Changes to rent, unit allocation, tenants, payment terms or lease dates
must be permission-controlled and audited.

------------------------------------------------------------------------

# 11. Payment Instructions and Financial Operations

## 11.1 Payment model

AMAALPROP does not require each landlord to configure a separate payment
API.

The organisation configures payment instructions. Tenants pay outside
AMAALPROP through the indicated bank or mobile-money channel. The tenant
submits the payment details through the portal. An authorised employee
checks the actual account transaction and verifies or rejects the
submission.

The uploaded receipt or screenshot is evidence only. It does not
automatically establish that funds were received.

## 11.2 Payment instruction hierarchy

Payment instructions may be configured at:

1.  Organisation.
2.  Portfolio.
3.  Property.
4.  Building.
5.  Lease.

A lower-level instruction may override an inherited instruction. The
system must resolve and display the effective instruction to the tenant.

## 11.3 Payment instruction fields

  ----------------------------------------------------------------------------------------
  Field          Type                           Required Entered by    Rules
  -------------- --------------------- ----------------- ------------- -------------------
  Instruction ID System UUID                         Yes System        Internal identity

  Organisation   Foreign key                         Yes System        Owning organisation
  ID                                                                   

  Scope type     Select                              Yes Staff         Organisation,
                                                                       portfolio,
                                                                       property, building,
                                                                       lease

  Scope ID       Reference                   Conditional Staff         Selected scope

  Instruction    Text                                Yes Staff         E.g. Main rent
  name                                                                 account

  Payment        Select                              Yes Staff         Bank, mobile money,
  channel                                                              cash office, other

  Bank/network   Text/select                 Conditional Staff         Channel-dependent
  name                                                                 

  Account holder Text                        Conditional Staff         As registered
  name                                                                 

  Account number Restricted text             Conditional Staff         Securely stored and
                                                                       masked in display

  Mobile money   Restricted phone            Conditional Staff         Securely stored and
  number                                                               masked where
                                                                       appropriate

  Branch         Text                                 No Staff         Bank-specific

  Currency       Currency                            Yes Staff         Supported currency

  Payment        Select/multi-select                 Yes Staff         Rent, deposit,
  purpose                                                              service charge,
                                                                       utilities, other

  Required       Text                                 No Staff         Guidance to tenant
  reference                                                            
  format                                                               

  Tenant         Multiline text                      Yes Staff         Human-readable
  instructions                                                         directions

  Effective from Date/time                           Yes Staff         Start

  Effective      Date/time                            No Staff         End
  until                                                                

  Default        Boolean                             Yes Staff         Default at scope
  instruction                                                          

  Status         Select                              Yes Staff         Draft, active,
                                                                       inactive, expired

  Created by /   System references                   Yes System        Audit
  at                                                                   

  Approved by /  System references           Conditional System        Approval if
  at                                                                   configured
  ----------------------------------------------------------------------------------------

Changes to active payment instructions must be audited and should
trigger notifications to affected tenants where appropriate. Historical
payment submissions must retain the instruction context that was
displayed at the time of submission.

## 11.4 Tenant payment submission fields

  -----------------------------------------------------------------------------------
  Field               Type                   Required Entered by      Rules
  ------------------- ------------- ----------------- --------------- ---------------
  Submission ID       System UUID                 Yes System          Internal
                                                                      identity

  Organisation ID     Foreign key                 Yes System          Organisation
                                                                      scope

  Tenant ID           Foreign key                 Yes System          Submitter

  Lease ID            Foreign key                 Yes Tenant/system   Relevant lease

  Payment instruction Foreign key                 Yes System          Instruction
  ID                                                                  used

  Payment purpose     Select                      Yes Tenant          Rent, deposit,
                                                                      service charge,
                                                                      utility, other

  Amount              Money                       Yes Tenant          Must be greater
                                                                      than zero

  Currency            Currency                    Yes System/tenant   Must be
                                                                      compatible with
                                                                      instruction and
                                                                      lease

  Payment date        Date/time                   Yes Tenant          Date paid

  Payment channel     Select                      Yes Tenant          Bank, mobile
                                                                      money, other

  Transaction         Text                  Yes where Tenant          Reference from
  reference                                 available                 provider

  Payer name          Text                         No Tenant          Optional

  Proof attachment    File                         No Tenant          Supporting
                                                                      evidence

  Tenant note         Text                         No Tenant          Optional
                                                                      explanation

  Submission status   Enum                        Yes System          Pending
                                                                      verification,
                                                                      under review,
                                                                      verified,
                                                                      rejected,
                                                                      disputed,
                                                                      withdrawn

  Submitted at        Timestamp                   Yes System          Immutable

  Reviewed by         User                Conditional System          Verifier
                      reference                                       

  Review timestamp    Timestamp           Conditional System          Audit

  Rejection/dispute   Text                Conditional Staff           Required when
  reason                                                              applicable
  -----------------------------------------------------------------------------------

## 11.5 Payment verification workflow

1.  Tenant opens the relevant lease or invoice.
2.  System displays the effective payment instructions.
3.  Tenant pays through the external channel.
4.  Tenant submits amount, date, reference, channel and optional proof.
5.  System creates a pending-verification submission.
6.  System notifies the authorised finance team.
7.  Verifier opens the submission.
8.  Verifier checks the actual account transaction or other reliable
    financial evidence.
9.  Verifier confirms amount, currency, date, reference and payer
    details as available.
10. Verifier selects Verify, Reject or Dispute.
11. If rejected or disputed, a reason is recorded and the tenant is
    notified.
12. If verified, the system creates the appropriate accounting entries
    through the financial posting service.
13. The verified payment is allocated to one or more outstanding
    charges.
14. An official receipt is generated.
15. Tenant ledger and organisation ledger are updated.
16. Tenant and relevant staff receive notifications.
17. The action is written to the audit log.

A screenshot alone must never trigger step 12.

## 11.6 Payment verification fields

-   Submission ID.
-   Verification result.
-   Actual transaction reference checked.
-   Verified amount.
-   Verified currency.
-   Verified date.
-   Financial account or statement reference.
-   Verification notes.
-   Verifier user ID.
-   Verification timestamp.
-   Rejection or dispute reason.
-   Related approval, if required.

The system should prevent the same external transaction reference from
being posted more than once within the relevant account and
organisation, subject to configurable provider reference rules.

## 11.7 Payment allocations

A verified payment may be allocated across multiple invoices or charge
items.

Allocation fields:

-   Payment ID.
-   Invoice ID.
-   Invoice item ID, optional.
-   Allocated amount.
-   Currency.
-   Allocation date.
-   Allocated by.
-   Allocation status.
-   Reversal reference, if applicable.

The sum of allocations must not exceed the available payment amount
unless an explicit overpayment or credit balance process is used.

## 11.8 Receipts

Receipt fields:

-   Receipt number.
-   Organisation.
-   Tenant.
-   Lease.
-   Payment.
-   Amount and currency.
-   Payment date.
-   Verification date.
-   Payment channel.
-   Transaction reference.
-   Allocated charges.
-   Issued by/system.
-   Receipt status.
-   Reversal reference if later reversed.

A receipt must clearly distinguish the date the tenant says they paid
from the date the landlord verified the payment.

## 11.9 Partial payments and overpayments

-   Partial payment reduces the outstanding balance by the allocated
    amount.
-   An unpaid balance remains outstanding.
-   Overpayments must be recorded as an unapplied credit or allocated
    according to an authorised rule.
-   A payment must not be allocated twice.
-   Refunds must be recorded as separate financial transactions.
-   Reversals must preserve the original transaction and reason.

## 11.10 Bank statement reconciliation

Future-ready reconciliation should support:

-   Import of bank statement files.
-   Statement account selection.
-   Transaction date, value date, description, reference, debit, credit
    and balance.
-   Suggested matching to tenant submissions.
-   Duplicate detection.
-   Unmatched transaction review.
-   Manual match and unmatch with audit.
-   Reconciliation completion and period locking.

The system must not claim that a transaction has been received solely
because a tenant uploaded an image.

------------------------------------------------------------------------

# 12. Double-Entry Accounting

## 12.1 Accounting requirement

AMAALPROP must implement a full double-entry accounting ledger for each
organisation.

Every posted journal entry must satisfy:

**Total debits = total credits.**

The ledger is the financial source of truth. Tenant balances, property
financial reports and organisation reports should be derived from posted
accounting entries and approved operational records, not from arbitrary
manually edited balance fields.

## 12.2 Accounting foundations

-   Organisation-specific chart of accounts.
-   Accounting periods.
-   Journal entries and journal lines.
-   General ledger.
-   Accounts receivable.
-   Accounts payable.
-   Income and expense accounts.
-   Assets and liabilities.
-   Security deposit liabilities.
-   Bank and cash accounts.
-   Property and portfolio dimensions.
-   Tenant and lease references.
-   Reversals and adjustments.
-   Trial balance.
-   Profit and loss.
-   Balance sheet.
-   Cash flow reporting.
-   Bank reconciliation.

## 12.3 Chart of accounts fields

  ------------------------------------------------------------------------
  Field            Type                          Required Purpose
  ---------------- ---------------- --------------------- ----------------
  Account ID       System UUID                        Yes Internal
                                                          identity

  Organisation ID  Foreign key                        Yes Organisation

  Account code     Text                               Yes Unique code
                                                          within
                                                          organisation

  Account name     Text                               Yes Display name

  Account type     Select                             Yes Asset,
                                                          liability,
                                                          equity, income,
                                                          expense

  Account subtype  Select                              No E.g. bank,
                                                          receivable,
                                                          rental income

  Parent account   Foreign key                         No Hierarchical
  ID                                                      reporting

  Normal balance   Select                             Yes Debit or credit

  Currency policy  Select                             Yes Organisation
                                                          base or
                                                          permitted
                                                          foreign currency

  Is control       Boolean                            Yes Prevent
  account                                                 inappropriate
                                                          direct postings

  Is system        Boolean                            Yes Protected
  account                                                 system-created
                                                          account

  Status           Select                             Yes Active or
                                                          inactive

  Created by / at  System                             Yes Audit
                   references                             
  ------------------------------------------------------------------------

## 12.4 Journal entry fields

  --------------------------------------------------------------------------------------
  Field            Type                          Required Purpose
  ---------------- ---------------- --------------------- ------------------------------
  Journal ID       System UUID                        Yes Internal identity

  Organisation ID  Foreign key                        Yes Organisation

  Journal number   Text                               Yes Unique sequential/reference
                                                          number

  Journal type     Select                             Yes Invoice, receipt, expense,
                                                          deposit, refund, adjustment,
                                                          reversal, opening balance

  Transaction date Date                               Yes Accounting date

  Posting date     Date/time                          Yes Actual posting time

  Accounting       Foreign key                        Yes Period
  period ID                                               

  Description      Text                               Yes Explanation

  Source module    Text/enum                          Yes Originating business module

  Source record ID Reference                          Yes Related
                                                          invoice/payment/expense/etc.

  Currency         Currency                           Yes Transaction currency

  Exchange rate    Decimal                    Conditional Required for foreign-currency
                                                          conversion

  Base currency    Money                              Yes Converted amount
  amount                                                  

  Status           Enum                               Yes Draft, pending approval,
                                                          posted, reversed

  Created by       User reference                     Yes Creator

  Approved by      User reference             Conditional Approver

  Posted by        User reference             Conditional Poster

  Reversal of      Foreign key                         No Original journal if reversal
  journal ID                                              

  Created at       Timestamp                          Yes Audit
  --------------------------------------------------------------------------------------

## 12.5 Journal line fields

-   Journal entry ID.
-   Line number.
-   Account ID.
-   Debit amount.
-   Credit amount.
-   Transaction currency.
-   Base currency debit.
-   Base currency credit.
-   Property dimension.
-   Building dimension, optional.
-   Unit dimension, optional.
-   Tenant dimension, optional.
-   Lease dimension, optional.
-   Description.
-   Tax code, where applicable.
-   Vendor reference, where applicable.

Each line must have a valid account and must not contain both a debit
and credit amount greater than zero. The journal must balance in base
currency and follow the configured multi-currency accounting policy.

## 12.6 Illustrative accounting events

The examples below are conceptual. The final account mapping must be
configured in the organisation's chart of accounts and reviewed for
applicable accounting standards and tax rules.

### Rent invoice for UGX 1,000,000

  Account                            Debit      Credit
  ---------------------------- ----------- -----------
  Tenant accounts receivable     1,000,000           0
  Rental income                          0   1,000,000

### Verified rent payment for UGX 1,000,000

  Account                            Debit      Credit
  ---------------------------- ----------- -----------
  Bank                           1,000,000           0
  Tenant accounts receivable             0   1,000,000

### Security deposit received for UGX 500,000

  Account                                 Debit    Credit
  ----------------------------------- --------- ---------
  Bank                                  500,000         0
  Tenant security deposit liability           0   500,000

### Landlord maintenance expense paid for UGX 200,000

  Account                   Debit    Credit
  --------------------- --------- ---------
  Maintenance expense     200,000         0
  Bank                          0   200,000

## 12.7 Posting controls

-   Draft journals may be edited by authorised users.
-   Posted journals cannot be edited or deleted.
-   Corrections use reversing or adjusting journals.
-   Journal posting must be atomic.
-   The system must reject unbalanced journals.
-   Period locks must prevent unauthorised backdated posting.
-   Approval rules may apply to journals above configured thresholds.
-   Every posting must reference its source business event.
-   Duplicate posting protection must use idempotency keys or equivalent
    controls.
-   A failed posting must not leave partial ledger entries.

## 12.8 Tenant and property statements

Tenant statements should show:

-   Opening balance.
-   Invoices and charges.
-   Verified payments.
-   Allocations.
-   Credits.
-   Adjustments.
-   Reversals.
-   Closing balance.

Property statements and reports should derive property dimensions from
ledger entries. Consolidated reports should aggregate across the
organisation without losing property-level detail.

------------------------------------------------------------------------

# 13. Billing, Rent Policies and Tenant Financial Health

## 13.1 Charge types

Support configurable charge types:

-   Rent.
-   Security deposit.
-   Service charge.
-   Water.
-   Electricity.
-   Waste collection.
-   Parking.
-   Late fee.
-   Maintenance recovery.
-   Other contractual charges.

Each charge type should map to appropriate accounting treatment.

## 13.2 Billing schedule fields

-   Schedule ID.
-   Organisation ID.
-   Lease ID.
-   Charge type.
-   Amount.
-   Currency.
-   Frequency.
-   Start date.
-   End date.
-   Due-date rule.
-   Advance or arrears billing setting.
-   Proration policy.
-   Grace period.
-   Reminder policy.
-   Late fee policy.
-   Status.
-   Created by.
-   Approved by.

## 13.3 Invoice fields

-   Invoice ID.
-   Organisation ID.
-   Invoice number.
-   Tenant.
-   Lease.
-   Property and unit references.
-   Issue date.
-   Due date.
-   Currency.
-   Subtotal.
-   Tax, where applicable.
-   Total.
-   Amount paid.
-   Outstanding balance, derived.
-   Invoice status.
-   Journal reference.
-   Notes.
-   Created by and timestamp.

Invoice line fields:

-   Charge type.
-   Description.
-   Quantity.
-   Unit amount.
-   Line total.
-   Tax code.
-   Property/unit reference.
-   Service period.

## 13.4 Due-date and reminder configuration

Landlords should configure:

-   Rent due day or date rule.
-   Billing frequency.
-   Reminder days before due date.
-   Reminder on due date.
-   Grace period.
-   Late fee rule.
-   Escalation schedule.
-   Different rules for residential and commercial leases.
-   Lease-specific overrides.

Example policy: rent is due on the 1st, a reminder is sent three days
before, a five-day grace period applies, and the financial status
becomes overdue after the configured grace period. This is an example,
not a fixed platform rule.

## 13.5 Green, orange and red status

Financial health status must be computed from charges, due dates,
verified allocations, grace periods and policy rules.

-   **Green:** no overdue obligation under the applicable policy.
-   **Orange:** due date is approaching, grace period applies, or a
    payment discrepancy requires attention.
-   **Red:** an outstanding obligation has passed the configured grace
    period or contractual threshold.

The system must record the status reason and status history. Pending
verification, disputed payments and unapplied credits require explicit
policy handling. The platform must not silently treat every pending
submission as a verified payment.

Financial health must remain separate from physical unit condition and
clearance status.

## 13.6 Status history fields

-   Tenant/lease ID.
-   Previous status.
-   New status.
-   Effective timestamp.
-   Reason code.
-   Relevant invoice/payment references.
-   Policy version.
-   Generated by system or user.
-   Override user and reason, if applicable.

------------------------------------------------------------------------

# 14. Expenses, Suppliers and Maintenance Costs

## 14.1 Expense record fields

  Field                 Type                     Required Entered by
  --------------------- ------------------- ------------- -------------
  Expense ID            System UUID                   Yes System
  Organisation ID       Foreign key                   Yes System
  Expense reference     Text                          Yes System/user
  Expense category      Select                        Yes Staff
  Description           Text                          Yes Staff
  Vendor                Reference/text                 No Staff
  Property              Reference             Conditional Staff
  Building              Reference                      No Staff
  Unit                  Reference                      No Staff
  Amount                Money                         Yes Staff
  Currency              Currency                      Yes Staff
  Expense date          Date                          Yes Staff
  Due date              Date                           No Staff
  Payment date          Date                           No Staff
  Payment method        Select                         No Staff
  Supporting document   File                           No Staff
  Approval status       Enum                          Yes System
  Accounting status     Enum                          Yes System
  Created by / at       System references             Yes System

## 14.2 Expense workflow

1.  Staff creates an expense request or records an incurred expense.
2.  System checks scope and permission.
3.  Staff attaches quotation, invoice or supporting evidence.
4.  System applies approval policy and thresholds.
5.  Approver approves, rejects or requests changes.
6.  Authorised finance staff records payment when made.
7.  Accounting posting service creates balanced journal entries.
8.  Expense appears in property and organisation reports.
9.  Audit history records the request, approvals and posting.

## 14.3 Maintenance request fields

-   Request ID.
-   Organisation.
-   Property, building, floor and unit.
-   Requester type and ID.
-   Tenant/lease reference if relevant.
-   Category.
-   Priority.
-   Description.
-   Date reported.
-   Images/files.
-   Assigned employee.
-   Contractor, if applicable.
-   Estimated cost.
-   Approval status.
-   Work status.
-   Completion notes.
-   Completion evidence.
-   Closed by and timestamp.

Maintenance status examples:

-   Submitted.
-   Acknowledged.
-   Inspection required.
-   Awaiting approval.
-   Approved.
-   Assigned.
-   In progress.
-   Awaiting parts/vendor.
-   Completed.
-   Rejected.
-   Cancelled.

------------------------------------------------------------------------

# 15. Inspections, Move-Out and Clearance

## 15.1 Move-out request

The tenant submits a formal request through the tenant portal.

### Move-out request fields

  Field                     Type                 Required Entered by
  ------------------------- ------------------ ---------- ---------------
  Request ID                System UUID               Yes System
  Organisation ID           Foreign key               Yes System
  Lease ID                  Foreign key               Yes Tenant/system
  Tenant ID                 Foreign key               Yes Tenant/system
  Unit IDs                  Related records           Yes System
  Notice date               Date                      Yes Tenant/system
  Intended departure date   Date                      Yes Tenant
  Reason                    Select/text                No Tenant
  Forwarding contact        Text/phone/email           No Tenant
  Maintenance concerns      Multiline text             No Tenant
  Supporting files          File collection            No Tenant
  Request status            Enum                      Yes System
  Submitted at              Timestamp                 Yes System

The system compares the notice date and intended departure date with
lease notice terms. It should flag a possible mismatch for review rather
than automatically making a legal determination.

## 15.2 Inspection templates

Organisations can create different inspection templates for residential,
commercial and other unit types.

### Inspection template fields

-   Template ID.
-   Organisation ID.
-   Template name.
-   Unit category.
-   Applicable lease type.
-   Description.
-   Version.
-   Status.
-   Created by.
-   Approved by.

### Inspection checklist item fields

-   Item ID.
-   Template ID.
-   Category.
-   Item name.
-   Instructions.
-   Response type.
-   Required flag.
-   Evidence required flag.
-   Severity options.
-   Sort order.

## 15.3 Example residential checklist

-   Walls and paint.
-   Doors and locks.
-   Windows and glass.
-   Electrical installations.
-   Plumbing and water.
-   Ceiling and floors.
-   Kitchen fittings.
-   Bathroom fittings.
-   Sanitation and cleanliness.
-   Fixtures and fittings.
-   Meter readings.
-   Keys and access cards.
-   General condition.
-   Photographs.

Commercial templates may add shutters, shopfronts, signage, partitions,
counters, business fittings, fire-safety equipment and other
landlord-provided installations.

## 15.4 Inspection report fields

  Field                    Type                   Required Entered by
  ------------------------ ----------------- ------------- ------------
  Inspection ID            System UUID                 Yes System
  Clearance case ID        Foreign key                 Yes System
  Template version         Reference                   Yes System
  Inspector ID             User reference              Yes System
  Inspection date/time     Timestamp                   Yes Caretaker
  Item result              Select                      Yes Caretaker
  Condition notes          Text                Conditional Caretaker
  Damage flag              Boolean                     Yes Caretaker
  Severity                 Select              Conditional Caretaker
  Photo evidence           File collection     Conditional Caretaker
  Meter reading            Decimal/text        Conditional Caretaker
  Key/card count           Integer             Conditional Caretaker
  Overall recommendation   Select                      Yes Caretaker
  Submitted at             Timestamp                   Yes System

## 15.5 Clearance workflow

1.  Tenant submits move-out request.
2.  System creates a clearance case.
3.  Assigned caretaker receives a task.
4.  Caretaker schedules and conducts inspection.
5.  Caretaker completes checklist and uploads evidence.
6.  Caretaker submits the report.
7.  Property manager reviews the report.
8.  Property manager verifies, rejects or requests additional
    inspection.
9.  Financial officer checks outstanding rent, service charges,
    utilities and other obligations.
10. Financial officer prepares deposit settlement.
11. Landlord or delegated authority reviews the case.
12. Approver approves, rejects or returns it for further action.
13. Handover is completed and recorded.
14. Lease is closed or terminated according to the effective date and
    approved process.
15. Unit occupancy is closed.
16. Unit becomes available for re-leasing when all relevant operational
    conditions are satisfied.
17. Historical lease, payment, inspection and clearance records remain
    available according to retention policy.

## 15.6 Clearance case fields

-   Clearance case ID.
-   Organisation ID.
-   Lease ID.
-   Tenant ID(s).
-   Unit ID(s).
-   Move-out request ID.
-   Assigned caretaker.
-   Assigned property manager.
-   Financial reviewer.
-   Final approver.
-   Inspection status.
-   Financial settlement status.
-   Handover status.
-   Final clearance status.
-   Requested departure date.
-   Actual handover date.
-   Final approval date.
-   Notes.
-   Created by and timestamp.

## 15.7 Approval responsibilities

Default flow:

-   Caretaker: performs physical inspection and submits evidence.
-   Property manager: verifies inspection and identifies required
    action.
-   Accountant/finance officer: prepares financial settlement.
-   Landlord or delegated authority: final approval.

Organisations may configure delegated substitutes, but the default
separation of duties should remain visible and auditable.

A person should not approve their own action where separation-of-duty
policy prohibits it.

## 15.8 Deposit settlement fields

-   Clearance case ID.
-   Deposit originally received.
-   Deposit balance.
-   Outstanding rent.
-   Outstanding utilities.
-   Service charges.
-   Approved damage deductions.
-   Other contractual deductions.
-   Refund amount.
-   Additional amount due.
-   Evidence references.
-   Prepared by.
-   Reviewed by.
-   Approved by.
-   Settlement status.
-   Accounting journal reference.

The system must preserve evidence and reasons for every deduction.
Disputed charges should be marked as disputed and not silently treated
as agreed.

## 15.9 Legal and operational distinction

A tenant's lawful ability to give notice or terminate a lease must not
be confused with completion of financial or physical clearance.
AMAALPROP records contractual obligations and operational steps; it must
not make unsupported legal determinations or unlawfully prevent a tenant
from exercising legal rights.

------------------------------------------------------------------------

# 16. Approval and Workflow Engine

## 16.1 Purpose

A configurable workflow engine controls decisions that require review or
approval. It should be reusable across leases, payments, expenses,
maintenance, staff assignments and clearance.

## 16.2 Approval policy fields

-   Policy ID.
-   Organisation ID.
-   Policy name.
-   Workflow type.
-   Scope type and scope ID.
-   Conditions.
-   Ordered approval steps.
-   Required role or permission.
-   Required approver scope.
-   Monetary threshold, where applicable.
-   Delegation rules.
-   Escalation interval.
-   Effective date.
-   Expiry date.
-   Status.
-   Created by.
-   Approved by.

## 16.3 Approval request fields

-   Request ID.
-   Organisation ID.
-   Workflow type.
-   Source record type and ID.
-   Requested action.
-   Requester.
-   Current step.
-   Assigned approver or approver group.
-   Requested amount, if relevant.
-   Status.
-   Submitted timestamp.
-   Due timestamp.
-   Completed timestamp.
-   Resolution reason.

## 16.4 Approval action fields

-   Approval request ID.
-   Step number.
-   Actor user ID.
-   Decision: approve, reject, return, delegate.
-   Comments.
-   Action timestamp.
-   Delegated-to user, if applicable.
-   Evidence attachment, if required.

## 16.5 Workflow rules

-   Approval sequence must be explicit.
-   Approval authority must be checked at the time of action.
-   Delegation must be time-bound and auditable.
-   Rejected or returned requests must include a reason.
-   Material changes after approval may require resubmission.
-   The workflow must prevent unauthorised self-approval where
    configured.
-   Approval must not directly mutate financial history without the
    relevant posting process.
-   A superior may review delegated activity, but must not silently
    rewrite historical actions.

------------------------------------------------------------------------

# 17. Notifications and Communication

## 17.1 Channels

Initial channels:

-   In-app notification.
-   Email.
-   SMS.

Future channel:

-   WhatsApp Business, subject to approved integration and consent
    requirements.

## 17.2 Notification event types

-   Organisation registration and verification.
-   Employee invitation.
-   Employee activation or suspension.
-   Tenant invitation.
-   Tenant portal activation.
-   Lease created, approved, renewed or nearing expiry.
-   Invoice generated.
-   Payment due reminder.
-   Payment submission received.
-   Payment verified, rejected or disputed.
-   Receipt issued.
-   Maintenance request update.
-   Inspection scheduled.
-   Inspection report submitted.
-   Move-out request received.
-   Clearance approval or rejection.
-   Deposit settlement update.
-   Approval request assigned or escalated.

## 17.3 Notification fields

-   Notification ID.
-   Organisation ID.
-   Recipient user or tenant reference.
-   Channel.
-   Event type.
-   Template ID/version.
-   Subject.
-   Message payload.
-   Related record type and ID.
-   Created timestamp.
-   Sent timestamp.
-   Delivery status.
-   Provider reference.
-   Failure reason.
-   Read timestamp, where applicable.

Notifications must not expose sensitive information unnecessarily. Email
and SMS should direct users to the authenticated portal for confidential
details.

## 17.4 Communication history

The system should preserve a communication timeline linked to the
relevant tenant, lease, unit, property or employee.

Examples:

-   Rent reminder sent.
-   Payment submitted.
-   Payment rejected with reason.
-   Maintenance complaint lodged.
-   Lease renewal notice issued.
-   Inspection scheduled.
-   Move-out request submitted.
-   Clearance approved.

------------------------------------------------------------------------

# 18. Dashboards, Reports and Analytics

## 18.1 Landlord dashboard

Potential metrics:

-   Total properties.
-   Total buildings.
-   Total rentable units.
-   Occupied units.
-   Vacant units.
-   Units under maintenance.
-   Occupancy rate.
-   Rent billed.
-   Rent collected.
-   Outstanding rent.
-   Pending payment verifications.
-   Disputed payments.
-   Expenses.
-   Net operating result, subject to configured accounting definitions.
-   Upcoming lease expiries.
-   Open maintenance tasks.
-   Pending approvals.
-   Move-out cases awaiting action.

## 18.2 Property manager dashboard

-   Assigned properties and buildings.
-   Unit occupancy.
-   Tenants requiring attention.
-   Payment health by lease.
-   Payment submissions awaiting review.
-   Maintenance tasks.
-   Inspections due.
-   Lease renewals.
-   Move-out cases.
-   Approval requests.

## 18.3 Caretaker dashboard

-   Assigned buildings and units.
-   Inspection tasks.
-   Maintenance assignments.
-   Upcoming appointments.
-   Open work orders.
-   Handover tasks.
-   Tasks awaiting submission.

## 18.4 Accountant dashboard

-   Invoices issued.
-   Payments pending verification.
-   Verified payments awaiting allocation.
-   Unallocated credits.
-   Overdue balances.
-   Expenses awaiting approval or posting.
-   Deposit liabilities.
-   Bank reconciliation status.
-   Period close status.

## 18.5 Tenant dashboard

-   Current lease and units.
-   Rent and charge summary.
-   Next due date.
-   Applicable payment instructions.
-   Payment submission history.
-   Verified receipts.
-   Maintenance requests.
-   Notices.
-   Lease expiry or renewal information.
-   Move-out request and clearance status.

## 18.6 Report filters

Reports should support relevant filters:

-   Organisation.
-   Portfolio.
-   Region.
-   Property.
-   Building.
-   Floor.
-   Unit.
-   Unit category.
-   Tenant type.
-   Lease status.
-   Manager/caretaker.
-   Date range.
-   Currency.
-   Payment status.
-   Financial status.

Users must only see results within their authorised scope.

## 18.7 Staff performance

Performance reports may include:

-   Task completion.
-   Maintenance response and resolution times.
-   Inspection completion.
-   Lease administration turnaround.
-   Payment verification turnaround.
-   Approval turnaround.

Performance metrics must be contextual. Staff should not be unfairly
penalised for inherited arrears, unavailable contractors, tenant delays
or other factors outside their control.

------------------------------------------------------------------------

# 19. Documents and File Management

## 19.1 Document categories

-   Organisation registration documents.
-   Property documents.
-   Building plans.
-   Unit images and inventories.
-   Tenant identification documents.
-   Tenant agreements.
-   Lease agreements and amendments.
-   Payment evidence.
-   Receipts.
-   Expense invoices and quotations.
-   Inspection photographs.
-   Maintenance evidence.
-   Handover documents.
-   Deposit settlement evidence.

## 19.2 Document metadata fields

-   Document ID.
-   Organisation ID.
-   Related entity type and ID.
-   Document category.
-   Original filename.
-   Stored object reference.
-   MIME type.
-   File size.
-   Checksum.
-   Uploaded by.
-   Uploaded timestamp.
-   Access classification.
-   Retention category.
-   Expiry date, if applicable.
-   Scan status.
-   Version.
-   Status.

Files must be stored privately. The database should store metadata and
secure object references rather than relying on public file URLs.

------------------------------------------------------------------------

# 20. Audit, Activity History and Data Governance

## 20.1 Actions requiring audit records

-   Organisation setting changes.
-   Employee invitations and activation.
-   Role and permission changes.
-   Scope assignments and revocations.
-   Property and unit changes.
-   Tenant identity changes.
-   Lease creation and amendments.
-   Payment instruction changes.
-   Payment verification, rejection and reversal.
-   Journal creation, approval, posting and reversal.
-   Expense approval and posting.
-   Inspection submission and verification.
-   Clearance decisions.
-   Document access where required by policy.
-   Data export.
-   Account suspension and restoration.

## 20.2 Audit log fields

-   Audit ID.
-   Organisation ID.
-   Actor user ID.
-   Actor role and scope snapshot.
-   Action.
-   Entity type.
-   Entity ID.
-   Previous value or change summary, subject to sensitive-data
    controls.
-   New value or change summary.
-   Reason.
-   IP/device metadata where lawfully collected.
-   Timestamp.
-   Request/correlation ID.
-   Result.

Audit logs should be append-only or protected against ordinary
application-user modification. Sensitive fields should not be copied
into logs in plain text.

## 20.3 Data retention

The organisation must be able to configure retention categories where
legally permitted. The platform should define default retention rules
for financial records, leases, identity documents, communication records
and operational evidence.

Deletion requests must be handled according to applicable legal,
contractual and accounting retention obligations. Financial records that
must be retained should be anonymised or restricted where appropriate
rather than casually deleted.

------------------------------------------------------------------------

# 21. Security and Privacy Requirements

## 21.1 Organisation isolation

Every organisation-owned record must be associated with an organisation.
All reads and writes must enforce the organisation scope.

The system must prevent:

-   Accessing another organisation's data by changing a URL or
    identifier.
-   Reusing another organisation's invitation.
-   Linking a tenant account to an unrelated tenant profile.
-   Viewing documents outside assigned scope.
-   Posting a financial transaction into another organisation's ledger.

## 21.2 Authentication

-   Secure password hashing.
-   Multi-factor authentication capability, especially for privileged
    roles.
-   Secure session expiry and revocation.
-   Rate limiting for authentication attempts.
-   Password reset with time-limited tokens.
-   Email/phone verification.
-   Optional single sign-on in future enterprise plans.

## 21.3 Authorisation

Every protected operation must check:

1.  Authenticated identity.
2.  Active organisation membership.
3.  Account status.
4.  Required permission.
5.  Scope assignment.
6.  Approval authority, if the action is an approval.
7.  Record ownership and relationship.

## 21.4 Sensitive data

-   Encrypt sensitive identification data at rest.
-   Use TLS for data in transit.
-   Restrict access to identity documents.
-   Mask bank account and mobile-money details where appropriate.
-   Avoid exposing sensitive details in notifications.
-   Record sensitive document access where required.
-   Use secure secrets management.
-   Define retention and disposal procedures.

## 21.5 File security

-   Validate file type and size.
-   Scan uploads for malware.
-   Store files outside public web roots.
-   Use expiring authorised download links.
-   Enforce organisation and entity-level access.
-   Keep file metadata and access logs.
-   Prevent executable content from being served as trusted application
    content.

------------------------------------------------------------------------

# 22. Technical Architecture

## 22.1 Recommended architectural style

Begin with a modular monolith. Each domain has clear internal
boundaries, service interfaces and data ownership. This keeps initial
deployment manageable while making future extraction possible if scale
or operational needs justify it.

## 22.2 Logical architecture

``` text
Web / Mobile Browser Interfaces
             |
             v
        Backend API
             |
  Authentication and Authorisation
             |
  -----------------------------------------
  | Organisation and Access Management   |
  | Property and Asset Management         |
  | Tenant and Lease Management           |
  | Billing and Financial Management      |
  | Double-Entry Accounting               |
  | Maintenance and Inspections           |
  | Workflow and Approvals                |
  | Notifications and Communications      |
  | Reporting and Analytics               |
  -----------------------------------------
             |
  -----------------------------------------
  | Relational Database                   |
  | Private Document/Object Storage       |
  | Background Job Queue                  |
  | Cache (where justified)               |
  -----------------------------------------
             |
  Email / SMS / Optional External Services
```

## 22.3 Technology requirements

The final stack should be selected after technical evaluation. The
architecture requires:

-   Responsive web frontend.
-   Backend API.
-   Relational database with transaction support.
-   Secure authentication and authorisation.
-   Private file/object storage.
-   Background job processing.
-   Email and SMS provider adapters.
-   Reporting and export capability.
-   Monitoring and error tracking.
-   Automated testing and deployment pipeline.

PostgreSQL is a strong candidate for the primary database because of
relational integrity, transactional consistency and reporting needs. The
choice of programming language and framework should be based on team
capability, ecosystem, maintainability and deployment requirements.

## 22.4 Domain boundaries

Recommended modules:

1.  Identity and Organisation.
2.  Access Control.
3.  Property and Asset.
4.  Tenant.
5.  Lease.
6.  Billing.
7.  Payment Verification.
8.  Accounting.
9.  Maintenance.
10. Inspection and Clearance.
11. Workflow and Approvals.
12. Notifications.
13. Documents.
14. Reporting and Analytics.
15. Audit and Compliance.

Modules should communicate through defined application services and
domain events rather than directly modifying each other's internal data.

## 22.5 Event-driven internal processing

The modular monolith may use internal domain events and a transactional
outbox pattern for reliable asynchronous work.

Examples:

-   `OrganisationCreated`
-   `EmployeeInvited`
-   `EmployeeActivated`
-   `PropertyCreated`
-   `UnitCreated`
-   `TenantRegistered`
-   `TenantPortalActivated`
-   `LeaseSubmittedForApproval`
-   `LeaseActivated`
-   `InvoiceGenerated`
-   `PaymentSubmitted`
-   `PaymentVerified`
-   `PaymentRejected`
-   `JournalPosted`
-   `MaintenanceRequested`
-   `InspectionSubmitted`
-   `InspectionVerified`
-   `MoveOutRequested`
-   `ClearanceApproved`
-   `LeaseClosed`

Events must have unique IDs, organisation context, event type, event
version, occurrence time, actor/source and related entity identifiers.

Consumers must be idempotent. Reprocessing an event must not create
duplicate invoices, receipts, journal entries or notifications.

------------------------------------------------------------------------

# 23. End-to-End Event Flows

This section defines how the main business events move through
AMAALPROP.

## 23.1 Organisation creation

``` text
Registrant submits organisation details
 -> Validate required data and email uniqueness
 -> Create organisation in onboarding state
 -> Verify official email
 -> Create owner identity and membership
 -> Assign organisation-owner role
 -> Apply default settings and chart-of-accounts template
 -> Activate organisation
 -> Notify owner
 -> Write audit events
```

Failure handling: - Duplicate email: reject and explain the
one-organisation rule. - Invalid registration data: return field-level
errors. - Verification expiry: allow secure resend. - Partial setup:
preserve draft onboarding state without exposing an active organisation.

## 23.2 Employee onboarding

``` text
Authorised inviter selects employee
 -> Validate email uniqueness
 -> Select role, scope and reporting manager
 -> Create invitation
 -> Send invitation
 -> Employee verifies contact
 -> Employee creates credentials
 -> Validate invitation and organisation
 -> Activate account/assignment
 -> Notify employee and inviter
 -> Record audit history
```

If the invitation expires, the inviter must issue a new invitation. If
the employee's role or scope changes, the system must record the change
and enforce the new permissions immediately.

## 23.3 Property and unit creation

``` text
Authorised user creates property
 -> Validate organisation ownership
 -> Save property
 -> Optionally create buildings
 -> Optionally create floors
 -> Create units
 -> Validate display-code uniqueness
 -> Configure unit category and attributes
 -> Configure occupancy rules
 -> Set availability status
 -> Publish property records to authorised staff
 -> Record audit history
```

A standalone unit may be created without a building or floor. A unit
cannot be attached to a building from another property.

## 23.4 Tenant registration and activation

``` text
Staff creates tenant profile
 -> Select individual/company/other supported type
 -> Enter required profile fields
 -> Add contacts and identification where required
 -> Associate tenant with draft lease
 -> Define tenant relationship and occupancy role
 -> Validate occupancy limits
 -> Save tenant profile
 -> Send invitation
 -> Tenant verifies contact and creates credentials
 -> Landlord/authorised manager reviews activation
 -> Activate portal access
 -> Notify tenant
 -> Record audit history
```

## 23.5 Lease creation

``` text
Staff selects one or more units
 -> Check unit availability and organisation scope
 -> Select/create tenant profiles
 -> Add co-tenants and authorised occupants
 -> Set primary tenant and responsibility rules
 -> Validate unit occupancy limits
 -> Enter lease dates and financial terms
 -> Attach agreement
 -> Calculate proposed billing schedule
 -> Submit for approval where required
 -> Approver reviews
 -> Activate lease on effective date
 -> Create occupancy records
 -> Schedule billing
 -> Notify tenants and staff
 -> Update unit status
```

## 23.6 Invoice generation

``` text
Billing schedule reaches generation date
 -> Check active lease and schedule
 -> Determine applicable units and charge items
 -> Calculate amount and proration if configured
 -> Create invoice and invoice lines
 -> Validate tax and currency rules
 -> Create balanced draft journal
 -> Apply approval if required
 -> Post journal
 -> Mark invoice issued
 -> Notify tenant
 -> Update financial health calculation
```

Billing jobs must be idempotent. A retry must not issue a duplicate
invoice for the same lease, charge period and schedule.

## 23.7 Payment submission and verification

``` text
Tenant views effective payment instructions
 -> Pays externally
 -> Submits amount/date/reference/channel
 -> System creates pending submission
 -> Notify authorised finance users
 -> Verifier checks actual transaction
 -> Verify / reject / dispute
 -> If verified, create payment record
 -> Post balanced accounting journal
 -> Allocate to invoice(s)
 -> Generate receipt
 -> Update tenant statement and arrears
 -> Recalculate financial health
 -> Notify tenant and relevant staff
 -> Audit all actions
```

If journal posting fails, the payment must not be marked as fully
posted. The system must preserve the verification decision and retry
safely or route the issue to finance operations without duplicating the
transaction.

## 23.8 Expense approval and payment

``` text
Staff creates expense request
 -> Attach supporting evidence
 -> Validate property scope and budget/threshold policy
 -> Route to approver
 -> Approve / reject / return
 -> Finance records actual payment
 -> Create balanced journal
 -> Post journal
 -> Update property and organisation reports
 -> Notify requester
 -> Audit
```

## 23.9 Maintenance request

``` text
Tenant or staff reports issue
 -> Create maintenance request
 -> Validate unit and requester access
 -> Assign caretaker/property manager
 -> Inspect and estimate work
 -> Request cost approval if needed
 -> Approve work
 -> Assign staff/contractor
 -> Record work progress
 -> Submit completion evidence
 -> Verify completion
 -> Close request
 -> Record any expense and accounting entry
 -> Notify requester
```

## 23.10 Tenant move-out and clearance

``` text
Tenant submits move-out request
 -> Validate tenant/lease relationship
 -> Check notice terms and flag exceptions
 -> Create clearance case
 -> Assign caretaker
 -> Schedule inspection
 -> Caretaker completes checklist and evidence
 -> Submit inspection
 -> Property manager verifies or returns it
 -> Finance prepares final account and deposit settlement
 -> Landlord/delegated authority reviews
 -> Approve / reject / return
 -> Complete handover and key records
 -> Close occupancy and lease according to effective dates
 -> Make unit available when operationally ready
 -> Preserve historical records
 -> Notify tenant and responsible staff
```

A rejected inspection must create a follow-up action, not erase the
original report. A disputed financial item must remain traceable and
must not be silently converted into an agreed deduction.

## 23.11 Staff reassignment

``` text
Authorised administrator selects employee
 -> Select new role/scope/reporting relationship
 -> Set effective date
 -> Validate authority
 -> Approve if required
 -> End or supersede old assignment
 -> Activate new assignment
 -> Recalculate access
 -> Notify employee and supervisors
 -> Record audit history
```

Reassignment must not alter historical records or the identity of the
employee who performed earlier actions.

------------------------------------------------------------------------

# 24. Data Model and Relationship Requirements

The ERD should be derived from these domain concepts. This list is not a
final physical schema; normalisation, exact cardinalities and
constraints must be settled during database design.

## 24.1 Core entities

### Organisation and access

-   `organisations`
-   `organisation_settings`
-   `users`
-   `organisation_members`
-   `invitations`
-   `roles`
-   `permissions`
-   `role_permissions`
-   `user_role_assignments`
-   `scope_assignments`
-   `reporting_relationships`
-   `approval_authorities`

### Property

-   `properties`
-   `buildings`
-   `floors`
-   `units`
-   `unit_types`
-   `unit_attributes`
-   `unit_facilities`
-   `unit_occupancy_policies`
-   `unit_status_history`

### Tenant and lease

-   `tenants`
-   `tenant_contacts`
-   `tenant_identifications`
-   `tenant_documents`
-   `tenant_portal_links`
-   `leases`
-   `lease_tenants`
-   `lease_units`
-   `lease_occupants`
-   `lease_terms`
-   `lease_amendments`
-   `lease_documents`
-   `occupancy_records`

### Billing and payment

-   `charge_types`
-   `billing_schedules`
-   `invoices`
-   `invoice_items`
-   `payment_instructions`
-   `payment_submissions`
-   `payment_verifications`
-   `payments`
-   `payment_allocations`
-   `receipts`
-   `refunds`
-   `payment_reversals`

### Accounting

-   `chart_of_accounts`
-   `accounting_periods`
-   `journal_entries`
-   `journal_entry_lines`
-   `bank_accounts`
-   `bank_transactions`
-   `bank_reconciliations`
-   `vendors`
-   `expenses`
-   `expense_approvals`

### Operations and clearance

-   `maintenance_requests`
-   `work_orders`
-   `contractors`
-   `inspection_templates`
-   `inspection_template_items`
-   `inspections`
-   `inspection_results`
-   `move_out_requests`
-   `clearance_cases`
-   `clearance_approvals`
-   `deposit_settlements`
-   `handover_records`

### Platform support

-   `approval_policies`
-   `approval_requests`
-   `approval_actions`
-   `notifications`
-   `notification_deliveries`
-   `documents`
-   `audit_logs`
-   `activity_logs`
-   `outbox_events`
-   `idempotency_records`

## 24.2 Relationship rules

-   One organisation has many properties.
-   One property belongs to exactly one organisation.
-   One property may have zero or many buildings.
-   One building belongs to one property.
-   One building may have zero or many floors.
-   One floor belongs to one building.
-   A unit belongs to one property and may optionally belong to a
    building and floor.
-   A lease belongs to one organisation.
-   A lease may cover one or many units through `lease_units`.
-   A lease may include one or many tenants through `lease_tenants`.
-   A tenant may participate in many leases over time.
-   A lease may have multiple authorised occupants.
-   A unit may have many historical occupancy records but must obey
    active occupancy constraints.
-   One invoice has one or many invoice lines.
-   One payment may have multiple allocations.
-   One invoice may receive multiple payment allocations.
-   One journal entry has two or more journal lines.
-   A journal entry must balance.
-   One move-out request belongs to a lease and one or more related
    units.
-   One clearance case may have multiple inspections and approval
    actions.

## 24.3 Database integrity requirements

-   Use UUIDs or another non-guessable internal identifier strategy for
    externally exposed records.
-   Enforce organisation foreign-key consistency.
-   Enforce unique organisation email membership according to the
    confirmed rule.
-   Enforce property ownership by one organisation.
-   Enforce unit display-code uniqueness at the selected scope.
-   Prevent invalid cross-property building/floor/unit relationships.
-   Prevent invalid lease-to-unit organisation relationships.
-   Prevent prohibited overlapping active occupancy.
-   Enforce tenant and occupant limits.
-   Enforce balanced journal entries at posting time.
-   Enforce unique invoice generation keys.
-   Enforce payment posting idempotency.
-   Preserve immutable financial and audit history.
-   Use database transactions for multi-record financial operations.

------------------------------------------------------------------------

# 25. API and Integration Requirements

The final API specification should be organised around domain resources
and actions.

Example resource groups:

-   `/organisations`
-   `/users`
-   `/invitations`
-   `/roles`
-   `/assignments`
-   `/properties`
-   `/buildings`
-   `/floors`
-   `/units`
-   `/tenants`
-   `/leases`
-   `/invoices`
-   `/payment-instructions`
-   `/payment-submissions`
-   `/payments`
-   `/receipts`
-   `/journal-entries`
-   `/expenses`
-   `/maintenance-requests`
-   `/inspections`
-   `/move-out-requests`
-   `/clearance-cases`
-   `/approvals`
-   `/notifications`
-   `/reports`

These are illustrative resource names, not a final endpoint contract.

## 25.1 API requirements

-   Authenticated requests.
-   Organisation context derived from secure membership, not trusted
    blindly from client input.
-   Scope-aware authorisation.
-   Request validation.
-   Consistent error responses.
-   Pagination and filtering.
-   Sorting.
-   Idempotency for financial and scheduled operations.
-   API versioning.
-   Rate limiting.
-   Correlation IDs.
-   Audit integration.
-   No sensitive fields returned unless authorised.

## 25.2 External integrations

Initial integration adapters may include:

-   Email provider.
-   SMS provider.
-   File/object storage.
-   Optional mapping/geocoding service.
-   Optional bank statement import.
-   Optional central payment provider.

The core payment workflow must remain functional without direct
individual landlord payment API credentials.

------------------------------------------------------------------------

# 26. Non-Functional Requirements

## 26.1 Performance

-   Common dashboard and record views should load within an agreed
    performance budget under expected load.
-   Lists must use pagination.
-   Search must support organisation and property scope.
-   Reports should avoid unbounded queries.
-   Large exports should run asynchronously.
-   Indexes should support common foreign-key, date, status and
    organisation filters.

Exact performance targets should be confirmed through expected tenant
counts, property counts, concurrent users and infrastructure testing.

## 26.2 Availability and recovery

-   Define service availability targets before production launch.
-   Back up database and documents.
-   Test restoration procedures.
-   Define recovery time objective (RTO) and recovery point objective
    (RPO).
-   Monitor background jobs and failed notifications.
-   Provide safe retry handling for failed financial workflows.

## 26.3 Scalability

The design should support growth in:

-   Organisations.
-   Users.
-   Properties.
-   Buildings and units.
-   Tenant profiles.
-   Leases.
-   Invoices.
-   Payment submissions.
-   Journal entries.
-   Documents.
-   Notifications.

Partitioning, read replicas or service extraction should be considered
only when measurements justify them.

## 26.4 Usability

-   Responsive layouts for mobile and desktop.
-   Clear forms with field-level validation.
-   Save drafts where appropriate.
-   Searchable property, tenant and unit selectors.
-   Clear status labels and reasons.
-   Accessible colour contrast.
-   Do not rely on colour alone to communicate payment or clearance
    status.
-   Support low-bandwidth operational conditions where practical.

## 26.5 Internationalisation

-   Organisation country and property country are separate fields.
-   Support multiple currencies.
-   Preserve transaction currency and original amount.
-   Store exchange rates and conversion basis for foreign-currency
    accounting.
-   Store timestamps consistently and display in relevant local time
    zones.
-   Support country-specific identity document types.
-   Support different address formats.
-   Support configurable date, number and phone formats.
-   Do not hard-code Uganda-only assumptions into the core schema.

------------------------------------------------------------------------

# 27. Acceptance Criteria

The following are release-level acceptance conditions. More detailed
test cases should be written during implementation.

## 27.1 Organisation isolation

-   A user from Organisation A cannot retrieve Organisation B's
    properties, tenants, leases, payments or documents.
-   Changing an ID in a request does not bypass organisation isolation.
-   Organisation-owned records cannot be created without valid
    organisation context.
-   Cross-organisation access attempts are logged appropriately.

## 27.2 Email uniqueness

-   An email already associated with one property organisation cannot be
    invited into another organisation.
-   Registration and invitation flows return a clear validation error.
-   Database constraints prevent duplicate organisation membership for
    the same email identity.

## 27.3 Property management

-   A landlord can create residential, commercial and mixed-use
    properties.
-   A property can contain several buildings.
-   Buildings can contain custom-named floors.
-   Units can use landlord-defined display codes.
-   Standalone units can be created without a building or floor.
-   A property cannot belong to two organisations.

## 27.4 Tenant and lease management

-   A tenant can be associated with multiple units.
-   One lease can cover multiple units.
-   A lease can support multiple co-tenants.
-   Occupancy limits are validated.
-   Legal tenants are distinguishable from authorised occupants and
    representatives.
-   Each portal account is explicitly linked to the correct tenant
    profile.
-   Activation requires landlord or authorised manager approval.

## 27.5 Payments

-   Landlord can configure payment instructions at supported scopes.
-   Tenant sees the effective applicable instruction.
-   Tenant can submit payment details and supporting proof.
-   Submission remains pending until staff verification.
-   Screenshot upload alone does not post money.
-   Only authorised staff can verify.
-   Verified payment creates a balanced journal and receipt.
-   Partial payments and overpayments are handled correctly.
-   Duplicate transaction references are flagged or blocked according to
    policy.
-   Reversals preserve original history.

## 27.6 Accounting

-   Every posted journal balances.
-   Unbalanced journals cannot be posted.
-   Posted journals cannot be edited or deleted through ordinary
    operations.
-   Reversals create traceable entries.
-   Tenant statements reconcile to relevant ledger entries.
-   Property reporting uses property dimensions.
-   Security deposits are accounted for separately from rent income.
-   Financial period locks are enforced.

## 27.7 Employee access

-   Roles can be configured.
-   Assignments can be scoped.
-   Reporting relationships can be configured.
-   Reassignment changes current access without rewriting historical
    records.
-   A caretaker cannot access financial records without explicit
    permission.
-   An employee cannot approve an action beyond their authority.

## 27.8 Move-out and clearance

-   Tenant can submit a move-out request.
-   Caretaker can complete and submit an inspection.
-   Property manager can verify, reject or return it.
-   Finance can prepare a settlement.
-   Landlord or delegated authority can give final approval.
-   All steps retain actor, time and decision history.
-   Disputed financial items remain distinguishable from agreed
    deductions.
-   Lease termination and financial clearance are not improperly
    conflated.

------------------------------------------------------------------------

# 28. Suggested Product Release Plan

## Release 1: Platform and property foundation

-   Organisation registration.
-   Authentication and one-email-per-organisation rule.
-   Organisation isolation.
-   Roles and permissions.
-   Employee invitations and assignments.
-   Property, building, floor and unit management.
-   Custom unit identifiers.
-   Residential, commercial and mixed-use configuration.

## Release 2: Tenants and leases

-   Tenant profiles.
-   Company tenants and representatives.
-   Co-tenants and occupancy limits.
-   Multiple units per lease.
-   Tenant invitations and activation.
-   Lease documents and amendments.
-   Unit occupancy lifecycle.

## Release 3: Billing and accounting

-   Charge types.
-   Billing schedules.
-   Invoice generation.
-   Payment instructions.
-   Tenant payment submissions.
-   Staff verification.
-   Payment allocation.
-   Receipts.
-   Chart of accounts.
-   Double-entry journals.
-   Tenant statements.
-   Core financial reports.

## Release 4: Property operations

-   Maintenance requests.
-   Work orders.
-   Expense requests and approvals.
-   Inspection templates.
-   Caretaker workflows.
-   Move-out requests.
-   Clearance and deposit settlement.

## Release 5: Communications and analytics

-   SMS and email notifications.
-   Approval reminders.
-   Occupancy and arrears dashboards.
-   Property performance.
-   Staff activity reporting.
-   Bank statement imports and reconciliation.
-   Advanced exports.

## Release 6: Expansion

-   Advanced commercial lease structures.
-   Service charge reconciliation.
-   Utility metering and allocations.
-   Multi-country operational settings.
-   Optional integrations.
-   Enterprise reporting.
-   Advanced analytics and forecasting.

The release plan may be adjusted after technical estimation, but
organisation isolation, financial integrity and access control must not
be postponed.

------------------------------------------------------------------------

# 29. Risks and Design Considerations

  -----------------------------------------------------------------------
  Risk                    Consequence             Required mitigation
  ----------------------- ----------------------- -----------------------
  Weak organisation       Exposure of one         Enforce scope in
  isolation               landlord's data to      backend, database and
                          another                 tests

  Overly rigid property   Cannot represent real   Support standalone
  hierarchy               buildings               units and configurable
                                                  floors

  Confusing tenant and    Incorrect access or     Separate tenant profile
  account records         duplicated people       from login identity

  Poor co-tenancy model   Incorrect occupancy and Separate lease tenants,
                          financial               occupants and
                          responsibility          representatives

  Duplicate billing       Incorrect tenant        Idempotent billing
                          balances                schedule processing

  False payment           Incorrect accounts and  Staff verification
  confirmation            receipts                before posting

  Mutable ledger          Loss of financial trust Immutable posted
                                                  journals and reversals

  Unclear delegated       Unauthorised approvals  Configurable approval
  authority                                       policy and audit

  Incomplete move-out     Deposit disputes        Versioned checklists,
  evidence                                        evidence and review
                                                  history

  One-email restriction   Employee onboarding     Clear validation and
                          friction                controlled transfer
                                                  process

  Uncontrolled document   Privacy and identity    Private storage,
  access                  theft risk              encryption and scoped
                                                  access

  Premature microservices High operational        Modular monolith
                          complexity              initially
  -----------------------------------------------------------------------

------------------------------------------------------------------------

# 30. Open Questions for Detailed Design

These questions do not invalidate the confirmed architecture. They must
be settled during detailed product, legal, accounting and technical
design.

1.  **Organisation verification:** What documents or verification
    process are required before a landlord organisation can operate?
2.  **Organisation transfer:** What happens when an employee changes
    organisations, given the one-email-per-organisation rule?
3.  **Tenant email rule:** Should the one-email restriction apply only
    to staff identities or to all portal identities, including tenants?
    The current product rule states one email per property organisation,
    so this needs an explicit scope decision before identity schema
    finalisation.
4.  **Joint liability:** Should joint co-tenants be jointly liable by
    default, or should each lease explicitly choose joint liability or
    allocated shares?
5.  **Occupant access:** Which information can authorised occupants view
    when they are not legal tenants?
6.  **Cash payments:** Should cash paid at an organisation office be
    supported in the first release, and what receipt controls are
    required?
7.  **Tax:** Which tax rules, withholding requirements and statutory
    reports apply in each launch country?
8.  **Accounting basis:** Should each organisation choose cash or
    accrual reporting, while the system maintains a consistent
    double-entry ledger?
9.  **Foreign currency:** Can one organisation maintain leases in
    multiple currencies, and how should realised and unrealised exchange
    differences be handled?
10. **Approval separation:** Which actions require the requester and
    approver to be different people?
11. **Tenant account transfer:** What happens when a tenant changes
    email or a lease is transferred to another tenant?
12. **Retention:** What are the required retention periods for leases,
    financial records, identification documents and inspection evidence?

These should be resolved before finalising the production database
schema and accounting policy.

------------------------------------------------------------------------

# 31. Glossary

  -----------------------------------------------------------------------
  Term                                Definition
  ----------------------------------- -----------------------------------
  AMAALPROP                           The property management SaaS
                                      platform

  Organisation                        An independent landlord or property
                                      management entity using AMAALPROP

  SaaS tenancy                        The separation of each
                                      organisation's data and access on
                                      the shared platform

  Real-estate tenant                  A person or organisation renting a
                                      property or unit

  Property                            A real estate asset managed by an
                                      organisation

  Building                            A physical structure within a
                                      property

  Floor / level                       A defined level within a building

  Unit                                A rentable premise, such as a
                                      house, apartment, shop or office

  Tenant profile                      Record of a person or company
                                      involved in a tenancy

  Portal account                      Login identity used to access
                                      AMAALPROP

  Lease                               Agreement connecting tenants,
                                      units, dates and contractual terms

  Co-tenant                           A legal tenant sharing a lease with
                                      another tenant

  Occupant                            A person permitted to occupy a unit
                                      who may not be a contractual tenant

  Payment instruction                 Landlord-provided details telling
                                      tenants where and how to pay

  Payment submission                  Tenant's declaration that an
                                      external payment has been made

  Payment verification                Staff confirmation that the money
                                      was actually received

  Payment allocation                  Application of a verified payment
                                      to one or more charges

  Journal entry                       A balanced accounting transaction

  Ledger                              The permanent accounting record of
                                      posted transactions

  Security deposit                    Money held under the lease and
                                      accounted for according to its
                                      liability and settlement treatment

  Scope assignment                    The property or organisational area
                                      within which an employee has
                                      authority

  Approval policy                     Rules defining who must approve a
                                      particular action

  Clearance case                      The coordinated move-out,
                                      inspection, financial settlement
                                      and handover process
  -----------------------------------------------------------------------

------------------------------------------------------------------------

# 32. Final Product Definition

AMAALPROP is a multi-organisation property management and accounting
SaaS platform. It enables each landlord organisation to manage its
properties, buildings, floors, rentable units, tenants, leases, billing,
payment verification, double-entry accounting, employees, maintenance,
inspections, approvals and tenant exits through one controlled system.

Its architecture is built around strict organisation isolation,
configurable property structures, flexible tenancy relationships, scoped
employee authority, externally paid and staff-verified transactions, an
auditable double-entry ledger and a structured tenant clearance process.

The product must represent the way real properties and tenancy
agreements operate rather than forcing landlords into a simplistic
property-and-tenant register. It must also preserve the integrity of
financial records and make every important operational decision
traceable.

**This PRD is the product requirements baseline for the next design
stages:** 1. Detailed user journeys and screen specifications. 2. Entity
Relationship Diagram (ERD). 3. Database schema and constraints. 4. API
specification. 5. Permission matrix. 6. Workflow/state-transition
specifications. 7. Technical implementation plan. 8. Test plan and
release criteria.
