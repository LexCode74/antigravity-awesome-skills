# Loki Mode - Claude Code Skill

Multi-agent autonomous startup system for Claude Code. Takes PRD to fully deployed, revenue-generating product with zero human intervention.

## Quick Start

```bash
# Launch Claude Code with autonomous permissions
claude --dangerously-skip-permissions

# Then invoke:
# "Loki Mode" or "Loki Mode with PRD at path/to/prd"
```

## Project Structure

```
SKILL.md                    # Main skill definition (read this first)
references/                 # Detailed documentation (loaded progressively)
  openai-patterns.md        # OpenAI Agents SDK: guardrails, tripwires, handoffs
  lab-research-patterns.md  # DeepMind + Anthropic: Constitutional AI, debate
  production-patterns.md    # HN 2025: What actually works in production
  advanced-patterns.md      # 2025 research patterns (MAR, Iter-VF, GoalAct)
  tool-orchestration.md     # ToolOrchestra-inspired efficiency & rewards
  memory-system.md          # Episodic/semantic memory architecture
  quality-control.md        # Code review, anti-sycophancy, guardrails
  agent-types.md            # 37 specialized agent definitions
  sdlc-phases.md            # Full SDLC workflow
  task-queue.md             # Queue system, circuit breakers
  spec-driven-dev.md        # OpenAPI-first development
  architecture.md           # Directory structure, state schemas
  core-workflow.md          # RARV cycle, autonomy rules
  claude-best-practices.md  # Boris Cherny patterns
  deployment.md             # Cloud deployment instructions
  business-ops.md           # Business operation workflows
  mcp-integration.md        # MCP server capabilities
autonomy/                   # Runtime state and constitution
benchmarks/                 # SWE-bench and HumanEval benchmarks
```

## Key Concepts

### RARV Cycle
Every iteration follows: **R**eason -> **A**ct -> **R**eflect -> **V**erify

### Model Selection
- **Opus**: Planning and architecture ONLY (system design, high-level decisions)
- **Sonnet**: Development and functional testing (implementation, integration tests)
- **Haiku**: Unit tests, monitoring, and simple tasks - use extensively for parallelization

