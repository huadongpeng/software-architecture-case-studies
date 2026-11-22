# Software Architecture Case Studies / 企业级软件架构案例复盘

This repository contains structured analyses of significant enterprise projects I have architected and led as a technical lead. The goal is to document real problem-solving — the constraints, tradeoffs, and engineering decisions — without exposing proprietary code.

> **中文导览**：本仓库整理了两个代表性企业项目的英文架构复盘（均已脱敏），按「业务约束 → 个人职责 → 关键架构决策 → 最大挑战 → 可量化结果」结构呈现：
> 1. **国家级农村土地管理多租户 SaaS 平台**（独立租户数据库隔离、低代码表单/流程引擎、异步电子签章解耦、统一鉴权）
> 2. **企业内部 LLM 知识库系统**（RAG 混合检索、按章节分块、强制来源引用、检索层权限隔离与 8s → <2s 性能优化）
>
> 更多近期 AI 应用工程案例与可运行本地恢复样例，见 👉 [ai-application-portfolio](https://github.com/huadongpeng/ai-application-portfolio)。

Each case study follows the same format: business context → my role → architectural decisions → hardest challenge → measurable outcome.

---

## 📋 Case Studies

1. [National-Level Rural Land Management SaaS Platform](#case-study-1-national-level-rural-land-management-saas-platform)
2. [Enterprise Internal Knowledge Base with LLM Integration](#case-study-2-enterprise-internal-knowledge-base-with-llm-integration)

---

## Case Study 1: National-Level Rural Land Management SaaS Platform

### 🎯 Business Problem

A national-level project to digitize the entire lifecycle of rural land management across multiple provinces in China. The core challenge: build a highly flexible, multi-tenant SaaS platform that different provincial governments could adapt to local regulations, while ensuring strict data isolation and high availability.

### 👤 My Role

Sole technical authority — requirements analysis, architecture design, team coordination, deployment, and ongoing governance.

### 🏗️ Key Architectural Decisions

- **Database-per-tenant multi-tenancy** — mandatory for government data isolation requirements. Schema-sharing was ruled out on day one.
- **Low-code customization engine** — built a custom runtime for dynamic forms, configurable workflows (Activiti), and report templates (Freemarker). This let non-technical administrators configure business processes without dev involvement, cutting secondary development cost by ~50%.
- **Decoupled e-signature integration** — integrated a legally compliant electronic seal system via async message queues. Third-party failures could not cascade into the core application. Document integrity verified with cryptographic hashes.
- **Unified auth layer** — Spring Security + JWT across all tenants and modules, with fine-grained RBAC.

### 💡 Hardest Challenge

The provincial rollout requirement: each region had different field schemas, approval workflows, and reporting formats. A rigid data model would have required a new deployment per province. The low-code engine solved this, but building a reliable, extensible runtime from scratch — not a product — was the riskiest part of the project.

### 📈 Outcome

- Platform adopted as a benchmark for similar national digitalization projects
- ~70% faster rollout in new pilot regions vs. the previous monolithic system
- Formal commendation from the client for system stability

---

## Case Study 2: Enterprise Internal Knowledge Base with LLM Integration

### 🎯 Business Problem

A company's institutional knowledge was scattered: historical project documents in shared drives, internal wikis that nobody updated, and expertise locked in individuals' heads. New engineers took 2–3 months to get productive. The goal was to build a searchable, AI-augmented knowledge base that could surface the right document — or the right person — from a natural language question.

### 👤 My Role

Sole technical lead — evaluated the approach, designed the pipeline, built the MVP, and handed off the maintenance playbook.

### 🏗️ Key Architectural Decisions

- **RAG over fine-tuning** — the knowledge base needed to be updatable without retraining a model. Retrieval-Augmented Generation (embed documents → store vectors → retrieve on query → generate answer) fit the constraint of a small team with no MLOps infrastructure.
- **Section-based chunking** — documents were split by section headings rather than fixed token windows. This preserved semantic coherence and significantly improved retrieval precision on structured technical documents.
- **Hybrid search** — combined dense vector search (semantic similarity) with BM25 keyword search (exact terms like API names, error codes). Neither alone was sufficient.
- **Mandatory source citations** — every answer returned with document links and excerpts. This was a deliberate design constraint: users needed to verify answers, not just trust them. It also made hallucinations immediately visible.
- **Access control at retrieval time** — documents inherited their original permissions. The search layer enforced ACL before returning chunks to the LLM, so the model never saw content the user wasn't authorized for.

### 💡 Hardest Challenge

Getting engineers to actually use it. The first MVP had good recall but slow response (~8s). Users tried it once and went back to Slack. Optimizing the pipeline to under 2 seconds — async chunked streaming + cached embeddings for frequently accessed docs — was the turning point for adoption.

### 📈 Outcome

- New engineer onboarding time reduced from ~10 weeks to ~6 weeks in the first 3 months of rollout
- ~60% of "where do I find X" Slack questions replaced by self-service queries
- Unexpected win: the system surfaced several deprecated internal APIs still referenced in new code — found by querying the codebase, not by manual review

---

## 🗒️ Format Reference

Each case study follows this structure:

| Section | Content |
| :--- | :--- |
| Business Problem | The actual constraint that made this hard |
| My Role | Scope of ownership |
| Architectural Decisions | The choices made and why, including what was ruled out |
| Hardest Challenge | The thing that could have killed the project |
| Outcome | Measurable results where possible |
