# ICF Registry Specification v0.1 (Draft)

*[日本語版 / Japanese](SPEC.md)*

| Item | Content |
|---|---|
| Status | Draft |
| Version | 0.1.0 |
| Published by | Mirai no Design Inc. (drafter) / ICF HUB Project |
| License | Text CC BY 4.0 / Schemas MIT |

The keywords MUST, SHOULD and MAY follow RFC 2119.

## 1. Goals and non-goals

### 1.1 Goal

To provide an **address book of shared vocabulary** that different services and organizations consult when exchanging information about human functioning, activities, participation, environments and support.

### 1.2 Non-goals (what this registry will not do)

- Advanced search, ranking, recommendation, similarity computation
- NLP or AI auto-mapping (the **results** of mapping may be registered)
- Holding personal data (the registry contains vocabulary only — never data about individuals)
- Redistributing ICF classification text (§7)

## 2. Data model

The registry consists of four entry types. All entries share the fields `id` / `status` / `since` / `deprecated` (optional) / `source` (optional).

### 2.1 CodeSystem

Declaration of a referenced code system.

| Field | Type | Req. | Description |
|---|---|---|---|
| `id` | string | MUST | System ID (e.g., `who-icf`, `org-example-childcheck`) |
| `name` | string | MUST | Name |
| `publisher` | string | MUST | Publisher |
| `version` | string | SHOULD | Referenced version |
| `url` | string | SHOULD | URL of the authoritative source |
| `license_note` | string | SHOULD | Note on terms of use |

### 2.2 Concept

One entry of shared vocabulary: a reference to an ICF code, or a shared concept not covered by ICF (support codes, personal-context layer, etc.).

| Field | Type | Req. | Description |
|---|---|---|---|
| `id` | string | MUST | Stable ID of the form `icfhub:concept/{code}` |
| `codesystem` | string | MUST | CodeSystem id (e.g., `who-icf`, `icfhub-support`) |
| `code` | string | MUST | Code within the system (e.g., `d740`) |
| `label_ja` / `label_en` | string | MUST (ja) | Short label. For ICF, keep it a **short form** only |
| `layer` | string | MUST | `body` / `activity` / `participation` / `environment` / `personal` / `support` |
| `parent` | string | MAY | Parent concept id (hierarchy) |
| `definition_ref` | string | SHOULD | URL of the authoritative definition (WHO source for ICF) |
| `version` | string | MUST | Entry version |

### 2.3 Term

A word used in the field, mapped to concepts. One term may map to several concepts.

| Field | Type | Req. | Description |
|---|---|---|---|
| `id` | string | MUST | `icfhub:term/{number}` |
| `surface_form` | string | MUST | The written form |
| `lang` | string | MUST | Language (BCP 47, e.g., `ja`) |
| `domain` | string | SHOULD | `work` / `education` / `community` / `care` / `tourism` … |
| `concepts[]` | array | MUST | Array of `{concept, confidence(0–1)}` |
| `source` | string | MUST | Source (which field or literature the mapping came from) |

### 2.4 Mapping

Correspondence between an organization-specific code and a concept.

| Field | Type | Req. | Description |
|---|---|---|---|
| `id` | string | MUST | `icfhub:mapping/{number}` |
| `source_system` | string | MUST | CodeSystem id of the org-specific system |
| `source_code` | string | MUST | Code within that system |
| `target_concept` | string | MUST | Target concept id |
| `relation` | string | MUST | `exact` / `broader` / `narrower` / `related` (following SKOS) |
| `confidence` | number | SHOULD | 0–1 |
| `evidence` | string | MUST | Source and rationale (who confirmed this mapping, and how) |

## 3. Identifiers

- Entry IDs take the form `icfhub:{type}/{local}` and, once issued, are **never deleted or reused** (MUST)
- Retirement is expressed with `status: "deprecated"` and `deprecated: "<version>"` (MUST)
- If a resolvable `https://` URI namespace is adopted later, `icfhub:` IDs remain valid aliases

## 4. Versioning

- The registry as a whole uses **semantic versioning** (`MAJOR.MINOR.PATCH`) per release
- Adding entries or sources = MINOR. Fixing label typos = PATCH. Meaning-changing edits = a new ID plus deprecation of the old one (grounds for MAJOR)
- Exchanges (see icf-exchange-spec) SHOULD state the referenced registry version as `registry_version`

## 5. Distribution

- The canonical form is `registry/*.json` in this repository (UTF-8)
- Each file has the top-level form `{"registry_version": "...", "entries": [...]}`
- All entries MUST validate against `schema/*.schema.json` (JSON Schema draft 2020-12)

## 6. Optional read-only API

If the registry is served over HTTP, the following conventions apply. **The API returns copies of the files and MUST NOT provide advanced search.**

```
GET /v0/codesystems
GET /v0/concepts            (?layer= filtering MAY be offered)
GET /v0/concepts/{id}
GET /v0/terms?surface_form= (exact / prefix match only, MAY)
GET /v0/mappings?source_system=
```

- Responses are JSON (`application/json; charset=utf-8`)
- Every response MUST include `registry_version`
- No authentication (public data only). Personal data MUST NOT be served through this API

## 7. Handling of WHO ICF

- This registry MUST NOT hold or redistribute ICF classification text (full definitions, inclusion/exclusion notes, etc.)
- ICF references in Concepts are limited to code number, short label, and `definition_ref`
- Implementers needing full ICF definitions consult the WHO's official distribution under its terms

## 8. Prohibition of personal data

Information that could identify an individual, or assessment data about an individual, MUST NOT be registered. Exchanging data about people is the domain of icf-exchange-spec, and even there, consent management is a precondition.
