# VulnerableCode: On-demand live evaluation of packages and Integration with VulnTotal and its browser extension

## Organization
[AboutCode](https://www.aboutcode.org)

**Michael Ehab Mikhail**  
GitHub: [michaelehab](https://github.com/michaelehab)  
LinkedIn: [@michaelehab16](https://www.linkedin.com/in/michaelehab16/)  
Project: [VulnerableCode](https://github.com/aboutcode-org/vulnerablecode)  
Official GSoC project page: [Project Link](https://summerofcode.withgoogle.com/programs/2025/projects/uF0kzMAg)  
GSoC Proposal: [Proposal Link](https://docs.google.com/document/d/1Tkk4MoPWXFj9r_U5cp3E4AhJW6QlHxTElyzpII_f4LM/edit?usp=sharing)  

---

## Overview

VulnerableCode traditionally relied on **batch importers** to fetch and store all advisories from a source at once. While effective for building complete databases, batch importers are slow and resource-heavy for developers who only need vulnerability data for a **single package**.

This project introduces **live importers**, a new class of importers that operate in a *package-first* mode. Instead of pulling all advisories, they run against a single PackageURL (PURL), returning only the advisories affecting that package. This makes vulnerability evaluation **faster, more efficient, and more personalized**, since the database is gradually filled with only the advisories that matter to each user.

To support this, I added:

- A new **`LIVE_IMPORTERS_REGISTRY`** that tracks available live importers.
- A new **API endpoint** that accepts a PURL and runs all compatible live importers in parallel (unless the `no_threading` flag is set).
- Integration with **VulnTotal** and its **browser extension**, enabling users to evaluate packages in real-time through a seamless interface.

This work bridges the gap between **batch-first databases** and **package-first queries**, improving VulnerableCode's flexibility and enabling better integration with developer workflows.

> **Note:** A PURL (Package URL) is a universal way to identify and locate software packages. [More on PURL](https://github.com/package-url)

---

## Project Design and Architecture

The new live importers system builds on existing batch importers, while introducing a parallel registry and execution model for package-first runs.

### Importer Registries

- `IMPORTERS_REGISTRY` continues to hold batch importers (V1/V2).
- `LIVE_IMPORTERS_REGISTRY` holds live importers.

Each live importer:

- Inherits from its batch importer (when logic can be reused), or directly from `VulnerableCodeBaseImporterPipelineV2` when a separate implementation is needed.
- Declares a `supported_types` array, defining compatible package ecosystems (`"pypi"`, `"npm"`, `"maven"`, `"generic"`, etc).
- Implements a package-first `collect_advisories()` method, which restricts results to advisories relevant to the given PURL.

<img width="828" height="626" alt="image" src="https://github.com/user-attachments/assets/7276fb93-7d39-40dd-b515-daa831aa3d9f" />

*Class architecture showing relationship between `IMPORTERS_REGISTRY` and `LIVE_IMPORTERS_REGISTRY`.*

---

### API Endpoint

The new API endpoint is responsible for handling live evaluation requests.

- **Input:**
  - `purl_string` (required)
  - `no_threading` (optional, default `false`)
- **Execution:**
  - Checks `LIVE_IMPORTERS_REGISTRY` for importers whose `supported_types` match the PURL.
  - Runs compatible importers in parallel unless `no_threading` is true.
- **Output:**
  - A set of advisories affecting the requested PURL, imported directly into the database and returned as JSON.
  - 
<img width="1202" height="591" alt="image" src="https://github.com/user-attachments/assets/7170b44f-ad15-4fc2-9577-f8c88c3b427a" />


*Flow of API endpoint: selecting compatible live importers and executing them in parallel.*

---

### Integration with VulnTotal

The new API was integrated into VulnTotal as an optional datasource:

- VulnTotal now checks the local environment for `VCIO_HOST`, `VCIO_PORT`, and `ENABLE_LIVE_EVAL` flags in `.env`.
- If enabled, VulnTotal queries VulnerableCode in package-first mode.
- This allows VulnTotal to use both its proprietary datasources **and** the user's gradually built local database, improving coverage and personalization.

---

### Integration with VulnTotal Browser Extension

The VulnTotal browser extension was updated to support live importers:

- Users can enable the "Local VulnerableCode" datasource and live evaluation option.
- When enabled, package lookups are forwarded to the new API, retrieving advisories in real-time.
- This reduces setup effort—developers can get live vulnerability checks directly in their browser, provided they have a local VC instance.

![Recording2025-08-21232838-ezgif com-optimize (1)](https://github.com/user-attachments/assets/756b33a6-38ba-4bfc-bfeb-374b89dbe6aa)

*VulnTotal and its browser extension consuming the new live evaluation API.*

---

## Linked Pull Requests

| Sr. no | Name                                                                | Link |
|--------|---------------------------------------------------------------------|------|
| 1      | Add Live Evaluation API endpoint and PyPa live pipeline importer     | [aboutcode-org/vulnerablecode#1969](https://github.com/aboutcode-org/vulnerablecode/pull/1969) |
| 2      | Add Gitlab Live V2 Importer                                          | [aboutcode-org/vulnerablecode#1910](https://github.com/aboutcode-org/vulnerablecode/pull/1910) |
| 3      | Add Curl Live Importer V2                                            | [aboutcode-org/vulnerablecode#1923](https://github.com/aboutcode-org/vulnerablecode/pull/1923) |
| 4      | Add Elixir Security Live V2 Importer                                 | [aboutcode-org/vulnerablecode#1935](https://github.com/aboutcode-org/vulnerablecode/pull/1935) |
| 5      | Add NPM Live Importer V2                                             | [aboutcode-org/vulnerablecode#1941](https://github.com/aboutcode-org/vulnerablecode/pull/1941) |
| 6      | Add GitHub OSV Live V2 Importer Pipeline                             | [aboutcode-org/vulnerablecode#1977](https://github.com/aboutcode-org/vulnerablecode/pull/1977) |
| 7      | Add Postgres Live V2 Importer Pipeline                               | [aboutcode-org/vulnerablecode#1982](https://github.com/aboutcode-org/vulnerablecode/pull/1982) |
| 8      | Add PySec Live V2 Importer Pipeline                                  | [aboutcode-org/vulnerablecode#1983](https://github.com/aboutcode-org/vulnerablecode/pull/1983) |
| 9      | Add Local VulnerableCode Datasource in VulnTotal and allow live eval | [aboutcode-org/vulnerablecode#1985](https://github.com/aboutcode-org/vulnerablecode/pull/1985) |
| 10     | Integrate Local VulnerableCode datasource and live evaluation        | [aboutcode-org/vulntotal-extension#17](https://github.com/aboutcode-org/vulntotal-extension/pull/17) |

---

## Closing Thoughts

This project was an exciting step forward from my 2024 GSoC work. By moving from batch importers to package-first live importers, we enabled a faster, more personalized, and more flexible way of building vulnerability databases.

I especially enjoyed designing the **registry + API architecture** and integrating it seamlessly across **VulnerableCode, VulnTotal, and the browser extension**. This work lays the foundation for even richer interactivity in the ecosystem and brings vulnerability evaluation closer to developers' workflows.

I appreciated the weekly status calls and the feedback I received from my mentors and the amazing team. They were really helpful and supportive.  

- [Philippe Ombredanne](https://github.com/pombredanne)  
- [Ayan Sinha Mahapatra](https://github.com/AyanSinhaMahapatra)  
- [Tushar Goel](https://github.com/TG1999)  
- [Keshav Priyadarshi](https://github.com/keshav-space)  
