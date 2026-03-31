# Cartographer-Framework-Tool
The Cartographer Framework is the Active Counterpart to the Passive Framework ATLAS, established by MITRE. This is the tool which serves the purpose as outlined in the whitepaper.


## A MITRE ATLAS-Aligned Automated Adversarial Assessment Framework

Cartographer-Framework is an integrated pipeline for conducting structured, repeatable red team security assessments against Large Language Models. It operationalizes the MITRE ATLAS threat matrix into an automated workflow that generates adversarial payloads, dispatches them to model provider APIs, analyzes responses for indicators of compromise, and produces quantified deliverable reports mapped to the ATLAS tactic taxonomy.

---

## Pipeline Overview

The framework is organized into a six-tab GUI pipeline, each stage feeding the next:

**GN → RS → AN → DL → RD → PB**

Generation → Responses → Analyses → Deliverable → Raw Data → Plugin Builder

---

## Main Generator (GN)

The generation tab provides granular technique-level control over payload creation, organized by the MITRE ATLAS tactic hierarchy.

### Technique Organization by Tactic

Techniques are displayed in a navigable tree organized by ATLAS tactic categories (Reconnaissance, Initial Access, Execution, Persistence, etc.). Each technique node shows its AML identifier, human-readable name, and assigned processing mode. Operators select individual techniques or tactic groups for targeted payload generation.

<!-- Image: Tactic tree sidebar showing expandable tactic groups with technique nodes -->
<!-- Commentary: The tactic tree mirrors the MITRE ATLAS framework structure, allowing operators to quickly navigate to specific attack categories and select techniques for generation. -->

<!-- Image: Technique detail panel showing AML ID, goals, mode badge, and generation controls -->
<!-- Commentary: Selecting a technique reveals its ATLAS goals, assigned processing mode, and generation parameters. The operator can adjust patterns, bypasses, and AD profile settings before generating. -->

### Payload Generation by AML Technique

Each payload is generated in the context of a specific AML technique identifier (e.g., `AML.T0051.000` — Direct Prompt Injection, `AML.T0056` — LLM Meta Prompt Extraction). This ensures every test case traces back to a catalogued threat, enabling structured coverage analysis in later pipeline stages.

### Ollama Model Payload Generation

Payload generation is driven by a local Ollama instance. The framework sends structured prompts containing technique goals, typology context, probe seeds, and AD profile data to the Ollama model, which returns unique adversarial payloads. This approach provides automated efficiency while ensuring payload uniqueness — each generation pass produces novel prompt constructions rather than static templates.

### Ollama Model Control

Operators configure the Ollama endpoint URL and model selection (e.g., `llama3.1:8b`, `mistral:7b`) through the Settings tab. This allows teams to use different generation models depending on the sophistication required for a given assessment.

---

## Data Runner & Responses (RS)

The Responses tab aggregates generated payloads, dispatches them to target model providers, and analyzes the responses.

### Red Team Technique Aggregation

Payload files are automatically scanned and grouped by technique type (Prompt Injection, Enumeration, Interrogation, Web Poisoning, Data Poisoning) with filterable tabs. Each payload card displays its AML technique ID, technique name, description, parent tactic, and an expandable goals section — all pulled from the ATLAS TTP data.

<!-- Image: Responses tab file list showing technique cards with metadata -->
<!-- Commentary: Scanned payload files are presented as rich cards with ATLAS metadata. The operator sees at a glance what each file tests, which tactic it covers, and how many payloads it contains. -->

<!-- Image: Filter tabs showing technique type categories -->
<!-- Commentary: Filter tabs let operators focus on specific attack categories. The count badge shows how many files and payloads are in each category. -->

### Prepared Payloads Viewport

The file list serves as a viewport into all prepared payloads for the current project. Operators can select or deselect individual files via checkboxes, use "Select All" for bulk operations, and see real-time payload counts that update as selections change.

### Data Runner Panel — Multi-System Compatibility

