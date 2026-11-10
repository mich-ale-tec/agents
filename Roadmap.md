# Roadmap: mi sistema de agentes

Documento de referencia para construir un sistema propio de agentes de IA, organizado, portable y fácil de mantener.

> Nota: este terreno cambia rápido. Las rutas y formatos concretos de cada herramienta (Claude Code, Cursor, Copilot, Pi, etc.) deben verificarse en su documentación antes de implementarlos.

---

## 1. Conceptos clave

### ¿Los agentes son agnósticos al lenguaje?

- **El modelo sí**: el mismo agente puede programar en JS, Go, Lua o Perl.
- **Mis reglas no siempre**:
  - Universales: "máximo 3 parámetros primitivos" aplica a casi cualquier lenguaje.
  - Específicas: "replica esta plantilla Svelte" o "usa `max-params` de ESLint" solo aplican a JS.

**Principio:** separar siempre las reglas universales de las específicas.

### `AGENTS.md` no es "el agente"

`AGENTS.md` es el archivo de contexto general de un proyecto. Para tener varios agentes existen otros mecanismos:

| Mecanismo | Qué es | Ejemplo (Claude Code) |
|---|---|---|
| Subagentes | Un archivo `.md` por agente, con sus instrucciones y herramientas | `.claude/agents/` |
| Skills | Carpetas con un `SKILL.md` que se cargan solo cuando hacen falta | `.claude/skills/` |
| `AGENTS.md` anidados | Un `AGENTS.md` por carpeta del proyecto | `src/api/AGENTS.md` |

**Principio:** muchos archivos pequeños, cada uno con una responsabilidad. Nunca un archivo gigante.

### No tienen que vivir dentro del proyecto

Las herramientas suelen tener una ubicación **global de usuario** que aplica a todos los proyectos (en Claude Code: `~/.claude/agents/` y `~/.claude/skills/`).

Por eso se puede mantener **un repo git central** con todos los agentes.

---

## 2. Agentes vs. skills (la distinción más importante)

Evitar la explosión combinatoria (`programador-js`, `programador-go`, `tester-js`, `tester-go`…).

- **Agentes = roles.** Pocos.
  - programador, tester, devops, sistemas (configuración de SO).
- **Skills = conocimiento.** Muchos.
  - lenguajes (JS, Go, Lua, Perl), frameworks (Svelte), herramientas (Docker), reglas propias.

El agente *programador* trabajando en Go carga la skill de Go. El agente *tester* en Go, también. Cada pieza se escribe una sola vez.

---

## 3. Estructura del repo

```
mis-agentes/
├─ core/
│  └─ principios.md        # criterios universales (el "ADN")
├─ agents/                 # roles
│  ├─ programador.md
│  ├─ tester.md
│  ├─ devops.md
│  └─ sistemas.md
├─ skills/                 # conocimiento por tema
│  ├─ lenguajes/
│  │  ├─ js/SKILL.md
│  │  ├─ go/SKILL.md
│  │  ├─ lua/SKILL.md
│  │  └─ perl/SKILL.md
│  ├─ frameworks/
│  │  └─ svelte/SKILL.md
│  └─ reglas/
│     └─ funciones-3-params/SKILL.md
├─ checks/                 # scripts que validan reglas (linters, etc.)
└─ install.sh              # enlaza o copia todo a donde cada herramienta lo espera
```

### Qué queda en cada proyecto

Solo lo propio de ese proyecto, en su `AGENTS.md`. Por ejemplo:

- "Este proyecto usa Svelte."
- "La plantilla base está en `templates/Base.svelte`."

---

## 4. Reglas para no quedar atado a una herramienta

1. **Todo en Markdown plano.** Lo entienden todas las herramientas y sobrevive a cualquier cambio.
2. **El repo es la fuente de verdad.** Las carpetas de cada herramienta son solo destinos; `install.sh` (por ejemplo con enlaces simbólicos) los conecta. Si cambio de herramienta, cambio el script, no mis agentes.
3. **Conocimiento en formato skill**, que es el más portable hoy.
4. **Reglas verificables como scripts** en `checks/`. No dependen de ninguna IA.
   - Lo que se puede comprobar con código, se comprueba con código.
   - La IA aporta el criterio; el linter aporta la garantía.

---

## 5. Fases

### Fase 1 — Arranque mínimo
- [ X ] Crear el repo `mis-agentes` con la estructura completa (carpetas vacías).
- [ ] Escribir `core/principios.md`.
- [ ] Crear un agente: `agents/programador.md`.
- [ ] Crear dos skills: `lenguajes/js` y `reglas/funciones-3-params`.
- [ ] Escribir `install.sh` para la herramienta que use.

### Fase 2 — Uso real
- [ ] Usarlo en proyectos reales durante unas semanas.
- [ ] Cada vez que el agente haga algo que no me gusta, corregir o añadir la regla correspondiente.
- [ ] Añadir los primeros scripts en `checks/`.

### Fase 3 — Crecimiento
- [ ] Añadir agentes solo cuando uno no dé abasto: tester, devops, sistemas.
- [ ] Añadir skills de lenguajes según los necesite: Go, Lua, Perl.
- [ ] Revisar si `principios.md` sigue siendo común a todos o hay que dividirlo.

---

## Anexo: ejemplo de skill de reglas (Svelte + funciones)

```markdown
# Replicación de componentes Svelte

## Tarea
Generar archivos .svelte replicando EXACTAMENTE la plantilla de
`templates/Base.svelte`: misma estructura, orden de bloques
(<script>, markup, <style>), convenciones de nombres y estilo.
No añadas nada que la plantilla no tenga salvo lo que se pida.

## Reglas para funciones
1. Máximo 3 parámetros.
2. Cada parámetro debe ser:
   - un tipo primitivo (string, number, boolean, bigint, etc.), o
   - una clase/tipo que NO pertenezca a este proyecto
     (librería estándar o node_modules, ej. Date, Map, URL).
3. "Clase propia" = cualquier tipo importado desde `$lib/`, `./`,
   `../` o declarado en este repositorio.

## Si una función no puede cumplir las reglas
- NO la implementes. NO busques atajos (objetos agrupadores,
  `any`, desestructuración para esquivar el límite).
- Deja `// TODO(agent): ver TODO.md`.
- Regístrala en `TODO.md`.

## Formato de TODO.md
Árbol jerárquico por ruta, estilo explorador
(inspirado en Snacks.explorer de folke):

src/
└─ lib/
   └─ components/
      └─ UserCard.svelte
         └─ ✗ formatUser(user, opts, locale, tz)
            ├─ excede 3 parámetros (tiene 4)
            └─ `User` es clase propia (src/lib/models/User.ts)

- Agrupa por carpeta → archivo → función → motivos.
- Si la entrada ya existe, actualízala; no dupliques.

## Al terminar
Resume: archivos creados, funciones implementadas, entradas
añadidas a TODO.md.
```

### Dudas pendientes de decidir

- [ ] ¿`string[]` cuenta como primitivo?
- [ ] ¿Los tipos de Svelte (eventos, props de `$props()`) cuentan como clases propias?
- [ ] ¿Un parámetro opcional cuenta para el límite de 3?
- [ ] ¿Qué herramienta(s) de agente voy a usar? (define cómo escribir `install.sh`)
