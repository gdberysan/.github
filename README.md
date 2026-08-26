# .github — workflows compartidos de gdberysan

Repo especial de GitHub con los workflows reutilizables que comparten todos
mis repos. Es público porque los repos privados solo pueden llamar workflows
reutilizables públicos en el plan Free; **nunca contiene secretos**.

## `security.yml` — capa de seguridad estándar

Un solo workflow que cada repo consume desde su CI. Autodetecta el stack del
repo llamante y activa solo los jobs que aplican:

| Job | Se activa cuando | Qué hace |
|---|---|---|
| `gitleaks` | siempre | Busca secretos en **todo el historial de git** (fetch-depth 0) |
| `govulncheck` | existe `go.mod` | CVEs en dependencias Go **alcanzables** por el grafo de llamadas (casi cero falsos positivos); incluye la stdlib del toolchain fijado |
| `golangci-lint` | existe `go.mod` | Lint + SAST de Go (gosec) usando el `.golangci.yml` del repo llamante |
| `semgrep` | existe `package.json` en la raíz o un nivel abajo (`web/`, `site/`…) | SAST de JS/TS con los rulesets `p/javascript` + `p/typescript` (imagen pineada; sin telemetría) |

Añadir un ecosistema nuevo (Python, Rust…) = un job más **aquí**, gateado por
su manifiesto; todos los repos lo heredan sin tocar sus `ci.yml`.

### Uso desde cualquier repo

```yaml
jobs:
  seguridad:
    name: Seguridad
    uses: gdberysan/.github/.github/workflows/security.yml@main
    permissions:
      contents: read
      # gitleaks-action lee los commits del PR en eventos pull_request
      pull-requests: read
```

Los checks resultantes se llaman `Seguridad / gitleaks`, `Seguridad /
govulncheck`, etc. — copiar los nombres exactos de una corrida real al
configurar required status checks.

### Requisitos del repo llamante

- **Go**: un `.golangci.yml` (formato v2) en la raíz, mínimo:

  ```yaml
  version: "2"
  linters:
    default: standard
    enable:
      - gosec
  ```

- **JS/TS**: nada — Semgrep no necesita config en el repo.
- Un hallazgo se corrige o se excluye **inline con justificación**
  (`#nosec G### -- motivo`, `// nosemgrep: regla -- motivo`,
  `.gitleaksignore` comentado). Nunca se desactiva una regla globalmente.

## Auto-test y versionado

- `ci.yml` de este repo llama a su propio workflow en cada push (aquí solo
  corre gitleaks: no hay manifiestos de Go/JS).
- Mientras haya pocos consumidores se consume `@main`; a partir de ~3
  consumidores se taggea `@v1` y los bumps son deliberados.
- Dependabot vigila las versiones de las actions de este repo — los bumps
  aquí propagan a la CI de todos los consumidores.

## Contexto

Este workflow es una pieza del baseline de seguridad documentado en el repo
(privado) `security`: enforcement en CI, una herramienta por capa, dashboard
= pestaña Security de GitHub, y revisión de autorización por feature branch
para la clase que ningún escáner detecta.
