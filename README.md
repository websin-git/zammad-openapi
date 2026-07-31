# Zammad OpenAPI

An unofficial, code-generation-friendly OpenAPI 3.0 description of the
publicly supported Zammad REST API.

The specification was verified on 2026-07-31 against the current stable Zammad
**7.1.1** release (2026-06-25) and the matching upstream source tag. It defines
151 operations across 88 paths and 130 reusable schemas, covering the
documented REST resources, resource searches, Generic CTI, and the documented
self-hosted monitoring API.

## Files

- [`openapi.yaml`](openapi.yaml) — bundled OpenAPI specification
- [`.redocly.yaml`](.redocly.yaml) — lint configuration
- [`.github/workflows/openapi.yml`](.github/workflows/openapi.yml) — CI lint,
  validation, and client-generation smoke test

## Import into Postman

1. In Postman, select **Import** and choose `openapi.yaml`.
2. Set the server variables:
   - `scheme`: normally `https`
   - `fqdn`: the hostname of your Zammad instance
3. Select one authentication method:
   - Zammad access token: set the complete `Authorization` value to
     `Token token=<access-token>`.
   - OAuth2 access token: use Bearer authentication.
   - Basic authentication: only if enabled by the instance; it is not
     recommended and does not work with two-factor authentication.

The three global security definitions are alternatives, not cumulative
requirements. Generic CTI uses its secret path token. Monitoring endpoints can
use their query token or an authorized API credential.

## Generate a client

Any OpenAPI 3.0-compatible generator can use the bundled file. For example:

```bash
docker run --rm \
  -v "$PWD:/local" \
  openapitools/openapi-generator-cli:v7.22.0 generate \
  -i /local/openapi.yaml \
  -g typescript-fetch \
  -o /local/generated/typescript-fetch
```

Replace `typescript-fetch` with another supported generator as needed.
Generated output is ignored by Git.

## Validate

```bash
docker run --rm \
  -v "$PWD:/spec" \
  redocly/cli:2.43.1 lint /spec/openapi.yaml \
  --config /spec/.redocly.yaml

docker run --rm \
  -v "$PWD:/local" \
  openapitools/openapi-generator-cli:v7.22.0 validate \
  -i /local/openapi.yaml
```

The GitHub Actions workflow also generates and compiles a TypeScript Fetch
client and converts the specification with Postman's official converter.

## API conventions represented by the specification

- Base URL: `{scheme}://{fqdn}/api/v1/...`
- JSON requests and responses use UTF-8.
- `page` and `per_page` control pagination. Zammad enforces
  controller-specific hard limits.
- Search responses vary with `full`, `expand`, `with_total_count`, and
  `only_total_count`; the alternatives are modeled explicitly.
- `From` can act on behalf of another user and requires `admin.user`.
- Ticket, user, organization, and group schemas allow additional properties
  because Object Manager fields are installation-specific.
- Operation-specific Zammad rights are recorded in `x-zammad-permissions`.
- `DELETE /tags/remove` and `DELETE /links/remove` follow the upstream API and
  require JSON bodies. Ensure intermediaries do not discard DELETE bodies.

## Scope

This project covers the public API surface described by the official Zammad
REST documentation plus source-confirmed public search and monitoring
endpoints. Browser-internal Rails endpoints and undocumented administration UI
contracts are intentionally excluded: they are not stable public APIs and
including them would make generated clients misleading.

Some response fields depend on permissions, enabled features, installed
add-ons, locale, and Object Manager configuration. Such objects intentionally
allow documented extension points.

## Upstream sources

- [Zammad 7.1.1 release](https://zammad.com/en/product/releases/7-1-1-7-0-3)
- [Zammad REST API documentation](https://docs.zammad.org/en/latest/api/intro.html)
- [Zammad source tag 7.1.1](https://github.com/zammad/zammad/tree/7.1.1)
- [Monitoring API documentation](https://admin-docs.zammad.org/en/latest/system/monitoring.html)

## Updating for a new Zammad release

1. Pin the new stable source tag; do not validate against a moving branch.
2. Compare the official REST documentation, route definitions, controllers,
   policies, models, and breaking changes.
3. Update `info.version`, schemas, operations, permissions, and examples.
4. Run both validators and at least one client-generation smoke test.
5. Test-import `openapi.yaml` in a current Postman release.

## License

No license is declared in this draft because licensing is a repository-owner
decision. Add an appropriate `LICENSE` before inviting external reuse or
contributions.

Zammad is a trademark of Zammad Foundation. This repository is not an official
Zammad Foundation publication.
