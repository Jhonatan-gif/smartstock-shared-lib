# smartstock-shared-lib

Jenkins **Shared Library** del pipeline CI/CD de SmartStock Analytics (StockDinámico).
Cada archivo de `vars/` es un paso reutilizable con una sola responsabilidad, así el
`Jenkinsfile` del proyecto queda declarativo y el mismo pipeline sirve para otros
proyectos Node.js.

## Uso

En Jenkins: *Manage Jenkins → System → Global Pipeline Libraries* → nombre
`smartstock-shared-lib`, versión por defecto `main` (ya viene configurado por JCasC
en `infra/` del proyecto). En el `Jenkinsfile`:

```groovy
@Library('smartstock-shared-lib@main') _
```

## Pasos

| Paso | Responsabilidad | Optimización |
|---|---|---|
| `detectChanges(modules: [...])` | Exporta `<MODULO>_CHANGED` según el diff (PR → rama destino, feature → último build exitoso) | Ejecución selectiva |
| `isProtectedBranch()` | Regla GitFlow: `main`, `develop`, `release/*`, `hotfix/*` | Ejecución selectiva |
| `npmInstallCached('backend')` | Restaura `node_modules` por hash de `package-lock.json` + versión de Node | Caché |
| `npmScript('frontend', 'lint')` | Ejecuta cualquier script de `package.json` en un módulo | Modular |
| `securityAudit(['backend', 'frontend'])` | `npm audit` de producción → build UNSTABLE si hay vulnerabilidades altas | Calidad |
| `sonarScan()` | Análisis SonarQube con caché de plugins del scanner | Caché |
| `dockerBuild(name:, context:)` | Imagen Docker con BuildKit y `--cache-from` | Caché |
| `deploy(environment:, hookCredentialsId:)` | Deploy Hook (Render/Railway); simulado si la credencial no existe | — |
| `resolveStatusSha()` | En PRs apunta el estado al commit HEAD del PR, no al merge | Notificación |
| `githubStatus(state, description)` | Commit Status en GitHub (pending/success/failure/error) | Notificación |
| `notifyChat(result)` | Mensaje opcional a Discord (credencial `discord-webhook`) | Notificación |
| `pipelineSummary()` | Resumen final: rama, módulos ejecutados/omitidos, duración | — |

## Credenciales que usa

| ID | Tipo | Paso |
|---|---|---|
| `github-token` | Secret text (PAT con *Commit statuses: write*) | `githubStatus` |
| `render-hook-staging` / `render-hook-prod` | Secret text (opcional) | `deploy` |
| `discord-webhook` | Secret text (opcional) | `notifyChat` |
