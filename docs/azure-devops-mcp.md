# Conexión a Azure DevOps mediante MCP

Este repositorio incluye la configuración para conectar un cliente de IA
(Claude Code, VS Code + Copilot, Cursor, etc.) con Azure DevOps a través del
servidor MCP oficial de Microsoft: <https://github.com/microsoft/azure-devops-mcp>.

El objetivo es poder **leer** el backlog (épicas, features, work items) y
**crear** Historias de Usuario como work items en el proyecto directamente desde
el asistente, sin cargarlas a mano.

- Organización: `FabricaEscuela20262`
- Proyecto: `EAV03`
- URL: <https://dev.azure.com/FabricaEscuela20262/EAV03>

## Requisitos (una sola vez por máquina)

```powershell
# Node.js 20 o superior
node --version

# Azure CLI
winget install --id Microsoft.AzureCLI -e

# Extensión de Azure DevOps para la CLI
az extension add --name azure-devops

# Iniciar sesión con la cuenta que tiene acceso a la organización
az login
```

Verifica el acceso al proyecto:

```powershell
az devops project list --organization https://dev.azure.com/FabricaEscuela20262
```

## Configuración del MCP

### Claude Code

La configuración ya está versionada en [`.mcp.json`](../.mcp.json) en la raíz del
repositorio. Usa `--authentication azcli`, es decir reutiliza el token de
`az login` (no se necesita un PAT ni un login extra en el navegador). Al abrir
Claude Code dentro de esta carpeta, aceptar el servidor de proyecto cuando lo
solicite y luego:

```
/mcp
```

Debe mostrar `azure-devops` como `connected`.

Comprobado el 2026-09-02: con `az login --allow-no-subscriptions` hecho, el
servidor arranca (`@azure-devops/mcp` v2.9.0), responde el handshake y la
herramienta `core_list_projects` devuelve el proyecto `EAV03` de la organización.

### VS Code + Copilot / Cursor

Usar el mismo comando que declara `.mcp.json`:

```
npx -y @azure-devops/mcp FabricaEscuela20262
```

Seguir la guía del cliente correspondiente y la del repositorio del servidor
(`docs/GETTINGSTARTED.md` dentro de <https://github.com/microsoft/azure-devops-mcp>).

## Notas

- `.mcp.json` **no contiene secretos**: solo el nombre de la organización. Las
  credenciales viven en la sesión local de `az login` de cada persona.
- Si `/mcp` reporta error de autenticación, repetir `az login` y reintentar.
- Las Historias de Usuario se redactan siguiendo el skill
  [`.claude/skills/historias-usuario-invest`](../.claude/skills/historias-usuario-invest/SKILL.md)
  (modelo INVEST, criterios en Gherkin, sin detalle técnico).
