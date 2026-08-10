# icf-registry

*[日本語版 / Japanese](README.md)*

**An address book of shared vocabulary ── a small registry of ICF code references, support codes, field terms, and mappings**

This is the first layer (Registry = shared meaning) of [ICF HUB](https://github.com/mirai-no-design/icf-hub).

This repository registers and serves:

- references to ICF codes (body functions b/s, activities & participation d, environmental factors e)
- shared vocabularies for support, activities, participation and environments
- field terms (the words actually used on the ground: "bagging," "shelf stocking," "customer service"…)
- mappings between organization-specific codes and the shared vocabulary

## Design policy: nothing clever

This is an **address book**. Entries carry only **ID, label, meaning reference, version, mapping, and source**. Advanced search, recommendation, similarity computation, and AI auto-mapping are all deliberately excluded (they belong to the Applications layer).

**The canonical registry is the data files in this repository** (`registry/*.json`). A server is not required. If served over HTTP, the optional read-only API in [SPEC.en.md](SPEC.en.md) §6 returns nothing more than copies of the files.

## Layout

```
registry/
  codesystems.json   # declared code systems (ICF, in-house systems, org codes…)
  concepts.json      # concept entries (ICF references + shared vocabulary)
  terms.json         # field-term entries (with concept mappings and confidence)
  mappings.json      # org-specific code ⇄ concept mappings
schema/              # JSON Schemas for machine validation
examples/            # minimal examples for contributions
SPEC.md / SPEC.en.md # the registry specification (canonical)
GOVERNANCE.md        # rules for registration and change
```

## Minimal examples

```json
{
  "id": "icfhub:concept/d740",
  "codesystem": "who-icf",
  "code": "d740",
  "label_ja": "公的な対人関係",
  "label_en": "Formal relationships",
  "layer": "participation",
  "definition_ref": "https://icd.who.int/dev11/l-icf/en",
  "status": "active",
  "since": "0.1.0"
}
```

```json
{
  "id": "icfhub:term/000123",
  "surface_form": "袋詰め",
  "lang": "ja",
  "domain": "work",
  "concepts": [
    { "concept": "icfhub:concept/d440", "confidence": 0.85 }
  ],
  "source": "field registration (employment support)",
  "status": "active",
  "since": "0.1.0"
}
```

## About WHO ICF (important)

The ICF classification itself (full code texts, definitions, structural notes) is the property of the WHO. This registry **does not redistribute ICF classification text**. Concept entries hold only code numbers, short labels, and a reference to the WHO source (`definition_ref`). Implementers who need full definitions should consult the WHO's official distribution under its terms of use.

## Contributing

Additions and corrections are accepted via Pull Request. A source citation is mandatory. See [GOVERNANCE.md](GOVERNANCE.md).

## License

- Specification and documents: CC BY 4.0
- Registry data (`registry/`): CC BY 4.0 (ICF itself remains subject to WHO terms)
- JSON Schemas (`schema/`): MIT (for free embedding in implementations)

See [CITATION.cff](CITATION.cff) for citation. Each release receives a DOI via Zenodo.
