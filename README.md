# Studio Catalog

Public catalog for Self-Host Studio. Source of truth for documentation, provider packages, and workflow templates.

This repository is downloaded during Studio bootstrap and serves content to every Studio deployment.

## What's Here

```
studio-catalog/
├── docs/                        # User-facing documentation
│   ├── user.md                  # End-user guide
│   ├── admin.md                 # Organization admin guide
│   ├── super-admin.md           # Deployment and infrastructure guide
│   ├── manifest.json            # Doc registry (API reads this)
│   └── providers/               # Per-provider documentation
│       ├── index.md             # Provider catalog listing
│       └── {provider}.md        # Individual provider guides
├── packages/                    # Provider packages (basic tier)
│   └── {provider}-{version}.zip
├── templates/                   # Workflow templates (basic tier)
│   └── {template}.json
├── marketplace-catalog.json     # Package registry (all tiers)
├── templates-catalog.json       # Template registry (all tiers)
└── LICENSE                      # Marketplace Package License
```

## Tiers

| Tier | Where | Access |
|------|-------|--------|
| **Basic** | This repo (`docs/`, `packages/`, `templates/`) | Public, no auth |
| **Advanced** | `studio-marketplace` (private repo) | Requires marketplace token |

Both tiers are listed in the catalog JSON files. Basic content is downloaded directly from this repo. Advanced content URLs point to `studio-marketplace` releases.

## How Studio Uses This

1. **Bootstrap**: Studio downloads catalog files and docs from this repo
2. **API**: Serves docs at `/api/v1/docs/{id}/content` using `docs/manifest.json`
3. **Marketplace**: UI reads `marketplace-catalog.json` and `templates-catalog.json` to show available packages and templates
4. **Install**: When a user installs a package, Studio downloads the zip from the URL in the catalog

## Configuration

Point Studio at this catalog:

```
MARKETPLACE_CATALOG_URL=https://raw.githubusercontent.com/selfhosthub/studio-catalog/main/marketplace-catalog.json
TEMPLATES_CATALOG_URL=https://raw.githubusercontent.com/selfhosthub/studio-catalog/main/templates-catalog.json
```

## How It Works

Studio uses a two-tier catalog system. This repo is the public hub.

### Catalog Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                    CATALOG FILES (this repo)                    │
│                                                                 │
│   marketplace-catalog.json        templates-catalog.json        │
│   ┌─────────────────────┐         ┌──────────────────────┐      │
│   │ core        (basic) │         │ welcome-webhook      │      │
│   │ leonardo-ai (adv.)  │         │ httpbin-echo-flow    │      │
│   │ ...                 │         │ leonardo-pipeline    │      │
│   └─────────┬───────────┘         └──────────┬───────────┘      │
└─────────────┼────────────────────────────────┼──────────────────┘
              │                                │
              ▼                                ▼
┌──────────────────────────┐    ┌──────────────────────────────┐
│     MARKETPLACE UI       │    │      TEMPLATE LIBRARY        │
│                          │    │                              │
│  Browse & install        │    │  Browse & create workflows   │
│  provider packages       │    │  from templates              │
└────────────┬─────────────┘    └──────────────┬───────────────┘
             │                                 │
             ▼                                 ▼
┌──────────────────────────────────────────────────────────────────┐
│                       DOWNLOAD SOURCES                           │
│                                                                  │
│  studio-catalog (this repo)       studio-marketplace (private)   │
│  ┌────────────────────────┐       ┌────────────────────────────┐ │
│  │                        │       │                            │ │
│  │  packages/             │       │  packages/                 │ │
│  │    core-1.1.0.zip      │       │    leonardo-ai-1.1.0.zip   │ │
│  │  templates/            │       │  templates/                │ │
│  │    welcome-webhook     │       │    leonardo-pipeline       │ │
│  │    httpbin-echo-flow   │       │                            │ │
│  │                        │       │  Requires MARKETPLACE_TOKEN│ │
│  │  FREE -- no auth       │       │  Requires auth             │ │
│  └────────────────────────┘       └────────────────────────────┘ │
└──────────────────────────────────────────────────────────────────┘
```

### Packages → Templates → Workflows

```
  PACKAGE (Provider)          TEMPLATE                WORKFLOW
  ┌──────────────┐        ┌──────────────┐        ┌──────────────┐
  │ core         │        │ welcome-     │        │ My Welcome   │
  │              │ used   │ webhook      │creates │ Hook         │
  │ Services:    │-------▶│              │-------▶│              │--▶ Instance 1
  │  - HTTP POST │   by   │ requires:    │        │ Configured   │--▶ Instance 2
  │  - Set Fields│        │   [core]     │        │ per org      │--▶ Instance 3
  │  - Echo      │        │              │        │              │
  └──────────────┘        └──────────────┘        └──────────────┘
```

**Package** — provides services.

**Template** — wires services into a reusable blueprint.

**Workflow** — a configured copy of a template, scoped to an org.

**Instance** — a single run of a workflow.

## Status

| Content | Status |
|---------|--------|
| Documentation (docs/) | Ready |
| Provider packages | In progress — basic/advanced tier definitions not finalized |
| Workflow templates | In progress — basic/advanced tier definitions not finalized |
| Catalog JSON files | In progress — schema stable, content expanding |

## License

[Marketplace Package License](LICENSE) — use and modify freely on your deployment, redistribution prohibited.
