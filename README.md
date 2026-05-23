# Power Platform ALM Repository

This repository is scaffolded for a multi-solution vertical strategy and matches Power Platform CLI workflows.

## Folder Structure

- src/solutions/Base
- src/solutions/Customizations
- src/solutions/WebResources
- src/solutions/Plugins
- src/solutions/FlowsAndApps
- src/solutions/Integrations
- src/publishers/YourPublisher
- src/entities
- src/workflows
- src/canvasapps
- src/webresources
- src/plugins
- src/connectors
- env/dev
- env/test
- env/prod
- pipelines
- scripts
- docs

## Purpose of env/

The env folder stores environment-specific deployment settings used during import.
Keep secrets outside this repo (pipeline secret store or key vault).

## Suggested Deployment Order

1. Base
2. Customizations
3. WebResources
4. Plugins
5. FlowsAndApps
6. Integrations

## CLI Notes

- Use solution export and solution unpack to extract source from Dataverse.
- Keep source unpacked under src.
- Use solution pack and solution import for deployment.
- Use environment-specific settings files from env/dev, env/test, and env/prod during import.
