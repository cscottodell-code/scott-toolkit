# Advosy Business Context

## Metadata
- Last updated: 2026-02-25
- Version: 1.0

## About Advosy

Advosy is a home services company based in Arizona. Scott is the Head of Sales Training
and Development. The company has multiple subsidiaries and brands.

## Key People

| Person | Role | Relevance to Projects |
|--------|------|----------------------|
| **Scott O'Dell** | Head of Sales Training & Development | AI orchestrator, builds prototypes with Claude Code |
| **Brett** | Co-developer on Bresco | Vibe coder, may use the toolkit in the future |
| **Gary** | Production developer | Productionizes Scott's prototypes. Suggested the current tech stack. |

## Subsidiaries & Brands

| Name | Abbreviation | What They Do |
|------|-------------|-------------|
| Claimsforce | CF | Claims management CRM (built on EspoCRM) |
| Select Adjusters | SA | Insurance adjusting services |
| Everest Air | - | HVAC services |
| Bloque | - | Masonry/hardscape services |
| Pestkee | - | Pest control services |
| Southwest Packout | - | Contents restoration/packout |
| Homeforce | - | General home services |
| VRZA | - | Property management |

## Project Context for Advosy Work

When building tools for Advosy:
- **Users** are typically internal employees (adjusters, sales reps, managers)
- **Data** is often related to claims, customers, scheduling, or training
- **Claimsforce (EspoCRM)** is the existing CRM — new tools may need to integrate with it
- **Gary** will productionize anything that goes live — keep handoff in mind
- **n8n workflows** exist for Advosy operations — check Brett's instance for existing automations

## Claimsforce (CF) Integration Notes

- Built on EspoCRM platform
- URL: sa.claimsforce.net
- Has its own workflow system, placeholder syntax, and webhook capabilities
- When building features that touch claims data, check if Claimsforce already
  has the data or if you need to sync between systems
- See the /claimsforce skill for detailed patterns

## Advosy Communication

- Projects for Advosy should be professional and business-appropriate
- UI should feel corporate but approachable — not startup-flashy
- Data accuracy is critical for insurance and claims work
- Gary expects clean, well-documented code for productionization