The Data Runner dispatches selected payloads to target model endpoints. It supports three testing modes:

- **Main Provider Testing**: Sends payloads to commercial LLM APIs (Anthropic, OpenAI, Google, xAI, DeepSeek) using configured API keys. Supports per-provider payload count control with round-robin distribution.
- **Custom Model Testing**: Operators can specify custom API endpoints for testing proprietary or self-hosted models.
- **Ollama Model Testing**: Payloads can be sent to a local Ollama instance for testing open-source models in a controlled environment.

### A/B Prompt Testing

The System Prompt Editor supports Variant A and Variant B prompt configurations. This enables veracity testing — operators can define a baseline system prompt (Variant A) and a security-enhanced prompt (Variant B), then run identical payloads against both to measure the impact of prompt-level mitigations. This directly supports remediation validation: which prompt yields less vulnerable output?

<!-- Image: System prompt editor with A/B tabs -->
<!-- Commentary: The A/B prompt editor lets operators save two system prompt variants. Running the same payloads against both quantifies the defensive value of prompt engineering changes. -->

### System Prompt Editor

The integrated system prompt editor provides:

- **A/B variant tabs** for comparative testing
- **Save** to both the project directory and the global Prompts directory (ensuring the Data Runner reads the correct prompt)
- **Load** from existing project files
- **Import** from external text files

### Smart Response Analyzer — Ollama-In-The-Loop

The Response Analyzer processes model outputs through a multi-layer review pipeline:

- **String Review**: Pattern-based keyword matching for compliance indicators
- **AI Review (Ollama-In-The-Loop)**: Semantic analysis using a local Ollama model to determine whether the target model's response indicates compromise
- **Human Review**: Placeholder for manual analyst review of ambiguous cases

Each review produces an IOCD (Indicator of Compromise Determination) file with a binary determination (Y/N), confidence score, reasoning, and the full input/output pair with metadata.

---

## Raw Data Repository (RD)

The Raw Data tab centralizes all test artifacts into a single Master.json repository. The Build Master.json operation walks the project's Analysis directory, parses all IOCD files across all models, technique types, and review layers, and assembles a unified data structure suitable for statistical analysis.

<!-- Image: Raw Data tab showing Master.json browser -->
<!-- Commentary: The Master.json browser lets operators inspect the consolidated test repository before proceeding to analysis. -->

---

## Analysis Pipeline (AN)

The Analyses tab runs a multi-stage statistical analysis pipeline on the consolidated test data.

### Multi-Layer Statistical Analysis

The analysis pipeline operates in three stages:

1. **Atlas Analysis**: Computes Layer 1 rankings (by model, by technique type, by technique ID, by typology, by category) and Layer 2 cross-model divergence analysis (median, range, and inter-model variance per dimension).

2. **Atlas Visualization**: Generates chart PNGs from the analysis data — bar charts, heatmaps, strip plots, and dumbbell plots covering all Layer 1 and Layer 2 dimensions.

3. **Atlas Conclusions**: Produces narrative text conclusions for each chart, summarizing key findings in prose suitable for a deliverable report.

### Data Aggregation

The analysis aggregates data across all concepts tested: technique types, individual AML techniques, typologies/procedures, and enumeration categories. This produces ranked vulnerability profiles per model and comparative cross-model assessments.

### Visual and Textual Outputs

Each analysis dimension produces both a chart image (PNG) and a narrative conclusion (JSON text). These are stored in the project's charts and conclusions directories for use by the Deliverable Generator.

<!-- Image: Analyses tab showing the three-stage pipeline -->
<!-- Commentary: The analysis pipeline runs sequentially: analysis → visualization → conclusions. Each stage builds on the output of the previous one. -->

---

## Deliverable Generator (DL)

The Deliverable tab assembles a professional Word document (.docx) report from the analysis outputs.

### Report Assembly

The generator produces a themed, paginated report containing:

