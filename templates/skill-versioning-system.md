# Sistema de mejora continua del ecosistema de skills
> Especificación v1.0.0 · Sibbo Consulting · YYYY-MM-DD
> Aplicable a todos los SKILL.md del ecosistema

---

## §1. Por qué versionar los skills

Un skill sin versión es un artefacto sin historia. Cuando un anti-pattern nuevo entra, cuando una frontera se redefine, cuando un modo se absorbe o se elimina, el ecosistema cambia — y sin registro, cada sesión larga paga el coste de redescubrir qué cambió y por qué.

El sistema tiene tres objetivos:
1. Hacer rastreable cualquier cambio en un skill
2. Distinguir cambios de distinto impacto (corrección vs. nueva función vs. ruptura)
3. Conectar el ciclo knowledge-capture → skill update de forma explícita

---

## §2. Semver aplicado a skills

Se usa versionado semántico `MAJOR.MINOR.PATCH` con significado específico para skills:

| Nivel | Cuándo sube | Ejemplos |
|-------|------------|---------|
| **PATCH** (X.X.**1**) | Corrección sin cambio de comportamiento observable | Anti-pattern añadido · wording corregido · ejemplo actualizado · enlace roto reparado |
| **MINOR** (X.**1**.0) | Funcionalidad nueva sin romper lo existente | Nuevo modo `/comando` añadido · nueva sección · ampliación de triggers · integración con nuevo hook |
| **MAJOR** (**2**.0.0) | Cambio de frontera, trigger, función principal o absorción/división | Redefinición de qué cubre el skill · trigger cambia significativamente · skill absorbe función de otro · skill se divide |

**Regla de oro:** si alguien que usaba el skill en la versión anterior encontraría un comportamiento distinto al esperar el mismo resultado, es MAJOR. Si encuentra algo nuevo que antes no existía, es MINOR. Si no nota nada distinto en el uso normal, es PATCH.

---

## §3. Estructura de header en SKILL.md

Todo SKILL.md tiene un bloque de header estandarizado al inicio, **antes** de cualquier contenido funcional:

```markdown
# [nombre-del-skill]
**Versión:** X.Y.Z  
**Estado:** Estable / Beta / Deprecado  
**Última actualización:** YYYY-MM-DD  
**Skill owner:** [nombre o rol]  
**Depende de:** [lista de skills o herramientas que usa, o "ninguno"]  
**Usado por:** [agentes o skills que lo invocan, o "standalone"]

> [Una frase que describe la función única del skill]
```

**Ejemplo real:**

```markdown
# discrepancy-forensics
**Versión:** 1.2.0  
**Estado:** Estable  
**Última actualización:** 2026-05-14  
**Skill owner:** Sibbo Consulting  
**Depende de:** gam-adops, programmatic-deals (contexto de diagnóstico)  
**Usado por:** Agente α (Discrepancy Hunter)

> Analiza discrepancias entre datasets de publisher, SSP y buyer,
> aísla la causa raíz y produce hipótesis ordenadas por evidencia.
```

---

## §4. Bloque Changelog en SKILL.md

Todo SKILL.md tiene un bloque `## Changelog` **al final del documento**, después de todo el contenido funcional. Es la última sección.

### Formato de entrada

```markdown
## Changelog

### vX.Y.Z — YYYY-MM-DD
**Tipo:** MAJOR / MINOR / PATCH  
**Origen:** knowledge-capture [KC-ID] / sesión [fecha] / auditoría ecosistema / feedback cliente  
- [Verbo en infinitivo]: [descripción concisa del cambio]
- [Añadir / Cambiar / Eliminar / Corregir]: [descripción]

### vX.Y.Z-anterior — YYYY-MM-DD
...
```

### Ejemplo de bloque Changelog real

```markdown
## Changelog

### v1.2.0 — 2026-05-14
**Tipo:** MINOR  
**Origen:** knowledge-capture KC-2026-05-12 (caso TTD avails)  
- Añadir: Modo B — cuestionario estructurado para SSPs sin acceso a logs raw
- Añadir: sección de normalización de zona horaria en flujo de cruce de datos
- Corregir: ejemplo de cálculo de win rate (fórmula era incorrecta)

### v1.1.0 — 2026-04-28
**Tipo:** MINOR  
**Origen:** sesión 2026-04-28 — integración con Postman MCP  
- Añadir: integración con colecciones Postman para llamadas a SSP APIs
- Añadir: trigger "no entiendo por qué no puja [seat]"

### v1.0.0 — 2026-04-26
**Tipo:** MAJOR (creación)  
**Origen:** auditoría ecosistema abril 2026  
- Crear: skill inicial — flujo de aislamiento de causa raíz en discrepancias
```

---

## §5. Ciclo de actualización — write path

El mecanismo que conecta un hallazgo con una actualización de skill:

```
Hallazgo técnico verificado
        ↓
knowledge-capture (/aprender o autónomo)
        ↓
    ¿Afecta a un skill existente?
    ├── Sí → propone entrada Changelog (PATCH/MINOR/MAJOR)
    │         + diff de sección afectada en SKILL.md
    │         + espera aprobación del skill owner
    │         ↓
    │         Actualización de SKILL.md
    │         + bump de versión en header
    │         + entrada en Changelog
    └── No → se registra solo en knowledge-capture
              (candidato a nuevo skill si el patrón se repite ≥3 veces)
```

