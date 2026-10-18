<div align="center">

# Code2Skill

**Grounded synthesis of reusable procedural skills from source code**

[![Python](https://img.shields.io/badge/python-3.9%2B-3776AB?logo=python&logoColor=white)](pyproject.toml)
[![Website](https://img.shields.io/badge/project-website-1f6feb)](https://ant-international-research.github.io/developer-skill-hubs/)
[![Dataset](https://img.shields.io/badge/%F0%9F%A4%97%20dataset-DeveloperSkills--Code2Skill-ffcc4d)](https://huggingface.co/datasets/ant-intl/DeveloperSkills-Code2Skill)

[Pipeline](#pipeline) •
[Install](#install) •
[Quick start](#run-against-a-model-endpoint) •
[Offline demo](#offline-end-to-end-example) •
[Output contract](#output-contract) •
[Resources](#resources)

</div>

---

Code2Skill turns source-code units into grounded, reusable procedural skill
records. It extracts a structured skill, challenges that abstraction through
source-body-blind code regeneration, checks the regenerated behavior against
the source, and retains accepted records with provenance and retrieval-facing
metadata.

This repository is a publication-oriented implementation aligned with
the paper **Grounded Skill Synthesis from Code at Scale for Agentic
Intelligence**.

## Pipeline

| # | Stage | What it does |
|:-:|-------|--------------|
| 1 | **Source-unit tagging** | Scores parsed functions, methods, command entry points, and file-level components for reusable procedural evidence. |
| 2 | **Skill extraction** | Emits one typed record: `atomic`, `composite`, or `pattern`. |
| 3 | **Source-body-blind regeneration** | Receives the skill and interface, but not the source body, repository name, or file path. |
| 4 | **Source-aware equivalence judgment** | Compares source and regenerated code. |
| 5 | **Adjudication** | Separates an unsupported skill from a failed regeneration. |
| 6 | **Feature tagging** | Creates one retrieval record per accepted skill using closed vocabularies. |
| 7 | **Purpose indexing** | Deterministically removes low-value records, groups records by `(task_family, intent_action, intent_target)`, and selects an existing representative for each group. |

The judge is an LLM consistency check, not a proof of program equivalence.

## Install

Requires **Python 3.9+**.

```bash
python3 -m venv .venv
. .venv/bin/activate
python -m pip install --upgrade pip
python -m pip install -e .
```

The runtime has no third-party Python dependency. It calls an
OpenAI-compatible `/chat/completions` endpoint through the standard library.
It has no telemetry and does not upload repository content anywhere except the
model endpoint explicitly configured for a run.

## Run against a model endpoint

```bash
cp .env.example .env
# Fill the two values in .env, then load them into the current shell.
set -a
. ./.env
set +a

code2skill run /path/to/repository \
  --base-url "$CODE2SKILL_MODEL_BASE_URL" \
  --model your-model-name \
  --output-dir runs
```

Public model hosts are rejected by default. Add `--allow-public-model-url`
only when the endpoint is intentionally trusted.

### Other commands

| Command | Purpose |
|---------|---------|
| `code2skill run <input>` | Run the end-to-end pipeline on a repository or a directory of repositories. |
| `code2skill show-prompts --stage <stage>` | Inspect the model-visible messages of one stage without making a model call. |
| `code2skill purpose-index <in> <out>` | Build a new purpose index from accepted-record JSONL. |

Inspect the regeneration prompt, for example:

```bash
code2skill show-prompts --stage regeneration
```

Available stages: `source_tagging`, `extraction`, `regeneration`,
`equivalence`, `adjudication`, `feature_tagging`.

Build a purpose index:

```bash
code2skill purpose-index accepted_records.jsonl purpose_index.jsonl
```

## Offline end-to-end example

The deterministic example client exercises every stage without network access:

```bash
PYTHONPATH=src python examples/offline_demo.py
```

It writes a complete run under `examples/output/`, including decisions,
accepted records, retrieval records, prompt/response traces, and a purpose
index.

## Output contract

Each run directory contains:

```text
runs/
├── manifest.json
├── summary.json
├── source_units.jsonl
├── decisions.jsonl
├── accepted_records.jsonl
├── rejected_records.jsonl
├── retrieval_records.jsonl
├── purpose_index.jsonl
├── purpose_edges.jsonl
└── traces/<data_id>/<stage>.json
```

`accepted_stage` is `equivalence` for direct accepts and `adjudication` for
adjudicated accepts. In the latter case, the skill is retained but the failed
regenerated code is not an accepted artifact.

### Documentation

| Document | Contents |
|----------|----------|
| [docs/methodology.md](docs/methodology.md) | Paper-to-code mapping |
| [docs/data-contract.md](docs/data-contract.md) | Record schemas |
| [docs/open-source-checklist.md](docs/open-source-checklist.md) | Publication gates |

## Resources

- 🌐 [Project website](https://ant-international-research.github.io/developer-skill-hubs/)
- 🤗 [DeveloperSkills-Code2Skill dataset](https://huggingface.co/datasets/ant-intl/DeveloperSkills-Code2Skill)
- 📋 [Source repository list](additional/code2skill_repo_info.csv)

## Development

```bash
PYTHONPATH=src python -m unittest discover -s tests -v
python -m compileall -q src tests examples
```

See [CONTRIBUTING.md](CONTRIBUTING.md) for contribution guidelines and
[SECURITY.md](SECURITY.md) for security considerations.

## Citation

If you use Code2Skill, please cite the accompanying paper:

```bibtex
@article{tong2026grounded,
  title={Grounded Skill Synthesis from Code at Scale for Agentic Intelligence},
  author={Tong, Yongqi and Wang, Pan and Wang, Hang and Li, Jianshe and Zhang, Xin and Yang, Jiang-Ming and Wu, Wei},
  journal={arXiv preprint arXiv:2609.05571},
  year={2026}
}
```