- **Cover page** with assessment metadata (models tested, total requests, vulnerability threshold, tier ranges)
- **Executive Summary** with key findings in prose
- **Layer 1 — Basic Rankings**: One page per chart (by model, by technique type, by technique ID, by typology, by category, heatmaps) with narrative conclusions
- **Layer 2 — Median, Range & Divergence**: Strip plots and dumbbell plots per dimension with conclusions
- **Appendix — Vulnerability Tier Definitions**: Classification criteria for Resistant, Partially Vulnerable, Vulnerable, and Critically Vulnerable tiers
- **Red Team Coverage Map**: Visual diagram mapping tested techniques to ATLAS tactic categories with connecting lines
- **Tactics Not Addressed**: Radial diagram of uncovered tactics with suggested techniques for optimal coverage

### Theme Support

Reports support multiple visual themes (Midnight, Corporate, Ember, Arctic, Slate) for branding flexibility.

---

## Settings

The Settings tab provides:

- **Ollama Configuration**: Endpoint URL and model selection
- **Configuration Editors**: Quick access to AD Profile, Prompt Patterns, Technique Config, and Interrogation editors
- **Theme Selection**: Five GUI color schemes (Midnight, Vellum, Meridian, Terrain, Expedition) for operator preference

---

## Plugin Builder (PB)

The Plugin Builder enables operators and researchers to add new attack techniques to the framework without modifying core code.

### Overview

Adding a new technique involves defining a JSON schema (what to test), a Python plugin class (how to iterate and prompt), and a technique_config.json entry (how to route). The Plugin Builder GUI automates all three.

### Step 1: Add Mode Reference in technique_config.json

Before using the Plugin Builder, ensure the following configuration entries exist:

- **Mode Reference**: In the `mode_reference` section of `technique_config.json`, add an entry for the new mode name (e.g., `"API_FUZZING": "API input fuzzing"`).
- **Technique Override**: Add a subcategory object for the new `AML.T00NN` technique, specifying the mode, patterns, bypasses, and the subcategory designator field.
- **Subcategory Designator**: Each mode that uses the enumeration engine needs a field that connects the AML technique to a specific category in the JSON schema (e.g., `"enumeration_category": "API_Fuzzing"`).
- **Subcategory Metadata**: The category in the JSON must include a `technique_id` and `description` in its metadata.

### Step 2: Define the Subcategory Designator

If the technique addition can be subcategorized (which most can), create a subcategory designator field. This is the key that links a technique_config entry to a specific category within the techniques JSON file.

**Example**: The `enumeration_category` field connects `AML.T0007` to the `AI_Artifacts` category, `AML.T0013` to `Model_Ontology`, etc. A new mode like `API_FUZZING` might use `api_fuzzing_category` to point to `API_Fuzzing`.

### Step 3: JSON Validation Requirements

The plugin system validates the JSON schema before generation begins:

- **Subcategory Designator Validation**: Confirms the operator specified which category to use (e.g., `enumeration_category` is not empty).
- **Attack Type Designator Validation**: Confirms the category contains iterable attack types — either `typologies` (for typology-level iteration) or `mandatory_probes` within typologies (for probe-level iteration).

### Step 4: Extract Items

The extraction phase reads the JSON schema and produces a list of WorkItems for the engine to iterate:

- **System Prompt Extraction**: Each category defines a `system_prompt` that sets the Ollama generation context for all items in that category.
- **Category-Level Iteration**: The engine iterates through categories based on the subcategory designator.
- **Typology-Level Iteration**: Within each category, the engine iterates through the defined typologies (attack types).
- **Metadata Attachment**: Each typology should include at minimum an `objective` and `short_note` field. The security researcher can add additional context as needed, but these two fields are the minimum for the pipeline to produce meaningful output.

### Step 5: Build Prompt

The prompt construction phase assembles context for Ollama at three levels:

