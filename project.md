# rubber-duck 🦆

> *Don't give me the answer. Help me find it.*

Un plugin para Claude Code que invierte el rol del agente: en vez de dar soluciones, hace preguntas hasta que el usuario llega solo a la respuesta. Inspirado en la técnica clásica de rubber duck debugging.

------

## El problema

Claude Code por defecto resuelve. Siempre. Le dices "tengo un bug raro" y te devuelve la solución antes de que hayas terminado de pensar en el problema. Eso es útil cuando quieres velocidad, pero destruye el aprendizaje y a veces te hace depender del agente para entender tu propio código.

## La solución

Una skill que convierte a Claude en un interlocutor socrático. Escucha, pregunta, guía. No resuelve hasta que el usuario lo pide explícitamente. El usuario aprende, el usuario entiende, el usuario resuelve.

**Tagline:** *"Don't give me the answer. Help me find it."*

------

## Audiencia objetivo

- **Estudiantes y juniors** aprendiendo a programar con IA sin perder el razonamiento propio
- **Seniors** que quieren entender código ajeno antes de modificarlo
- **Cualquiera atascado** que prefiere llegar a la solución por sus propios medios

------

## Estructura del repositorio

```
rubber-duck/
├── .claude-plugin/
│   ├── plugin.json          # Metadata del plugin
│   └── marketplace.json     # (Iter 4) manifiesto para /plugin marketplace add
├── skills/
│   └── rubber-duck/
│       ├── SKILL.md          # Núcleo: instrucciones de comportamiento
│       ├── evals/
│       │   └── evals.json    # (Iter 2) casos de test input → output esperado
│       └── checkpoints.yaml  # (Iter 2) criterios de calidad medibles
├── commands/                 # Slash commands
│   ├── rubber-duck.toml      # /rubber-duck (activar)
│   ├── duck.toml             # /duck lite|full|ultra (intensidad)
│   └── duck-off.toml         # /duck-off (salir)
├── hooks/
│   └── activate.js           # (Iter 3) SessionStart hook (activación opcional)
├── LICENSE                   # MIT
├── README.md                 # Documentación humana + ejemplos + benchmarks
└── AGENTS.md                 # Compatibilidad con Cursor, Copilot, Windsurf
```

------

## Comportamiento esperado

### Triggers de activación

El modo se activa cuando el usuario dice cosas como:

- `"tengo un bug raro"`
- `"no entiendo por qué falla esto"`
- `"estoy atascado con..."`
- `"explícame qué hace este código"`
- `/rubber-duck`

### Protocolo de respuesta (cuando está activo)

1. **Nunca dar la solución de primeras.** Responder siempre con UNA sola pregunta.
2. **Preguntas socráticas en cadena** — cada pregunta lleva más profundo hacia la causa raíz.
3. **Pistas graduales** — si tras 3 intercambios no hay avance, dar una pista pequeña (no la solución).
4. **Confirmar cuando el usuario llega** — validar y consolidar el aprendizaje.
5. **Salida explícita** — `/duck-off` o "dime la solución" desactiva el modo.

### Lo que NUNCA hace

- Dar código directamente (a menos que se pida explícitamente)
- Hacer más de una pregunta por respuesta
- Explicar el problema si el usuario no lo ha articulado primero

------

## Iteraciones

### 🟢 Iter 1 — MVP funcional

*Objetivo: que la skill funcione y sea instalable*

- [x] `plugin.json` — metadata mínima (nombre, versión, descripción, autor). Sin `hooks` (Iter 3) ni bloque `interface` (Iter 4). Autor: Álvaro Díaz / github.com/alvaroddiaz
- [x] `SKILL.md` — instrucciones de comportamiento core (triggers, protocolo, salida, **3 niveles de intensidad lite/full/ultra**)
- [x] `README.md` — instalación en una línea + ejemplo de conversación antes/después (sin emojis)
- [x] `AGENTS.md` — reglas globales para compatibilidad con otros agentes
- [x] `commands/*.toml` — slash commands reales: `/rubber-duck` (activar), `/duck` (cambiar intensidad), `/duck-off` (salir). Sin esto el `/` literal no funciona, solo la activación por texto
- [x] `LICENSE` — MIT (coherente con plugin.json y SKILL.md)
- [x] Instalación manual verificada en Claude Code local (`npx skills add <url> --skill rubber-duck`, symlink global a `.claude/skills`)
- [x] Test manual: conversaciones de prueba con bugs reales. Core validado como plugin: 1 pregunta/turno, cava hondo, pistas graduales sin spoiler, confirma aterrizaje, sale limpio con petición explícita. **Gap detectado para Iter 2:** la skill responde en inglés aunque el usuario escriba en español — falta regla de "espejo de idioma" en SKILL.md
- [x] **Corrección de diseño:** `description` de SKILL.md auto-disparaba con "estoy atascado" (contradecía la decisión "activación manual"). Estrechado a activación SOLO explícita (`/rubber-duck`, `/duck`, o petición explícita de modo). Sin esto la skill secuestraba peticiones normales

