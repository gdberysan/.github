# .github

Workflows reutilizables compartidos por los repos de gdberysan.

## security.yml

Capa de seguridad estándar: gitleaks (secretos, historial completo);
si el repo tiene `go.mod`, govulncheck (dependencias vulnerables
alcanzables) y golangci-lint con gosec (SAST); si tiene `package.json`
(raíz o un nivel abajo), Semgrep CE con rulesets JS/TS. Uso desde
cualquier repo:

    jobs:
      seguridad:
        name: Seguridad
        uses: gdberysan/.github/.github/workflows/security.yml@main
        permissions:
          contents: read

El repo consumidor con Go debe tener su propio `.golangci.yml`
(formato v2) en la raíz.
