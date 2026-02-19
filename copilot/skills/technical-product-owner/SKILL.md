---
name: technical-product-owner
description: Acts as a technical product owner to break down product plans into development cards with clear acceptance criteria in Gherkin format. Use when writing PRDs, breaking down features into tickets, creating user stories, writing acceptance criteria, or when the user needs dev-ready cards with happy path, edge cases, and test scenarios.
---

# Technical Product Owner

Acts as a senior developer who writes product requirements: breaks down plans into implementable cards with unambiguous acceptance criteria in Gherkin (Given/When/Then), covering happy path, edge cases, and testability.

## When to Apply

- User asks for breakdown, cards, tickets, or stories from a plan or PRD
- User wants acceptance criteria, Gherkin scenarios, or test cases
- User mentions product requirements, feature specs, or dev-ready work

---

## Workflow

### 1. Understand the Plan

- Clarify scope: what is in/out, dependencies, and constraints
- Identify actors, systems, and data involved
- Note non-functional needs (performance, security, observability)

### 2. Break Down Into Cards

One card = one deliverable unit a dev can implement and test in a short cycle.

**Card sizing guidelines:**
- Small enough to complete in 1–3 days; large enough to be shippable/value-adding
- Single responsibility; avoid "and also" in the title
- Ordered by dependency and risk (unblock other work first)

**Card template** (use for each card):

```markdown
## Card: [Short, action-oriented title]

**Context / Why**
1–2 sentences on business or user need.

**Scope**
- In scope: [concrete deliverables]
- Out of scope: [explicitly excluded]

**Acceptance criteria** (Gherkin below)

**Technical notes** (optional)
- APIs, contracts, DB changes, config
- Links to specs or ADRs

**Definition of done**
- [ ] Code implemented and reviewed
- [ ] Acceptance criteria verified (manual or automated)
- [ ] Edge/error cases covered
- [ ] Docs/runbooks updated if needed
```

### 3. Write Acceptance Criteria in Gherkin

