# GitHub Copilot Instructions

This repository contains the **OpenAPI specification** for the Server Orchestrator API and handles automatic generation of server stubs and client SDKs.

## Project Context

Server Orchestrator is a Kubernetes-first platform for managing containerized game server infrastructure (starting with Minecraft). This API repo defines the contract for:
- Multi-tenant namespace management
- Server lifecycle operations (create, start, stop, delete)
- Template-based server provisioning with RBAC restrictions
- Resource bucket management (shared assets: configs, plugins, mods)
- Label definitions for categorization and access control
- File management and editor operations
- Audit logging and observability

## Technology Stack

- **API Definition**: OpenAPI 3.x (`openapi/openapi.yaml`)
- **Code Generation**: OpenAPI Generator via Maven
- **Server Stubs**: Spring Boot 3 (primary), JAX-RS (alternative)
- **Client SDKs**: TypeScript (fetch + axios)
- **Build**: Maven 3.8+, Java 17+
- **CI/CD**: GitHub Actions (validation, versioning, publishing)
- **Distribution**: GitHub Packages (Maven + npm)

## Code Generation Guidelines

### When suggesting OpenAPI changes:
1. Follow OpenAPI 3.0+ specification strictly
2. Use `components/schemas` for reusable models
3. Include comprehensive descriptions for all endpoints, parameters, and schemas
4. Define proper HTTP status codes (200, 201, 400, 401, 403, 404, 409, 500)
5. Use consistent naming: `kebab-case` for paths, `camelCase` for properties
6. Add `tags` to group related endpoints
7. Include `x-` extensions only when necessary for generator configuration

### Domain Model Entities:
- **Namespace**: Tenant boundary with settings, RBAC roles, and resource quotas
- **Server**: Containerized workload with labels, template, and attached resources
- **Template**: Blueprint with image, environment, resources, and restrictions
- **ResourceBucket**: Shared asset storage with labels
- **LabelDefinition**: Namespace-scoped categorization keys
- **NamespaceRole**: Custom RBAC roles with granular permissions

### API Patterns:
- **CRUD operations**: Standard REST verbs (GET, POST, PUT, PATCH, DELETE)
- **Namespace scoping**: Most resources are namespaced (e.g., `/namespaces/{namespaceId}/servers`)
- **Actions**: Use POST for non-CRUD operations (e.g., `/servers/{serverId}/start`)
- **Pagination**: Use query params `page`, `size`, `sort` for list endpoints
- **Filtering**: Use query params matching property names
- **Async operations**: Return 202 Accepted with operation tracking

### Security & RBAC:
- All endpoints require authentication (JWT bearer tokens)
- Permission checks at global and namespace levels
- Sensitive fields (secrets, passwords) should have `writeOnly: true`
- Audit logging for all mutations

### File Operations:
- Support upload, download, list, edit, delete operations
- Enforce label-based path restrictions from templates
- Validate write permissions via namespace roles

## Maven Configuration

When modifying `pom.xml`:
- Keep `revision` property for dynamic versioning
- Maintain separate `openapi-generator-maven-plugin` executions for each generator target
- Output directory: `generated/{generator-name}`
- Don't commit `generated/` directory
- Ensure `inputSpec` points to `openapi/openapi.yaml`

## Testing Considerations

When suggesting API changes, consider:
- Backward compatibility (use `deprecated: true` for breaking changes)
- Validation rules (required fields, patterns, min/max values)
- Error response schemas (consistent error format)
- Example values for documentation

## CI/CD Awareness

- Validation runs on changes to `openapi/**` or `pom.xml`
- Releases triggered by version tags (e.g., `v1.2.3`)
- Artifacts published to GitHub Packages automatically
- Local builds always use `VERSION` placeholder

## Common Operations

### Adding a new endpoint:
1. Define path under appropriate namespace
2. Add request/response schemas to `components/schemas`
3. Include security requirements
4. Add examples for request/response bodies
5. Document error cases

### Adding a new entity:
1. Create schema in `components/schemas`
2. Include all CRUD endpoints
3. Define relationships with other entities
4. Add validation constraints
5. Consider audit fields (createdAt, updatedAt, createdBy)

### Modifying existing endpoints:
1. Check for breaking changes
2. Add `deprecated: true` if replacing functionality
3. Update all affected schemas
4. Maintain backward compatibility when possible

## Code Style

- Use descriptive operation IDs (e.g., `createServer`, `listNamespaces`)
- Group related schemas together in the file
- Add inline comments for complex validation rules
- Keep descriptions concise but complete
- Use examples to clarify expected formats