**Regla de umbral para nuevo skill:** un hallazgo que no encaja en ningún skill existente se acumula en knowledge-capture. Cuando hay 3 o más hallazgos del mismo dominio sin skill que los cubra, se abre propuesta formal de nuevo skill con la plantilla de la §6.

---

## §6. Plantilla de propuesta de nuevo skill

Cuando el umbral de la §5 se activa, se crea un documento de propuesta antes de construir el skill:

```markdown
# Propuesta de skill: [nombre-propuesto]

**Fecha de propuesta:** YYYY-MM-DD  
**Hallazgos que la motivan:** [KC-ID-1, KC-ID-2, KC-ID-3]  
**Pain points atacados:** [P1 / P2 / etc.]

## Función única
[Una frase sin "y también"]

## Árbol de decisión aplicado
- ¿Info push? No → no es hook
- ¿Orquesta ≥2 skills? [Sí/No] → [skill orquestador / skill nuevo]
- ¿Variante de existente? [Sí/No] → [si sí: modo en X / si no: skill nuevo]

## Frontera (qué cubre / qué NO cubre)
**Cubre:** ...  
**NO cubre:** ...

## Triggers de activación
- "[frase de usuario que dispara el skill]"
- "[otra frase]"

## Esfuerzo estimado de construcción
[N sesiones]

## Dependencias
[Skills o herramientas que necesita]

## Aprobación
- [ ] Skill owner revisado
- [ ] Árbol de decisión aplicado
- [ ] Frontera sin solapamiento con skills existentes
- [ ] knowledge-capture configurado para capturar en este dominio
```

---

## §7. Tabla de estado del ecosistema

Este documento se mantiene actualizado con el estado de todos los skills. Se usa al inicio de cualquier sesión de arquitectura del ecosistema.

| Skill | Versión | Estado | Última actualización | Notas |
|-------|---------|--------|---------------------|-------|
| `prebid-adtech` | X.Y.Z | Estable | YYYY-MM-DD | |
| `video-adtech` | X.Y.Z | Estable | YYYY-MM-DD | |
| `gam-adops` | X.Y.Z | Estable | YYYY-MM-DD | |
| `programmatic-deals` | X.Y.Z | Estable | YYYY-MM-DD | |
| `cmp-consent-flows` | X.Y.Z | Estable | YYYY-MM-DD | |
| `knowledge-capture` | X.Y.Z | Estable | YYYY-MM-DD | |
| `context-guardian` | X.Y.Z | Estable | YYYY-MM-DD | |
| `prompt-master` | X.Y.Z | Estable | YYYY-MM-DD | |
| `fact-checker` | X.Y.Z | Estable | YYYY-MM-DD | Incluye /verify-deliverable desde vX.Y |
| `skill-creator` | X.Y.Z | Estable | YYYY-MM-DD | |
| `discrepancy-forensics` | —  | Propuesto | — | En construcción Semana 1 |
| `client-deliverable` | — | Propuesto | — | En construcción Semana 2 |
| `broadcaster-onboarding` | — | Propuesto | — | En construcción Semana 3 |
| `incident-response` | — | Propuesto | — | En construcción Semana 4 |
| `client-context-loader` | — | Propuesto | — | En construcción Semana 1 |

---

## §8. Revisiones del ecosistema

### Revisión trimestral (macro)
Disparada por: agente ε (`/auditar-conocimiento`) + calendario

Qué revisa:
- Skills con más de 90 días sin actualización y actividad alta → ¿necesitan PATCH?
- Skills con estado "Beta" desde hace >60 días → ¿promover a Estable o deprecar?
- Hallazgos en knowledge-capture sin skill asignado → ¿umbral de propuesta activado?
- Versiones de IMA SDK, Prebid.js, GAM API → ¿algún skill necesita actualización?

### Revisión por evento (micro)
Disparada por: hook `prebid-releases-watcher` / `iab-gvl-feed` / resolución de incidencia

Qué revisa:
- ¿El evento afecta a algún skill existente?
- Si sí → propone PATCH o MINOR según impacto
- Si implica comportamiento nuevo no documentado → entrada en knowledge-capture

### Gate de sesión (nano)
Al cerrar cualquier sesión técnica:
- ¿Hubo un hallazgo capturable? → `/aprender` antes de cerrar
- ¿El hallazgo afecta a un skill? → proponer entrada Changelog
- ¿Hay una incidencia resuelta de un cliente? → actualizar §6 de su `client-context.md`

---

## §9. Integración con client-context.md

El §7 de `client-context.md` (Changelog del contexto) sigue el mismo patrón semver:
- PATCH: actualización de datos (versión de SDK, estado de RDID, nueva incidencia)
- MINOR: nuevo dispositivo o entorno añadido
- MAJOR: cambio de stack de entrega (CSAI→SSAI, cambio de player principal)

Los IDs de knowledge-capture se referencian cruzadamente entre `client-context.md` y los SKILL.md afectados para trazabilidad completa.
