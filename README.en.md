# Post-Anesthesia Follow-up Agent · 麻醉术后访视助手

> A local-first AI agent for structured post-anesthesia follow-up visits and a medical knowledge base.
> Evolved from an early COZE multi-agent workflow prototype into a locally deployable full-stack application (FastAPI + Vue3) for research, teaching and personal technical showcase.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Backend: FastAPI](https://img.shields.io/badge/Backend-FastAPI-009688.svg)](https://fastapi.tiangolo.com/)
[![Frontend: Vue3](https://img.shields.io/badge/Frontend-Vue3-42b883.svg)](https://vuejs.org/)
[![LLM: Ollama](https://img.shields.io/badge/LLM-Ollama-000000.svg)](https://ollama.com/)

**⚠️ Disclaimer: this project is NOT a clinically validated or regulatory-approved medical device and must not replace the judgment of healthcare professionals. All reports are drafts that require clinical review. Use synthetic data and fully de-identified screenshots for any demonstration.**

## Overview

Built for the 24-hour postoperative follow-up scenario, this project integrates a local Ollama LLM, retrieval-augmented generation (RAG), a deterministic follow-up state machine and DOCX report generation into a deployable web application:

**Patient information collection → MEWS early-warning scoring → QoR-15 recovery-quality assessment → complication screening → wound assessment → structured visit report.**

The research background and the multidimensional clinical evaluation (a 150-case three-arm controlled study) are documented in `docs/`.

> **This is a showcase repository**: it contains documentation, architecture diagrams and screenshots only — **no runnable source code**. The full source code is maintained in a private repository; see "Source Code Access" below.

## Showcase

| Local System Flow | AI Agent Loop |
|---|---|
| ![flow](assets/local_system_flow.png) | ![loop](assets/agent_loop.png) |

| COZE Multi-Agent Prototype Architecture | Structured Visit Report (synthetic demo) |
|---|---|
| ![arch](assets/multi_agent_architecture.png) | ![report](assets/visit_report_demo.png) |

> The agent-loop and multi-agent architecture diagrams come from the research-stage materials; all patient information in the screenshots is synthetic example data.

## Key Features

- **Local models & knowledge base**: Ollama chat / embedding models, local Chroma vector store, BM25 hybrid retrieval with optional reranking — data never leaves the machine.
- **Multi-format document ingestion**: PDF / DOCX / TXT / Markdown / PPTX / XLSX / HTML; scanned PDFs supported via offline OCR (RapidOCR).
- **Controlled workflow**: a deterministic state machine drives staged follow-up — baseline info & MEWS → three rounds of QoR-15 → complication screening → wound assessment → structured report.
- **Local management UI**: chat, knowledge base, patient profiles, visit records and settings, with REST/SSE dual-channel interfaces (token-level streaming).
- **Report output**: downloadable per-session DOCX visit-report drafts for clinical review.

## Tech Stack

| Layer | Technologies |
| --- | --- |
| Frontend | Vue 3, Vite, Vue Router, Pinia, Tailwind CSS |
| Backend | Python, FastAPI, Pydantic, Uvicorn |
| Models | Ollama, LangChain |
| RAG | Chroma, rank-bm25, Sentence Transformers |
| Documents | PyMuPDF, python-docx, python-pptx, openpyxl, RapidOCR |

## Source Code Access

This is a **showcase repository**: it contains only documentation, architecture diagrams and screenshots — **no runnable source code**.

- The full source code (FastAPI backend + Vue3 frontend + deployment scripts) is maintained in a **private repository**. Because the related paper is under review and a patent is pending, the source code is not publicly downloadable for now.
- For academic exchange, collaboration, or to request source-code access, please contact **pillar_dl@163.com** with your affiliation and intended use. Approved requests receive collaborator access to the private repository.

## Documentation

Full system architecture, module design, API and deployment notes: [docs/GITHUB_PROJECT_DOCUMENTATION.md](docs/GITHUB_PROJECT_DOCUMENTATION.md); research-stage feature notes: [docs/README.legacy.zh-CN.md](docs/README.legacy.zh-CN.md). Tech stack: Python / FastAPI / Ollama / LangChain / Chroma / Vue 3 / Vite / Tailwind CSS.

## Privacy & Security

- The repository contains no real patient data, hospital-internal materials, knowledge-base source texts, sessions, reports, vector stores, logs, `.env` files or model weights.
- A manual review was performed against `.gitignore` before publishing; please follow the same medical-data compliance requirements when deploying on your own.

## License

[MIT](LICENSE)

---

*[中文 README](README.md)*