**Entregable:** repo público en GitHub, instalable vía `npx skills add`

------

### 🟡 Iter 2 — Calidad y evals

*Objetivo: medir que la skill realmente funciona, no solo que parece que funciona*

- [x] `evals/evals.json` — 12 casos conductuales (no exact-match, porque la skill es no determinista). Cubren: activación explícita, no-solución en turno 1, 1 pregunta/turno, espejo de idioma, no leading-answer, timing de pistas (lite/full/ultra), confirmación de aterrizaje, salida `/duck-off` + frase natural, seguridad directa
- [x] `evals/grade.py` — grader stdlib, sin deps. Assertions deterministas (1 pregunta, sin código, idioma, salida) decididas por código; semánticas (no-solución, hay-pista, confirma) marcadas MANUAL para juez humano/LLM. Tiene `--self-check` que prueba la lógica
- [x] `checkpoints.yaml` — gates de calidad con targets (% por gate, 100% en core y seguridad)
- [ ] Script de benchmark: baseline (sin skill) vs rubber-duck, n=10 prompts. **Pendiente:** requiere harness que llame al modelo (API key). grade.py ya puntúa replies pegadas a mano sin API
- [x] Ajuste del `SKILL.md` basado en test manual: regla de **espejo de idioma** añadida (responde en el idioma del usuario)

**Entregable:** evals ejecutables + resultados publicados en README

------

### 🟡 Iter 3 — Distribución multi-agente

*Objetivo: que funcione en Cursor, Windsurf, Copilot y similares*

- [ ] `hooks/activate.js` — hook SessionStart para auto-activación opcional
- [ ] Reglas para Cursor (`.cursor/rules/rubber-duck.mdc`)
- [ ] Reglas para Windsurf (`.windsurf/rules/rubber-duck.md`)
- [ ] Reglas para Copilot (`.github/copilot-instructions.md`)
- [ ] Test en cada agente compatible
- [ ] Instalador unificado o documentación por agente

**Entregable:** compatibilidad verificada con 3+ agentes

------

### 🔵 Iter 4 — Pulido y marketplace

*Objetivo: publicación en marketplaces y visibilidad*

- [ ] Submisión a [claudeskills.info](https://claudeskills.info/) y [agensi.io](https://agensi.io/)
- [x] Plugin marketplace de Claude Code: `.claude-plugin/marketplace.json` creado (adelantado de Iter 4 a Iter 1 — necesario para que `/rubber-duck` funcione como slash command vía `/plugin install`). El install por `npx skills` solo trae la skill, no los commands
- [ ] README con benchmarks reales (% de sesiones donde el usuario resuelve sin solución directa)
- [ ] Ejemplos de conversaciones reales (anonimizados)
- [ ] Versión ES/EN del README
- [ ] Post de lanzamiento para @CentsAndCode / LinkedIn

**Entregable:** skill publicada y con primeras installs

------

### 🔵 Iter 5 — Extensiones (backlog)

*Ideas para versiones futuras, priorizables según feedback*

- [x] ~~**Niveles de intensidad** (`/duck lite` / `/duck full` / `/duck ultra`)~~ — **adelantado a Iter 1.** Eje = andamiaje según experiencia: lite (novato, pista tras 1 intercambio + contexto cálido), full (default, pista tras 3), ultra (avanzado, sin pistas nunca).
- [ ] **Modo profesor** — adapta el nivel de las preguntas según el nivel percibido del usuario
- [ ] **Resumen de sesión** — al salir del modo, genera un resumen de lo que el usuario descubrió
- [ ] **`/duck-stats`** — contador de "aha moments" (veces que el usuario llegó solo a la solución)
- [ ] **Subagentes especializados** — duck-debugger, duck-reviewer, duck-architect, cada uno con preguntas adaptadas al tipo de problema

------

## Decisiones técnicas

| Decisión            | Elección                            | Motivo                                 |
| ------------------- | ----------------------------------- | -------------------------------------- |
| Activación          | Manual por defecto (`/rubber-duck`) | No interrumpir flujo normal de trabajo |
| Idioma del SKILL.md | Inglés                              | Máxima compatibilidad con marketplaces |
| Hook SessionStart   | Opcional en Iter 3                  | MVP sin dependencia de Node.js         |
| Intensidad          | 3 niveles (lite/full/ultra) en Iter 1 | Eje = andamiaje por experiencia; abre la skill a novatos y avanzados desde el MVP |
| Tests               | evals.json manual en Iter 2         | Misma estrategia que caveman           |

------

## Referencias

- [ponytail](https://github.com/DietrichGebert/ponytail) — referencia de estructura y evals
- [caveman](https://github.com/JuliusBrussee/caveman) — referencia de hooks, multi-agente e instalador
- [claude-code plugins spec](https://github.com/anthropics/claude-code/blob/main/plugins/README.md) — spec oficial de Anthropic
- [claudeskills.info](https://claudeskills.info/) — marketplace principal
- [agensi.io](https://agensi.io/) — marketplace secundario con métricas de adopción