- **Category-Level Prompting**: The `system_prompt` from the JSON category sets the overall generation framing (e.g., "You are an AI security researcher generating test prompts designed to...").
- **Typology-Level Prompting** (`_build_typology_prompt`): For categories without mandatory probes, builds a prompt from the typology's `description`, `short_note`, and `example` fields. One payload per typology.
- **Probe-Level Prompting** (`_build_probe_prompt` and `_build_framework_context`): For categories with mandatory probes, builds a prompt from the probe's `objective`, `prompt_seed`, and `rationale`, enriched with typology context and optional framework reference data. One payload per probe.

### Step 6: Tag Payloads with Metadata

Every generated payload is tagged with lineage metadata that flows through the entire pipeline into the analysis and deliverable stages:

| Field | Value | Purpose |
|---|---|---|
| `source` | Name of the technique plugin | Identifies which plugin generated the payload |
| `category` | Subcategory key | Links to the JSON category (e.g., `AI_Artifacts`) |
| `typology` | Typology key | Identifies the specific attack type (e.g., `Session_State_Probing`) |
| `iteration_level` | `"probe"` or `"typology"` | Indicates the iteration granularity |
| `probe_key` | Probe identifier (probe-level only) | Identifies the specific test case |
| `objective` | Probe objective (probe-level only) | Human-readable goal of the test |
| `short_note` | Typology short note | Brief label for display and grouping |

This metadata enables the analysis pipeline to break down results by any dimension — which probes had the highest compromise rate, which typologies were most effective, which categories revealed the most vulnerabilities.

---

## Architecture

```
Cartographer-Framework/
├── run_gui_v3.py                  ← Entry point
├── gui/
│   ├── main_app_v3.py             ← Integrated 7-tab GUI
│   ├── theme.py                   ← Theme system with 5 palettes
│   └── themes/                    ← CTk theme JSON files
├── core/
│   ├── project_manager.py         ← Project lifecycle management
│   ├── generator_bridge.py        ← Technique → payload routing
│   └── backend/
│       ├── models.py              ← Data models and enums
│       ├── payload_generator.py   ← Plugin-based generation engine
│       ├── config_loader.py       ← JSON configuration loading
│       ├── bypass_processor.py    ← Bypass transformation suite
│       └── plugins/               ← Auto-discovered mode plugins
│           ├── __init__.py        ← PluginRegistry (auto-discovery)
│           ├── base_plugin.py     ← ModePlugin abstract base class
│           ├── general_flow.py    ← Prompt injection patterns
│           ├── enumeration.py     ← Typology/probe iteration
│           ├── interrogation.py   ← Extraction technique iteration
│           └── delegate_modes.py  ← Web poison, data poison, etc.
├── editors/                       ← Configuration editor windows
├── tools/
│   ├── data_runner.py             ← Multi-provider API dispatcher
│   ├── response_analyzer.py       ← IOCD determination engine
│   ├── Raw_Repo_Creator.py        ← Master.json builder
│   ├── atlas_analysis.py          ← Statistical analysis engine
│   ├── atlas_visualize.py         ← Chart generation
│   ├── atlas_conclusions.py       ← Narrative conclusion generation
│   └── ATLAS_Deliverable_Maker.py ← Word document report builder
└── data/                          ← JSON configuration files
    ├── atlas_ttp.json
    ├── technique_config.json
    ├── prompt_patterns.json
    ├── enumeration_techniques.json
    ├── interrogation_techniques.json
    └── informational_printouts.json
```

---

## Adding a New Attack Technique

The fastest path to adding a new technique:

1. **Write the JSON**: Define your typologies and probes in `enumeration_techniques.json` (or a standalone JSON with the same schema)
2. **Add technique_config entry**: Point the AML technique to your category with `"mode": "ENUMERATION"` and `"enumeration_category": "Your_Category"`
3. **Generate**: The existing enumeration engine handles iteration, prompt building, and tagging automatically

For techniques requiring custom iteration logic, subclass `ModePlugin`, implement `extract_items()`, `build_prompt()`, and `tag_payload()`, and drop the file into `core/backend/plugins/`. The Plugin Builder GUI automates this process.

---

## License

[License information here]

## Citation

[Citation information here]
