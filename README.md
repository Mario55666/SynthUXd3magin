# SynthUXd3magin v2.0
### Plataforma de Investigación UX con Usuarios Sintéticos · Open Source

[![PWA Ready](https://img.shields.io/badge/PWA-Ready-4f8ef7?style=flat-square&logo=pwa)](https://web.dev/progressive-web-apps/)
[![OCEAN](https://img.shields.io/badge/Modelo-OCEAN%20Big%20Five-7c5ef7?style=flat-square)](https://en.wikipedia.org/wiki/Big_Five_personality_traits)
[![ACT-R](https://img.shields.io/badge/Memoria-ACT--R-27d49a?style=flat-square)](http://act-r.psy.cmu.edu/)
[![Hofstede](https://img.shields.io/badge/Cultura-Hofstede%20CAT-f7b24f?style=flat-square)](https://geerthofstede.com/)
[![PRISMA 2020](https://img.shields.io/badge/Metodología-PRISMA%202020-f05e7a?style=flat-square)](https://www.prisma-statement.org/)

---

## ¿Qué es?

**SynthUXd3magin** es una plataforma web de código abierto que orquesta agentes de inteligencia artificial para simular cómo distintos tipos de usuarios interactúan con productos digitales — desde niños de 8 años hasta adultos mayores, en cuatro culturas distintas.

Simula sesiones de usabilidad mediante agentes cognitivos realistas basados en tres modelos psicológicos validados científicamente:

- **OCEAN / Big Five** — personalidad y comportamiento del agente
- **ACT-R** — arquitectura de memoria de trabajo realista (decay, chunks, spreading activation)
- **Hofstede + Hall** — dimensiones culturales que condicionan decisiones

Los agentes generan acciones JSON que un orquestador Python ejecuta en un navegador real mediante **Playwright**, produciendo reportes con journey maps, hallazgos priorizados y métricas de fricción.

> **Paridad conductual documentada: 85–92%** con usuarios reales en estudios controlados (Park et al., 2025 · PADO Framework).

---

## Características principales

### Motor de simulación
- Perfil cognitivo completo: OCEAN + ACT-R + Hofstede + JTBD + COM-B + Rogers
- 8 perfiles predefinidos (niños 8–11, adolescentes 12–17, adultos, adultos mayores)
- Anti-sycophancy activo: Amabilidad (A) ≤ 4 elimina el sesgo de validación del LLM
- **Chain-of-Feeling** (Synthetic Users, 2025): progresión emocional con rueda de Plutchik
- **Spreading Activation** (Zhao et al., 2026 ACM CHI): recuperación probabilística de memoria
- **σ variabilidad estocástica** por rasgo OCEAN (BIG5-CHAT · Bhandari et al., 2025)

### Modelos LLM soportados
| Modelo | Proveedor | Coste |
|--------|-----------|-------|
| GPT-4o mini | OpenAI | ~$0.02/estudio |
| Gemini 1.5 Flash | Google AI | Gratis (15 RPM) |
| Claude 3 Haiku | Anthropic | Trial gratuito |
| Llama 3.1 70B | Groq | 100% gratuito |
| Gemma 2 9B | Ollama local | Gratuito (local) |
| Mistral 7B | Ollama local | Gratuito (local) |

### Base de conocimiento RAG
- Indexación de entrevistas, tickets de soporte, CRM, encuestas NPS
- Recuperación semántica con similitud coseno
- **Agentic RAG** (Singh et al., 2025): re-consulta dinámica ante fricción detectada
- Compatible con OpenAI Embeddings, Google Embedding-001 y Nomic (Ollama local)

### Validación científica
- **Hofstede CAT** (Xu et al., 2025, COLING): valida alineación cultural del perfil antes de lanzar
- **GRADE evidence** (⊕⊕⊕⊕ a ⊕◯◯◯) en cada hallazgo
- **PISA 2022/2025** domain tags por tipo de hallazgo
- **WEIRD bias compensation** para perfiles latinoamericanos (Kumar et al., 2025)
- Aviso automático cuando Neuroticismo ≥ 7 (Zhu et al., 2025, Pearson r < 0.26)

### Exportación
- JSON completo con journey, transcript, scores y steps
- **CSV PRISMA 2020** — 27 ítems para reportes de investigación formal
- Markdown para Notion, Confluence o GitHub

### PWA — Progressive Web App
- Funciona sin conexión a internet (Service Worker con estrategia cache-first)
- Instalable en escritorio y dispositivos móviles
- Indicador de estado offline en tiempo real

---

## Inicio rápido

### Opción 1 — Solo navegador (sin backend)

1. Descarga `synthuxd3magin_v2_updated.html`
2. Ábrelo directamente en Chrome, Edge o Firefox
3. Ve a **Configuración** → añade tu clave de API (Groq es gratuito)
4. Ve a **Nuevo estudio** → configura URL + perfil → lanza simulación

No requiere instalación ni servidor. Las claves de API se guardan solo en tu navegador (`localStorage`).

### Opción 2 — Con orquestador Python (simulación completa con Playwright)

```bash
# Requisitos: Python 3.10+, pip
git clone https://github.com/tu-usuario/synthuxd3magin.git
cd synthuxd3magin/orchestrator

pip install -r requirements.txt
playwright install chromium

# Configura tu clave de API
cp .env.example .env
# Edita .env y añade OPENAI_API_KEY, ANTHROPIC_API_KEY, etc.

# Inicia el orquestador
uvicorn main:app --reload --port 8000
```

Abre `synthuxd3magin_v2_updated.html` en el navegador. En **Configuración**, el endpoint del orquestador apunta a `http://localhost:8000/api/v1` por defecto.

---

## Arquitectura del sistema

```
┌─────────────────────────────────────────────────────────┐
│                  NAVEGADOR (SPA)                        │
│   Dashboard · Estudio · Prompts · RAG · Personas        │
│   Resultados · Roadmap · Relevancia · Config            │
└────────────────────┬────────────────────────────────────┘
                     │ HTTP REST
                     ▼
┌─────────────────────────────────────────────────────────┐
│            ORQUESTADOR PYTHON (FastAPI)                  │
│                                                          │
│  1. Ensambla System Prompt (OCEAN + ACT-R + Hofstede)   │
│  2. Recupera contexto RAG (similitud coseno)            │
│  3. Envía request a API LLM                             │
│  4. Recibe JSON → ejecuta en Playwright                 │
│  5. Agrega métricas → genera reporte UX                 │
└──────────┬───────────────────────────┬──────────────────┘
           │                           │
           ▼                           ▼
┌──────────────────┐        ┌─────────────────────┐
│   API LLM        │        │  Playwright Browser  │
│  OpenAI / Gemini │        │  Headless Chromium   │
│  Claude / Llama  │        │  Screenshots + DOM   │
└──────────────────┘        └─────────────────────┘
```

### Flujo de simulación por paso

```
Sistema:  [System Prompt: identidad cognitiva completa]
               ↓
Usuario:  "Paso N: {estado DOM actual} — ¿qué haces?"
               ↓
Agente:   {"action":"click","target":".btn","emotion":"confused","friction":true}
               ↓
Playwright: ejecuta acción → captura screenshot → nuevo estado DOM
               ↓
Repetir hasta max_steps o tarea completada
```

---

## Perfiles cognitivos incluidos

| Perfil | Edad | País | Rogers | OCEAN destacado |
|--------|------|------|--------|-----------------|
| Sofía Ramírez | 10 | Colombia | Innovador | O=9, C=4, A=7 |
| Alejandro Torres | 9 | Perú | Mayoría temprana | O=8, N=7 |
| Mateo López | 14 | México | Innovador | O=8, E=8, A=4 |
| Valentina Cruz | 16 | Argentina | Early Adopter | O=9, A=4 |
| María García | 38 | Perú | Early Adopter | C=9, A=3 ⚠ anti-syco |
| Carlos Mendoza | 52 | México | Mayoría tardía | A=8, N=6 |
| Lucía Torres | 26 | Argentina | Innovador | O=9, A=3 ⚠ anti-syco |
| Miguel Fernández | 68 | España | Rezagado | C=8, N=7 |
| Esperanza Vargas | 71 | Chile | Mayoría tardía | C=9, A=8 |

> **Perfiles infantiles** (≤ 12 años) incluyen restricciones cognitivas Piaget automáticas: estadio operacional concreto, chunks ACT-R reducidos a 3, atención sostenida máxima 8–10 min.

---

## Formato de respuesta del agente

Cada paso de la simulación produce un objeto JSON:

```json
{
  "step": 4,
  "action": "click | type | scroll | navigate | observe",
  "target": "selector CSS o descripción del elemento",
  "value": "texto a escribir si action=type",
  "thought": "razonamiento en primera persona del agente",
  "emotion": "neutral | confused | frustrated | satisfied | lost",
  "friction": true,
  "friction_reason": "filtro de precio no visible above the fold"
}
```

---

## Variables del System Prompt

| Variable | Descripción |
|----------|-------------|
| `{{persona_name}}` | Nombre del perfil simulado |
| `{{O}}` `{{C}}` `{{E}}` `{{A}}` `{{N}}` | Valores OCEAN 1–10 |
| `{{decay}}` `{{wm_chunks}}` `{{threshold}}` | Parámetros ACT-R |
| `{{pdi}}` `{{idv}}` `{{uai}}` `{{hal}}` | Dimensiones Hofstede |
| `{{jtbd}}` | Jobs-to-be-Done del perfil |
| `{{rag_context}}` | Fragmentos recuperados de la base de conocimiento |
| `{{rogers_segment}}` | Segmento de adopción tecnológica |
| `{{comb_of}}` `{{comb_cp}}` `{{comb_mot}}` | Barreras COM-B |
| `{{sim_tone}}` `{{sim_verbosity}}` `{{sim_confidence}}` | Parámetros de simulación |

---

## Métricas de calidad del reporte

| Dimensión | Descripción |
|-----------|-------------|
| **Navegación** | Claridad de la arquitectura de información y flujos |
| **Rendimiento** | Percepción del agente sobre velocidad de carga |
| **Accesibilidad** | Barreras detectadas según perfil demográfico |
| **Contenido** | Comprensión del copy y jerarquía visual |
| **Paridad conductual** | Estimación de alineación con usuarios reales (85–92%) |

---

## Base científica

| Modelo | Referencia | Aplicación |
|--------|-----------|------------|
| OCEAN / Big Five | Costa & McCrae (1992) | Personalidad del agente |
| ACT-R | Anderson et al. (2004) | Memoria de trabajo + decay |
| Hofstede | Hofstede, Hofstede & Minkov (2010) | Dimensiones culturales |
| Hofstede CAT | Xu et al. (2025) COLING | Validación de alineación cultural |
| PADO Framework | Wang et al. (2025) COLING | Detección OCEAN multi-agente |
| BIG5-CHAT | Bhandari et al. (2025) | Variabilidad σ por rasgo |
| Chain-of-Feeling | Synthetic Users (2025) | Progresión emocional Plutchik |
| Spreading Activation | Zhao et al. (2026) ACM CHI | Recuperación probabilística |
| Anti-sycophancy | Chen et al. (2026) ELEPHANT | Reducción sesgo de validación |
| WEIRD bias | Kumar et al. (2025) | Compensación sesgo occidental |
| Hyper-accuracy | Aher et al. (2023) ICML | Calibración de paridad |
| Rogers | Rogers (1962, 2003) | Curva de adopción tecnológica |
| COM-B | Michie et al. (2011) | Barreras conductuales |
| JTBD | Christensen et al. (2016) | Motivación funcional-emocional |
| Piaget | Piaget (1952) | Restricciones cognitivas infantiles |
| PRISMA 2020 | Page et al. (2021) BMJ | Reporte sistemático |
| GRADE | Guyatt et al. (2008) BMJ | Niveles de evidencia |
| PISA 2022 | OECD (2023) | Dominios de pensamiento creativo |

---

## Limitaciones conocidas

Los usuarios sintéticos son **hipótesis iniciales**. No detectan:

- Tiempos de carga reales (CLS, FCP, LCP)
- Bugs de rendering (z-index, hover states, overlaps)
- Comportamientos espontáneos e impredecibles
- Emociones corporales (eye-tracking, hesitación, micromovimientos)
- Contexto social real (interrupciones, presión de tiempo)

**Siempre valida los hallazgos críticos con al menos 5 participantes reales antes de decisiones de diseño definitivas.** (Hamalainen et al., CHI 2023)

---

## Uso responsable

Este sistema es una herramienta de investigación preliminar. Los resultados generados:

- **Son hipótesis** — no conclusiones
- **Requieren validación humana** para decisiones de producto
- **Pueden contener sesgos** del modelo LLM subyacente
- **No reemplazan** la observación de usuarios reales

---

## Licencia

MIT License — libre para uso académico y comercial con atribución.

---

## Créditos

**Diseño y desarrollo:** Mg. Mario Quiroz Martinez · [d3magindesign](https://d3magindesign.com) · 2026

Basado en investigación de: Costa & McCrae · Anderson & Lebiere · Geert Hofstede · Everett Rogers · Susan Michie · Clayton Christensen · Jean Piaget · Lev Vygotsky · Jakob Nielsen · Alan Cooper · Lewis et al. (RAG) · Park et al. (PADO) · Xu et al. (CAT) · Zhao et al. (Spreading Activation) · Chen et al. (ELEPHANT)