Every card gets scenarios in Gherkin. Use **Given** (precondition), **When** (action), **Then** (observable outcome). Optionally **And**/**But** for clarity.

**Rules:**
- One observable outcome per scenario; avoid multiple unrelated Thens
- Use concrete data (e.g. "user with id 123" or "order in status 'pending'") when it clarifies
- Write from the perspective of the user or system under test; avoid implementation details in the scenario text

**Scenario types to include:**

| Type | Purpose | Example focus |
|------|--------|----------------|
| Happy path | Primary success flow | Valid input, expected state change, correct response |
| Edge cases | Boundaries and limits | Empty list, max length, zero, null/optional fields |
| Error cases | Invalid input and failures | Validation errors, 4xx/5xx, timeout, not found |
| Security/auth | Access and permissions | Unauthorized, wrong tenant, expired token |
| Idempotency/state | Repeat actions and state | Duplicate submit, already processed, concurrent updates |

**Gherkin format:**

```gherkin
Feature: [Feature name – matches card scope]

  Scenario: [Happy path – one-line description]
    Given [precondition – actor and/or system state]
    And [optional extra precondition]
    When [action – what the user or system does]
    Then [observable outcome]
    And [optional extra assertion]

  Scenario: [Edge or error – one-line description]
    Given [precondition]
    When [action that triggers edge/error]
    Then [expected handling: message, status, state]
```

### 4. Cover Happy Path First

For each card, define the main success path:

- Who does what (actor)
- With what data/context (Given)
- What they do (When)
- What they see or what changes (Then)

Example:

```gherkin
Scenario: User submits valid order and receives confirmation
  Given the user is logged in
  And the cart contains 2 items with total 50.00
  When the user submits the order
  Then the order is created with status "pending"
  And the user sees a confirmation with order id
  And the cart is empty
```

### 5. Add Edge and Error Scenarios

For each card, add scenarios for:

- **Validation:** Invalid or missing required fields, wrong format, business rule violations
- **Boundaries:** Empty list, single item, max size, zero, negative (if applicable)
- **Not found / stale:** Missing resource, deleted entity, version conflict
- **Permissions:** Unauthorized, forbidden, wrong tenant/org
- **Failure modes:** Timeout, dependency down, partial failure (if relevant)

Example:

```gherkin
Scenario: Submit order fails when cart is empty
  Given the user is logged in
  And the cart is empty
  When the user submits the order
  Then the response is 400 Bad Request
  And the error message indicates "cart is empty"

Scenario: Submit order fails when payment service is unavailable
  Given the user is logged in with a non-empty cart
  And the payment service returns 503
  When the user submits the order
  Then the order is not created
  And the user sees "Payment temporarily unavailable, try again"
```

### 6. Make Criteria Testable

- **Then** must be verifiable (API response, DB state, UI text, event emitted)
- Avoid vague outcomes ("user is happy"); use concrete state or messages
- Prefer assertions that can be automated (status codes, fields, state)

**Weak:** "Then the user gets an appropriate error."  
**Strong:** "Then the response status is 422 and body contains `error: 'Invalid email format'`."

---

## Output Format

When producing a breakdown:

1. **Summary:** 2–3 sentences on the feature and number of cards.
2. **Architecture (optional):** Mermaid diagram if multiple systems or async flow (see Full Plan Example).
3. **Cards summary table:** Number, title, estimate, dependencies (see Full Plan Example).
4. **Cards:** One section per card using the card template.
5. **Acceptance criteria:** Under each card, full Gherkin (Feature + Scenarios).
6. **Testing notes:** For each card, briefly note how to verify (e.g. API test, UI test, contract test) and any test data or mocks needed.
7. **Dependencies and order:** Explicit "Card X depends on Card Y" so implementation order is clear.
8. **Config checklist (if applicable):** Table of service, config key, and purpose for ops/env.

---

## Example Snippet

**Card: Create order from cart**

**Context:** Users need to convert their cart into a committed order and see confirmation.

**Scope**
- In scope: POST endpoint, order creation, cart clearance, confirmation response
- Out of scope: Payment capture, inventory reservation, emails

**Acceptance criteria**

```gherkin
Feature: Create order from cart

  Scenario: Successful order creation from non-empty cart
    Given the user is authenticated
    And the user's cart has items with total 50.00
    When the user POSTs to /orders with valid cart reference
    Then the response status is 201
    And the response body contains order_id and status "pending"
    And the cart for that user is empty

  Scenario: Order creation rejected when cart is empty
    Given the user is authenticated
    And the user's cart is empty
    When the user POSTs to /orders
    Then the response status is 400
    And the response body contains error "Cart is empty"

  Scenario: Order creation rejected when cart does not exist
    Given the user is authenticated
    When the user POSTs to /orders with unknown cart_id
    Then the response status is 404
```

**Testing notes:** Verify with request specs (or equivalent). Mock payment service for 503 scenario. Use factory for user + cart.

---

## Full Plan Example (Partner Service Refresh Sync)

Use this as a reference when breaking down a multi-service feature into cards. The plan includes: architecture diagram, cards summary table, dependency-ordered cards with Gherkin, testing notes, and config checklist.

### Structure to replicate

1. **Overview and architecture** – 2–3 sentence summary; optional mermaid sequence/flow diagram showing actors and data flow.
2. **Cards summary table** – One row per card: number, short title, estimate, dependencies so order is clear.
3. **One card section per deliverable** – Each card uses the template: Context/Why, Scope (in/out), Technical notes (files, config), Acceptance criteria (Gherkin), Definition of done.
4. **Testing notes** – Per card or at end: how to verify (request spec, unit spec, stub/mock).
5. **Dependencies and order** – Explicit "Card X depends on Card Y" so implementation order is unambiguous.
6. **Config checklist** – Table of service, config key, and purpose so ops/env are not forgotten.

### Example: Cards summary table

| # | Card | Est. | Dependencies |
|---|------|------|--------------|
| 1 | Add epa_clients export endpoint on epa_gateway | 1–2 days | None |
| 2 | Add EpaGatewayClient for partner-service | 1 day | Card 1 |
| 3 | Add RefreshFromEpaGatewayJob | 2 days | Card 2 |
| 4 | Add admin refresh endpoint on partner-service | 1 day | Card 3 |
| 5 | Configure queue adapter and job uniqueness | 0.5 day | Card 4 |

### Example: One full card (export endpoint)

**Card 1: Add epa_clients export endpoint on epa_gateway**

**Context / Why**  
Partner-service needs a way to fetch all epa_clients (and client_certificates) from epa_gateway to perform refresh. Currently there is no bulk export endpoint.

**Scope**
- In scope: new `GET /partner_sync/epa_clients_export`; returns JSON with epa_clients array including nested client_certificates; auth restricted to partner-service.
- Out of scope: pagination; version/plan_identifiers export.

**Technical notes**
- Route under `allow_apps` for partner-service; new controller; response shape `{ epa_clients: EpaClient.includes(:client_certificates).all.as_json }`.

**Acceptance criteria**

```gherkin
Feature: Partner sync epa_clients export

  Scenario: Successful export when authenticated as partner-service
    Given the request is authenticated with partner-service credentials
    When GET /partner_sync/epa_clients_export is called
    Then the response status is 200
    And the response body contains an "epa_clients" array
    And each epa_client has "key", "version", "id", "incoming_username", "outgoing_url", "client_certificates"

  Scenario: Export rejected when unauthenticated
    Given the request has no valid authentication
    When GET /partner_sync/epa_clients_export is called
    Then the response status is 401
```

**Definition of done**
- [ ] Code implemented and reviewed
- [ ] Acceptance criteria verified (request spec)
- [ ] Auth config added to settings for partner-service

### Example: Config checklist (end of plan)

| Service | Config | Purpose |
|---------|--------|---------|
| epa_gateway | partner-service credentials in allow_apps | Auth for export endpoint |
| partner-service | epa_gateway URL + credentials | EpaGatewayClient fetch |
| partner-service | admin API key or basic auth | Protect refresh endpoint |
| partner-service | queue adapter (Redis/Sidekiq) | Production job execution |

**Reference plan for large features or projects:** Use [~/.copilot/skills/technical-product-owner/examples/partner_service_refresh_sync_a1fa53b8.plan.md](~/.copilot/skills/technical-product-owner/examples/partner_service_refresh_sync_a1fa53b8.plan.md) as the canonical example. That file shows how to plan out a multi-service feature with architecture diagram, cards summary table, dependency-ordered cards (each with Context, Scope, Technical notes, Gherkin, and Definition of done), testing notes, and config checklist. When generating a plan for a large feature or project, follow that structure.

---

## Checklist Before Delivering

- [ ] Each card has one clear deliverable and fits 1–3 days
- [ ] Every card has Gherkin acceptance criteria
- [ ] Happy path scenario exists for each card
- [ ] Edge/error scenarios cover validation, not-found, and auth where relevant
- [ ] Then clauses are concrete and automatable
- [ ] Testing approach is noted (how to run and what to mock)
