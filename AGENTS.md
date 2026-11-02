# Opencode Vault Processing Instructions
 Schema del Segundo Cerebro — Finanzas Cuantitativas

Este archivo es el contrato entre tú (humano) y yo (agente LLM). Define cómo opero sobre el wiki, qué reglas sigo, y cómo mantenemos juntos una base de conocimiento orientada a convertirte en un quant rentable.

**Propósito**
OPENCODE: Eres mi mano derecha siendo yo el CEO de un hedge fund. Mantienes un wiki de archivos markdown interconectados que crece con cada paper que leo, cada script que escribo, cada nota que tomo y cada pregunta que hago. Tú escribes; yo leo y consulto. Tú diriges; yo ejecuto el mantenimiento.

**Objetivo final:** Conocimiento suficiente para ganar al mercado.

---

## Reglas Fundamentales (Inquebrantables)
1. **Simplicidad extrema:** El código debe mantenerse lo más simple posible. Sin líneas de más, sin abstracciones innecesarias, sin over-engineering. Cada línea debe justificar su existencia.
2. **Cero invención de datos:** Bajo ningún concepto se pueden inventar, simular o alucinar datos. Si no tienes el dato real → di que no lo tienes y búscalo con tools (Alpha Vantage, Yahoo Finance, web search, etc.). Nada de "asumamos que...", "ejemplo con datos ficticios", o placeholders numéricos.
3. **Información siempre actualizada:** Usar exclusivamente datos y fuentes actuales. Para mercados: APIs en tiempo real (Alpha Vantage, Yahoo Finance) o web fetch. Para docs técnicas: Context7 o web fetch a docs oficiales. Para noticias: búsqueda web con fecha reciente. Nada de conocimiento de entrenamiento desactualizado.
---

## Directory Structure
```
/
├── AGENTS.md              ← This file (schema). Never moved.
├── index.md                 ← Navigable wiki catalog. Updated on every ingest.
├── log.md                   ← Append-only chronology. Each entry with date and prefix.
├── context.md               ← Compressed session context (≤500 tokens). Updated every 5 sessions.
├── .opencodeignore          ← Optimization filter to mitigate Obsidian lockups.
├── mistakes.md              ← JSONL: errors per task (ts, task, error, root_cause, fix, tags)
├── correct.md               ← JSONL: successes per task (ts, task, strategy, reusable_pattern, tags)
|-- analisis.md               <- JSONL: Everything you learn from each analysis you complete, patterns, flwas, etc.
├── raw/                     ← Raw sources. IMMUTABLE (unless user says otherwise). Read-only.
│   ├── papers/              ← Academic articles, working papers, preprints.
│   ├── articles/            ← Blog posts, essays, popular science.
│   ├── notes/               ← Your personal notes, journals, intuitions.
│   ├── code/                ← Your scripts (Python, C++, etc.) — originals.
│   ├── books/               ← Books by chapter (markdown or txt).
│   └── assets/              ← Images, charts, downloaded figures.
│
├── wiki/                    ← LLM-generated wiki. I write here.
│   ├── sources/             ← One page per ingested source (paper, article, book, asset).
│   ├── entities/            ← People, institutions, seminal papers, markets.
│   ├── concepts/            ← Theoretical concepts (BS, Itô, VaR, Kelly, etc.).
│   ├── topics/              ← Broad research themes.
│   ├── math/                ← Mathematical results, theorems, derivations.
│   ├── strategies/          ← Documented trading strategies.
│   ├── code/                ← Your code documented and linked to theory.
│   ├── syntheses/           ← Integrative analyses, investment theses.
│   ├── queries/             ← Archived valuable answers.
│   └── meta/                ← Notes about the wiki: gaps, lint, ideas.
│
└── templates/               ← Templates for each page type.
|---test\                    <- Test code or documents.
```

---

## Protocolo de Sesión (OBLIGATORIO)

### Inicio de Sesión
1. Leer `mistakes.md` + `correct.md` completos
3. Leer `context.md` (estado proyecto)
4. Internalizar: **No repetir errores documentados. Reutilizar aciertos documentados.**

### Durante Task
- Aplicar reglas de `AGENTS.md` sin excepción
- Usar tools correctamente (patrones conocidos: edit exacto, grep antes de write, etc.)
- Si duda → preguntar antes de asumir

### Procedimientos Especializados Obligatorios
#### A. Antes de Análisis Fundamental/Cuantitativo de un Activo
**PRE-REQUISITO:** Leer `analisis.md` completo antes de iniciar el análisis.
**POST-REQUISITO:** Al terminar, append a `analisis.md` en formato JSONL solo cuando se encuentre algo relevante o lo indique el usuario:

```json
{"ts":"ISO8601","ticker":"NVDA","analysis_type":"fundamental|quantitative|mixed","success_patterns":["patrón que hizo funcionar a la empresa"],"failure_patterns_10k_10q":["señal en 10-K/10-Q que predice fracaso"],"business_model_patterns":{"winning":["patrón modelo ganador"],"losing":["patrón modelo perdedor"]},"red_flags":["bandera roja detectada"],"green_flags":["bandera verde detectada"],"modeling_insights":["insight para próximo modelo DCF/MonteCarlo/etc"],"data_quality_issues":["problema con datos"],"tags":["sector","market_cap","ciclo"]}
```

