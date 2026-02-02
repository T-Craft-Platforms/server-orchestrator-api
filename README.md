# Server Orchestrator API

This repository contains the OpenAPI specification for the Server Orchestrator API and handles the automatic generation of server stubs and client SDKs.

## Repo Structure

- `.github/workflows/ci-cd.yml`: GitHub Actions for validation and release.
- `openapi/openapi.yaml`: The single source of truth for the API definition.
- `pom.xml`: Root Maven configuration for orchestration and code generation.
- `target/`: Output directory for generated code.

## Local Development

### Prerequisites

- Java 17+
- Maven 3.8+

### Generation Flow

The generator is driven by Maven profiles. Pick the outputs you need:

```bash
mvn clean compile -P backend-spring,backend-jaxrs,frontend-fetch,frontend-axios
```

This will populate the `generated/` directory with:
- `spring-boot-server`: Spring Boot 3 code.
- `jaxrs-server`: Jakarta EE (JAX-RS) code.
- `typescript-fetch-client`: TypeScript client using fetch API.
- `typescript-axios-client`: TypeScript client using axios.

To generate a single target, use just one profile:

```bash
mvn clean compile -P backend-spring
```

The version used for local builds is always `VERSION`.

### Generator Properties

All generator behavior is driven by Maven properties in `pom.xml`. You can override any property at runtime using `-D`, for example:

```bash
mvn clean compile -P backend-spring -Dgenerator.backend.interfaceOnly=true
```

#### Key properties

| Property                                 | Description                                             |
|------------------------------------------|---------------------------------------------------------|
| revision                                 | Version used for local builds and npm package versions. |
| generator.version                        | OpenAPI Generator version.                              |
| generator.inputSpec                      | Path to the OpenAPI spec file.                          |
| generator.baseOutputDir                  | Base output directory for generated code.               |
| generator.backend.interfaceOnly          | Generate interfaces only for backend targets.           |
| generator.backend.delegatePattern        | Use delegate pattern for backend targets.               |
| generator.backend.basePackage            | Base Java package for backend targets.                  |
| generator.frontend.basePackage           | Base npm package name for frontend targets.             |
| generator.frontend.supportsES6           | Enable ES6 support for frontend targets.                |
| generator.frontend.withInterfaces        | Generate TypeScript interfaces.                         |
| generator.frontend.generateDocumentation | Toggle API/model documentation generation.              |

## Validation

The OpenAPI specification is validated on every push and pull request that affects the `openapi/` directory or generator configurations.

## CI and Release Behavior

### Continuous Integration
- Every push to `main` or pull request triggers a validation job.
- Only runs when relevant files (`openapi/**`, `pom.xml`) are modified.

### Release
- Triggered by pushing a tag (e.g., `v1.2.3`).
- Automatic versioning:
    - Strips `v` prefix from the tag to determine the version.
    - Updates Maven `revision` property and npm package versions.
- Artifacts published to GitHub Packages:
    - Maven artifacts (pom only, as this is a generator repo).
    - npm packages for both fetch and axios clients.

## Adding Endpoints

1. Edit `openapi/openapi.yaml`.
2. Run the generator for the targets you need (see "Generation Flow").
3. Commit changes to `openapi.yaml`.

## Best Practices

- Never hand-edit files under `generated/`.
- Regenerate after any schema change before reviewing, so diffs reflect the actual outputs.
- Treat `revision` as the package version; CI will replace `VERSION` on tagged releases.
- Prefer adding new endpoints over changing or removing existing ones unless you also coordinate client/server updates.
