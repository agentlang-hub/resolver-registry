# Agentlang Resolver Registry

A public registry of resolver metadata for the [Agentlang](https://github.com/agentlang-ai/agentlang) platform. Resolvers are components that route entity CRUD operations through JavaScript functions that call external APIs (GitHub, Stripe, Slack, etc.) instead of default database storage. This registry allows tools like [AgentCraft](https://github.com/agentlang-ai/agentcraft) to discover and integrate resolvers automatically.

## Directory Structure

```
contents/
  registry/
    github.json
    stripe.json
    slack.json
    ...
```

Each JSON file in `contents/registry/` describes one resolver.

## How It Works

The registry is hosted on GitHub and consumed via the GitHub Contents API:

```
GET https://api.github.com/repos/agentlang-hub/resolver-registry/contents/registry/
```

This returns a directory listing of all `.json` files. Each file can then be fetched individually by its `download_url` to retrieve the full resolver metadata.

### Custom Registry URL

AgentCraft users can override the default registry URL by setting the `CRAFT_REGISTRY_URL` environment variable to point to an alternative GitHub Contents API endpoint.

## Consumer Workflows

### Search

Search the registry for resolvers matching a query (e.g., "github issues"):

1. Fetch the directory listing from the registry API
2. Filter files by service prefix (first word of the query)
3. Fetch each candidate entry and score it against query keywords
4. Return results sorted by relevance score

Scoring weights:
- **Service name match:** 3 points per keyword
- **Tag match:** 2 points per keyword
- **Description match:** 1 point per keyword

### Integration

Once a resolver is selected, consumers can:

1. Use the `repo` field to clone the resolver source repository
2. Read the `.al` and `.js` files from the cloned resolver directory
3. Use `entities` metadata to understand what data types the resolver provides
4. Use `auth.envVars` to know which environment variables need to be configured

### Pipeline Integration

AgentCraft's `craft new` pipeline uses the registry during app generation:

1. After the API specification step, the pipeline scans requirements for external service references
2. If detected, it offers to search the registry for a matching resolver
3. The selected resolver's entities and metadata are injected into subsequent code generation steps
4. The final assembly includes the resolver `.al` and `.js` files alongside the generated application code

## JSON Format Specification

Each registry entry is a JSON file conforming to the following schema:

### Top-Level Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | yes | Unique identifier for the resolver (e.g., `"github"`, `"stripe"`) |
| `service` | string | yes | Name of the external service this resolver integrates with |
| `description` | string | yes | Human-readable summary of the resolver's capabilities |
| `tags` | string[] | yes | Keywords for search and categorization |
| `entities` | Entity[] | yes | List of entities provided by the resolver |
| `auth` | Auth | yes | Authentication requirements |
| `repo` | string | yes | GitHub repository slug containing the resolver source (e.g., `"agentlang-hub/resolvers"`) |
| `version` | string | yes | Semantic version of the resolver |

### Entity Object

Each entry in the `entities` array describes one data type managed by the resolver:

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | yes | Entity name as defined in the `.al` file (e.g., `"Issue"`, `"Customer"`) |
| `operations` | string[] | yes | Supported CRUD operations: `"create"`, `"query"`, `"update"`, `"delete"` |
| `attributes` | string[] | yes | List of attribute names defined on the entity |

### Auth Object

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `envVars` | string[] | yes | Environment variable names required for authentication (e.g., `["GITHUB_ACCESS_TOKEN"]`) |
| `scheme` | string | yes | Authentication scheme used by the resolver: `"bearer"`, `"basic"`, `"api-key"`, or `"custom"` |

### Example

```json
{
  "name": "github",
  "service": "github",
  "description": "Manage GitHub issues, repositories, files, organizations, and users via the GitHub REST API",
  "tags": ["github", "git", "issues", "repositories", "pull-requests", "code", "version-control"],
  "entities": [
    {
      "name": "Issue",
      "operations": ["create", "query", "update", "delete"],
      "attributes": ["id", "owner", "repo", "issue_number", "title", "author", "state", "body", "date_created"]
    },
    {
      "name": "Repository",
      "operations": ["create", "query", "update", "delete"],
      "attributes": ["id", "owner", "name", "full_name", "description", "url", "date_created"]
    }
  ],
  "auth": {
    "envVars": ["GITHUB_ACCESS_TOKEN"],
    "scheme": "bearer"
  },
  "repo": "agentlang-hub/resolvers",
  "version": "0.0.1"
}
```

## Available Resolvers

| Resolver | Service | Entities | Auth |
|----------|---------|----------|------|
| airtable | Airtable | Base, Table, Record, Field | `AIRTABLE_API_KEY` |
| box | Box | File, Folder, SharedLink, User | `BOX_ACCESS_TOKEN` |
| expensify | Expensify | Expense, Report, Policy, Employee | `EXPENSIFY_PARTNER_USER_ID`, `EXPENSIFY_PARTNER_USER_SECRET` |
| freshdesk | Freshdesk | Ticket, Contact, Company, Agent | `FRESHDESK_API_KEY`, `FRESHDESK_DOMAIN` |
| github | GitHub | Issue, Repository, File, Organization, User | `GITHUB_ACCESS_TOKEN` |
| gmail | Gmail | Message, Thread, Label, Draft | `GMAIL_ACCESS_TOKEN` |
| google-drive | Google Drive | File, Folder, Permission, Revision | `GOOGLE_DRIVE_ACCESS_TOKEN` |
| hubspot | HubSpot | Contact, Company, Deal, Ticket, Engagement | `HUBSPOT_ACCESS_TOKEN` |
| infoblox | Infoblox | Host, Network, Zone, Record, Lease | `INFOBLOX_HOST`, `INFOBLOX_USERNAME`, `INFOBLOX_PASSWORD` |
| jira | Jira | Issue, Project, Board, Sprint | `JIRA_HOST`, `JIRA_EMAIL`, `JIRA_API_TOKEN` |
| salesforce | Salesforce | Account, Contact, Lead, Opportunity, Case, Task | `SALESFORCE_INSTANCE_URL`, `SALESFORCE_ACCESS_TOKEN` |
| servicenow | ServiceNow | Incident, Problem, ChangeRequest, User, ConfigurationItem | `SERVICENOW_INSTANCE`, `SERVICENOW_USERNAME`, `SERVICENOW_PASSWORD` |
| stripe | Stripe | Customer, Product, Price, Subscription, Invoice, PaymentIntent, Charge, Refund, Payout, InvoiceItem | `STRIPE_API_KEY` |
| teams | Microsoft Teams | Team, Channel, Message, Member | `TEAMS_ACCESS_TOKEN` |
| zendesk | Zendesk | Ticket, User, Organization, Group | `ZENDESK_SUBDOMAIN`, `ZENDESK_EMAIL`, `ZENDESK_API_TOKEN` |
| zohocrm | Zoho CRM | Lead, Contact, Account, Deal, Task | `ZOHO_CRM_ACCESS_TOKEN` |
| zohoexpense | Zoho Expense | Expense, Report, Trip, Currency, User | `ZOHO_EXPENSE_ACCESS_TOKEN`, `ZOHO_EXPENSE_ORG_ID` |
| zoom | Zoom | Meeting, User, Webinar, Recording | `ZOOM_ACCESS_TOKEN` |

## Contributing

To add a new resolver to the registry:

1. Create a JSON file in `contents/registry/` named `<service>.json`
2. Follow the JSON format specification above
3. Ensure all fields are populated accurately from the resolver's `.al` and `.js` source files
4. Submit a pull request