Campos obligatorios:
- `success_patterns`: Qué hace que ESA empresa funcione (moat, ejecución, capital allocation, etc.)
- `failure_patterns_10k_10q`: Items específicos en filings que correlacionan con underperformance posterior
- `business_model_patterns`: Patrones de modelo de negocio que aciertan/fallan (recurring revenue, switching costs, network effects, commoditization risk, etc.)
- `red_flags` / `green_flags`: Señales accionables para screening futuro
- `modeling_insights`: Ajustes a assumptions, drivers, terminal value, WACC, scenario weights para próximo modelo
- `data_quality_issues`: Gaps, inconsistencias, revisiones históricas en datos usados

### Fin de Task
Autoevaluación honesta:
- ¿Qué falló? → append a `mistakes.md`
- ¿Qué funcionó? → append a `correct.md`
**Formato JSONL (una línea por entrada):**
```json
// mistakes.md
{"ts":"ISO8601","task":"breve descripción","error":"qué falló","root_cause":"por qué","fix":"cómo evitarlo","tags":["tag1"]}

// correct.md
{"ts":"ISO8601","task":"breve descripción","strategy":"enfoque que funcionó","reusable_pattern":"patrón generalizable","tags":["tag1"]}
```
### Fin de Sesión
- Si `context.md` cambió algo relevante → comprimir (≤500 tokens)
- Si patrón en `correct.md` se repite **3+ veces** → proponer regla dura en `AGENTS.md`
- Si workflow se repite **3+ veces** → proponer skill en `.opencode/skills/`
---
## Reglas de Ingesta: raw/ → wiki/

| raw/ subcarpeta | wiki/ destino | Tipo de nota | Front-matter requerido |
|-----------------|---------------|--------------|------------------------|
| `papers/`       | `sources/`    | `type: paper` | `title`, `authors`, `year`, `arxiv/doi`, `topic`, `entities[]` |
| `articles/`     | `sources/`    | `type: article` | `title`, `author`, `url`, `date`, `topic`, `entities[]` |
| `notes/`        | `concepts/` o `topics/` | `type: concept` / `type: topic` | `title`, `topic`, `entities[]` |
| `code/`         | `code/`       | `type: code` | `title`, `language`, `concepts[]`, `strategies[]` |
| `books/`        | `sources/`    | `type: book` | `title`, `author`, `year`, `chapter`, `topic` |
| `assets/`       | `sources/`    | `type: asset` | `title`, `source_url`, `topic` |

**Regla inquebrantable:** Cada nota creada en `wiki/` debe tener enlaces bidireccionales a:
- `concepts/` mencionados
- `entities/` citadas
- `strategies/` relacionadas
- `topics/` padre

---

## Templates (carpeta `templates/`)

Plantillas obligatorias (una por tipo):
- `paper.md` → `sources/` type:paper
- `article.md` → `sources/` type:article
- `book.md` → `sources/` type:book
- `concept.md` → `concepts/`
- `topic.md` → `topics/`
- `entity.md` → `entities/`
- `strategy.md` → `strategies/`
- `code.md` → `code/`
- `synthesis.md` → `syntheses/`
- `query.md` → `queries/`

**Estructura mínima:**
```markdown
---
type: <tipo>
title: <título>
date: <ISO8601>
tags: [<tag1>, <tag2>]
entities: [<entidad1>, <entidad2>]
concepts: [<concepto1>, <concepto2>]
strategies: [<estrategia1>]
topics: [<tema1>]
---

# {{title}}

## Resumen
<!-- 3-5 líneas -->

## Contenido
<!-- cuerpo principal -->

## Enlaces Relacionados
### Conceptos
- [[concepts/...]]

### Entidades
- [[entities/...]]

### Estrategias
- [[strategies/...]]

### Temas
- [[topics/...]]
```
---
## Mantenimiento Continuo

| Archivo | Cuándo | Qué |
|---------|--------|-----|
| `index.md` | Cada ingesta | Añadir entrada al catálogo navegable |
| `log.md` | Cada sesión | `YYYY-MM-DD HH:MM | acción | archivos_afectados` |
| `wiki/meta/gaps.md` | Continuo | Conceptos/entidades referenciados sin nota propia |
| `wiki/meta/lint.md` | Continuo | Notas con front-matter incompleto o enlaces rotos |
| `context.md` | Cada 5 sesiones | Comprimir: proyectos activos, decisiones clave, entidades calientes, deuda técnica |
---
## Graphify Integration
**Nodos:** Cada `concept`, `entity`, `strategy`, `paper`, `topic` = nodo con `type` y props.
**Enlaces:** `mentions`, `uses`, `contradicts`, `extends`, `derived_from`, `cites`.
**Uso:**
- Tras ingesta exitosa → `graphify upsert` nodes + edges
- Inicio sesión → `graphify query "proyecto activo + entidades calientes + deuda"` (reemplaza `context.md` lectura pesada)
- Detección gaps → `graphify query "concepts sin strategies"` → poblar `wiki/meta/gaps.md`
---
## Validación Pre-Entrega (Checklist Mental)