### Quality Gates
1. Static analysis (CodeQL, ESLint)
2. 3-reviewer parallel system (blind review)
3. Anti-sycophancy checks (devil's advocate on unanimous approval)
4. Severity-based blocking (Critical/High/Medium = BLOCK)
5. Test coverage gates (>80% unit, 100% pass)

### Memory System
- **Episodic**: Specific interaction traces (`.loki/memory/episodic/`)
- **Semantic**: Generalized patterns (`.loki/memory/semantic/`)
- **Procedural**: Learned skills (`.loki/memory/skills/`)

### Metrics System (ToolOrchestra-inspired)
- **Efficiency**: Task cost tracking (`.loki/metrics/efficiency/`)
- **Rewards**: Outcome/efficiency/preference signals (`.loki/metrics/rewards/`)

## Development Guidelines

### When Modifying SKILL.md
- Keep under 500 lines (currently ~370)
- Reference detailed docs in `references/` instead of inlining
- Update version in header AND footer
- Update CHANGELOG.md with new version entry

### Version Numbering
Follows semantic versioning: MAJOR.MINOR.PATCH
- Current: v2.35.0
- MINOR bump for new features
- PATCH bump for fixes

### Code Style
- No emojis in code or documentation
- Clear, concise comments only when necessary
- Follow existing patterns in codebase

## Testing

```bash
# Run benchmarks
./benchmarks/run-benchmarks.sh humaneval --execute --loki
./benchmarks/run-benchmarks.sh swebench --execute --loki
```

## Research Foundation

Built on 2025 research from three major AI labs:

**OpenAI:**
- Agents SDK (guardrails, tripwires, handoffs, tracing)
- AGENTS.md / Agentic AI Foundation (AAIF) standards

**Google DeepMind:**
- SIMA 2 (self-improvement, hierarchical reasoning)
- Gemini Robotics (VLA models, planning)
- Dreamer 4 (world model training)
- Scalable Oversight via Debate

**Anthropic:**
- Constitutional AI (principles-based self-critique)
- Alignment Faking Detection (sleeper agent probes)
- Claude Code Best Practices (Explore-Plan-Code)

**Academic:**
- CONSENSAGENT (anti-sycophancy)
- GoalAct (hierarchical planning)
- A-Mem/MIRIX (memory systems)
- Multi-Agent Reflexion (MAR)
- NVIDIA ToolOrchestra (efficiency metrics)

See `references/openai-patterns.md`, `references/lab-research-patterns.md`, and `references/advanced-patterns.md`.


---

## Directiva Norte y Anti-Alucinación (innegociable — agregada 2026-05-03)

### Filtro Norte — toda decisión pasa por aquí

Antes de implementar / recomendar / gastar / publicar, pasar por el filtro:

1. ¿Colma un **ANHELO** de Manuel o de los buyer-personas de este subproyecto?
2. ¿Resuelve un **MIEDO** de Manuel o de los BPs?
3. ¿Previene un **MOMENTO DE ABANDONO** de los BPs?

Si las 3 son NO → cuestionar severamente, postergar o descartar.

**BPs específicos del subproyecto:** documentar en la sección `Buyer Personas` de este CLAUDE.md. Si está vacía, heredar BP1 (Ingeniero Curioso 35-50) + BP2 (Joven Técnico 24-34) del `Antigravity-Proyectos/CLAUDE.md` global.

### Anti-alucinación — consulta NotebookLM en decisiones críticas

**Antes de ejecutar/recomendar cualquier cosa que implique:**

- 💸 **Gasto de dinero** (ads, herramientas IA, infra, contratación, compra de assets)
- 🔒 **Seguridad informática** (auth, secrets, RLS, headers, vulnerabilidades) o **de usuarios** (datos personales, GDPR, Hotmart compliance)
- ⏰ **Gasto de tiempo significativo** (>2h, decisiones arquitectónicas, refactors grandes)
- 🎯 **Eficacia / efectividad de solución** (afirmaciones tipo "esto convierte mejor", "esto es la best practice", "esto es viral")

→ **Consultar primero NotebookLM:**

```bash
/home/lexcode74/miniforge3/envs/notebooklm/bin/python \
  /home/lexcode74/Antigravity-Proyectos/notebooklm-skill/scripts/nlm.py \
  ask --notebook-id <ID-relevante> "pregunta específica"
```

**Notebooks principales (validar IDs con `nlm.py list`):**

| Tema | Notebook ID | Cubierto por skill (SÍ/NO) |
|------|-------------|----------------------------|
| Meta Ads, hyperoptimización campañas | `ee60c6bf-3037-4a06-96f5-e063019cfba1` | ✅ `growth-marketing-meta` con NOTEBOOKLM-ALWAYS |
| Videos virales — estrategia y benchmarks | `7869ed80-3346-4103-b5a8-98f17a947ac8` | ⚠️ skill `seedance` cubre técnica, NO viralidad |
| Email marketing info-products técnicos | `6e7a2ccd-29bc-45b6-94d7-3964d27fe67f` | ⚠️ skill `email-marketing-omnisend` cubre edición, NO copy/deliverability strategy |
| Campañas Automatizadas multi-canal | `ee60c6bf-3037-4a06-96f5-e063019cfba1` | ✅ `growth-marketing-meta` |
| Videos virales largos 10min | `0d61fa52-f35c-4437-a050-3ab1c187516f` | — |
| Agente Contenido Arquitectura | `2a5d8852-d5eb-4661-85f6-e0d5e8c5855e` | — |
| AI Social media automation | `ad5aca05-c6c5-4cef-b955-307c0788cce7` | — |

Si el tema no tiene notebook dedicado → declarar explícitamente *"inferencia no verificada"* antes de proceder.

**Razón (cita Manuel 2026-04-26):** *"A mí me interesa que cumplamos los pinches objetivos, no tener la razón. Las cosas que digo pueden llegar a tener errores, así que siempre antes de implementar debes consultar NotebookLM."*

---

## 🛑 DIRECTIVA NO-INVENTAR INFRAESTRUCTURA (canon · 2026-05-24 · hereda del raíz)

> Antes de afirmar nada sobre DNS · hosting · registrador · proveedor · plan contratado · CDN · capacidades de Vercel/Hotmart/Meta/Supabase/etc. de Manuel: verificar con comando reproducible (`getent hosts`, `curl`, `gh`, WebFetch a docs vivos, NotebookLM) o preguntar UNA línea concreta. NUNCA escribir "tu hosting" · "tu DNS" · "tu proveedor" como si supieras cuál es. Reportar verificación con output literal, no paráfrasis. Detalle completo + anti-patrón en `~/Antigravity-Proyectos/CLAUDE.md` § Directiva Global No-Inventar Infraestructura.