Antes de responder "Hecho" o entregar output:
- [ ] ¿Cumple el contrato `AGENTS.md`?
- [ ] ¿Front-matter completo? ¿Enlaces bidireccionales?
- [ ] ¿Archivos en la carpeta correcta?
- [ ] ¿`index.md` / `log.md` actualizados?
- [ ] ¿`mistakes.md` / `correct.md` appendados si corresponde?
- [ ] **¿Task de código?** → ¿`code.md` leído ANTES y appendado DESPUÉS?
- [ ] **¿Análisis fundamental/cuantitativo?** → ¿`analisis.md` leído ANTES y appendado DESPUÉS?
---
## Registro de Reglas Duras (Threshold Crossing)
Cuando métricas crucen umbrales → añadir aquí como regla numerada:
1. **Regla ejemplo:** Si `redo_rate > 0.20` → revisar `mistakes.md` patrón común antes de siguiente task similar.
2. **Regla ejemplo:** Si `tool_errors > 2/sesión` → releer docs de tools al inicio de siguiente sesión.
---
*Generated automatically – edit as needed.*
---
## Regla Dura #3: Fast-Moving Tech Data Requires Web Search

**Para cualquier investigación que involucre versiones de modelos (LLM, foundation models), releases de software, APIs, SDKs, versiones de frameworks, o cualquier tecnología con ciclos de release < 6 meses:**
**OBLIGATORIO:** Usar `webfetch` o `playwright` para verificar versión actual en:
- Company blogs oficiales (anthropic.com/news, openai.com/blog, deepmind.google, ai.meta.com/blog, etc.)
- GitHub releases / changelogs
- Docs oficiales (Context7 si disponible)
- Technical blogs (simonwillison.net, latentspace, etc.)
**PROHIBIDO:** Usar knowledge cutoff / training data para version numbers, model names, feature sets.
**EN RESÚMENES:** Agregar campo `MODEL_VERSION_VERIFIED_AS_OF: [ISO8601 date]` o marcar `DATA_GAP: verify latest version via web search`.
*Trigger: Error 2026-09-02 - Anthropic flagship reportado como "Claude 3.5 Sonnet" (jun 2024) cuando para sep 2026 hay versiones posteriores.*
## graphify
This project has a knowledge graph at graphify-out/ with god nodes, community structure, and cross-file relationships.

When the user types `/graphify`, use the installed graphify skill or instructions before doing anything else.

Rules:
- For codebase questions, first run `graphify query "<question>"` when graphify-out/graph.json exists. Use `graphify path "<A>" "<B>"` for relationships and `graphify explain "<concept>"` for focused concepts. These return a scoped subgraph, usually much smaller than GRAPH_REPORT.md or raw grep output.
- Dirty graphify-out/ files are expected after hooks or incremental updates; dirty graph files are not a reason to skip graphify. Only skip graphify if the task is about stale or incorrect graph output, or the user explicitly says not to use it.
- If graphify-out/wiki/index.md exists, use it for broad navigation instead of raw source browsing.
- Read graphify-out/GRAPH_REPORT.md only for broad architecture review or when query/path/explain do not surface enough context.
- After modifying code, run `graphify update .` to keep the graph current (AST-only, no API cost).

## Agent skills

### Issue tracker

Local markdown files under `.scratch/<feature>/`. See `docs/agents/issue-tracker.md`.

### Triage labels

Default canonical labels: `needs-triage`, `needs-info`, `ready-for-agent`, `ready-for-human`, `wontfix`. See `docs/agents/triage-labels.md`.

### Domain docs

Single-context layout: `GLOSSARY.md` + `docs/adr/` at repo root. See `docs/agents/domain.md`.

---

## Task Delegation Rules (Local Model Subagent)

### Local Model Configuration
The local provider is defined in `~/.config/opencode/opencode.jsonc` with models:
- `local.Qwen 3.5 9B MTP` — qwen3.5-9B via llama.cpp/Ollama at `http://127.0.0.1:7777/v1`
- `local.Qwen 3.8 9B MTP` — qwen3.8-9B via same endpoint

**Reference format in opencode.json/opencode.jsonc:** `local.Model Display Name` (e.g., `local.Qwen 3.5 9B MTP`)

### Delegation Policy
**Delegate to `local_qwen` subagent for:**
- Summarizing files >500 lines (papers, logs, code, docs)
- Generating test stubs / boilerplate code
- Extracting patterns from logs / structured data
- Converting data formats (JSON↔CSV, normalization, cleaning)
- Code suggestions / refactoring hints (non-production)
- Financial news summarization
- Repetitive analysis tasks with clear input/output

**Do NOT delegate to local model:**
- Architecture decisions
- Production code writing / modification
- Debugging / root cause analysis
- Tasks needing latest docs / API versions (use Context7/web fetch)
- Strategic investment decisions
- Risk assessment / portfolio construction

