# blu

Command-line interface for the *API do blü* API.

[![Built by Speakeasy](https://img.shields.io/badge/Built_by-SPEAKEASY-374151?style=for-the-badge&labelColor=f3f4f6)](https://www.speakeasy.com/?utm_source=github-com/blu-direct/blu-cli&utm_campaign=cli)
[![License: AGPL-3.0-only](https://img.shields.io/badge/LICENSE_//_AGPL--3.0--only-3b5bdb?style=for-the-badge&labelColor=eff6ff)](https://www.gnu.org/licenses/agpl-3.0.html)


<!-- Start Summary [summary] -->
## Summary

API do blü: Envie e receba mensagens no WhatsApp, gerencie agentes de IA, campanhas e webhooks.

A API do blü segue o modelo REST. Ela usa URLs previsíveis e orientadas a
recursos, recebe corpos de requisição em JSON, devolve respostas em JSON e
usa códigos de status, autenticação e métodos HTTP padrão.

## Vocabulário

A API usa os mesmos termos do WhatsApp. Um **número** é um número de
WhatsApp conectado à sua conta. Uma **conversa** liga um dos seus números a
um `wa_id` e reúne as **mensagens** trocadas entre eles. Uma **campanha**
envia um modelo de mensagem aprovado para vários destinatários.

## Tipos de plataforma

Um número usa a API do WhatsApp Cloud, da Meta (`cloud_api`), o WhatsApp
conectado por QR code (`qr`) ou os dois ao mesmo tempo (`coexistence`). O
campo `capabilities` do número informa o que ele permite, e a API devolve
um `whatsapp_error` com um `code` específico quando a requisição não segue
uma regra.

## Permissões

Cada chave de API tem permissões, e cada endpoint informa a que exige:

| Permissão | Permite |
| --- | --- |
| `phone_numbers:read` | Consultar os números de WhatsApp conectados. |
| `inbox:read` | Consultar conversas, mensagens e mídias. |
| `inbox:write` | Abrir e atualizar conversas, enviar mensagens e enviar mídias. |
| `agents:read` | Consultar agentes de IA. |
| `agents:write` | Alterar agentes de IA. |
| `campaigns:read` | Consultar modelos de mensagem, campanhas e custos. |
| `campaigns:write` | Criar e excluir modelos e preparar campanhas. |
| `campaigns:send` | Iniciar campanhas, junto com `campaigns:write`. |
| `webhooks:manage` | Gerenciar os endpoints de webhook. |

Uma chave também pode ser restrita a alguns números. Os outros números, e o
que pertence a eles, não são encontrados (`404`).

## Convenções

- **Ids** são opacos e começam com um prefixo do tipo de objeto, como `pn_`, `chat_` ou `msg_`.
- Todo objeto tem `object`, com o tipo dele. Os objetos guardados também têm
  `id`, e as datas são em segundos Unix.
- **Números**: o que pertence a um número fica sob ele, como
  `POST /v1/phone_numbers/{phone_number}/chats`. As listas gerais, como
  `GET /v1/chats`, aceitam `phone_number` para filtrar por número.
- `POST` numa coleção cria e devolve `201`; `PATCH` num objeto atualiza os
  campos enviados; `DELETE` exclui e devolve `204`, sem corpo.
- **Listas** devolvem um objeto de lista com `has_more`. A maioria é
  paginada por cursor, com `limit`, `starting_after` e `ending_before`. A
  de destinatários de campanha é paginada com `limit` e `offset` e também
  devolve `total`.
- **Limite de requisições**: as respostas trazem `X-RateLimit-Limit`,
  `X-RateLimit-Remaining` e `X-RateLimit-Reset`. Acima do limite, a resposta
  é `429 rate_limit_exceeded`, com `Retry-After` em segundos.
<!-- End Summary [summary] -->

<!-- Start Table of Contents [toc] -->
## Table of Contents
<!-- $toc-max-depth=2 -->
* [blu](#blu)
  * [Vocabulário](#vocabulário)
  * [Tipos de plataforma](#tipos-de-plataforma)
  * [Permissões](#permissões)
  * [Convenções](#convenções)
  * [CLI Installation](#cli-installation)
  * [Shell Completion](#shell-completion)
  * [CLI Example Usage](#cli-example-usage)
  * [For AI agents](#for-ai-agents)
  * [Authentication](#authentication)
  * [Commands](#commands)
  * [Request Body Input](#request-body-input)
  * [Server Selection](#server-selection)
  * [Output Formats](#output-formats)
  * [Error Handling](#error-handling)
  * [Diagnostics](#diagnostics)
* [Development](#development)
  * [Maturity](#maturity)
  * [Contributions](#contributions)

<!-- End Table of Contents [toc] -->

<!-- Start CLI Installation [installation] -->
## CLI Installation

### Quick Install (Linux/macOS)

```bash
curl -fsSL https://raw.githubusercontent.com/blu-direct/blu-cli/main/scripts/install.sh | bash
```

### Quick Install (Windows PowerShell)

```powershell
iwr -useb https://raw.githubusercontent.com/blu-direct/blu-cli/main/scripts/install.ps1 | iex
```

### Go Install

Alternatively, install directly via Go:

```bash
go install github.com/blu-direct/blu-cli/cmd/blu@latest
```

### Manual Download

Download pre-built binaries for your platform from the [releases page](https://github.com/blu-direct/blu-cli/releases).
<!-- End CLI Installation [installation] -->

<!-- Start Shell Completion [completion] -->
## Shell Completion

Shell completions are available for Bash, Zsh, Fish, and PowerShell.

### Bash

```bash
# Add to ~/.bashrc:
source <(blu completion bash)

# Or install permanently:
blu completion bash > /etc/bash_completion.d/blu
```

### Zsh

```zsh
# Add to ~/.zshrc:
source <(blu completion zsh)

# Or install permanently:
blu completion zsh > "${fpath[1]}/_blu"
```

### Fish

```fish
blu completion fish | source

# Or install permanently:
blu completion fish > ~/.config/fish/completions/blu.fish
```

### PowerShell

```powershell
blu completion powershell | Out-String | Invoke-Expression
```
<!-- End Shell Completion [completion] -->

<!-- Start CLI Example Usage [usage] -->
## CLI Example Usage

### Example

```bash
blu conta get-account --secret-key 'Bearer test_token'

```
<!-- End CLI Example Usage [usage] -->

<!-- Start For AI agents [agents] -->
## For AI agents

This CLI is built to be driven by AI coding agents as well as people: everything an agent needs is discoverable from the binary itself, and every command can be validated without credentials. Work down this ladder:

| Run | You get |
|-----|---------|
| `blu --help`, `blu agentes list-agents --help` | Commands by category, runnable examples, flags |
| `blu --usage`, `blu agentes list-agents --usage` | The command surface as machine-readable [KDL](https://kdl.dev): commands, aliases, flags, defaults, env vars, config keys |
| `blu agentes update-agent --schema` | The exact JSON Schema of the command's request body (all `$ref`s bundled) — build a valid `--body` from it |
| `blu agentes list-agents --dry-run` | The exact HTTP request (method, URL, headers, body), with no credentials or network call |
| `blu agentes list-agents --output-format json` (or `--jq`) | Machine-readable output |

### Discover the command surface

```bash
# Every command, flag, default, env var and config key, as KDL
blu --usage

# One command's subtree only
blu agentes list-agents --usage
```

### Read the exact request schema

`--schema` is available on every command that accepts a request body (`--body`, stdin, or a whole-body flag where the command has one), including intent commands. It prints the JSON Schema the request is validated against and exits without calling the API.

```bash
# JSON Schema (draft 2020-12) of the request body, with every $ref bundled under $defs
blu agentes update-agent --schema
```

### Probe before you spend

Start quota-spending commands with `--dry-run`. It validates inputs, resolves the request, redacts secrets and binary payloads, makes no network call, and exits 0. It never reads the OS keychain; credentials supplied by flag, environment, or config file are included only as `[REDACTED]`.

```bash
# Human preview: the [DRY-RUN] block is on stderr and stdout is empty
blu agentes list-agents --dry-run

# Machine preview: compact JSON on stdout and silent stderr
blu agentes list-agents --dry-run --output-format json
```

The machine form writes one object per would-be request, one per line (NDJSON for multi-request commands), with exactly this shape:

```json
{"dry_run":true,"request":{"method":"POST","url":"https://…","headers":{"Accept":["application/json"],…},"body":<JSON value | string | null>}}
```

`body` is a parsed JSON value when the body is JSON, a string for text, `"<bytes:N>"` for binary data, and `null` when absent. An explicit caller `--jq` also selects this JSON preview protocol, but the filter is not applied to preview objects. Command-declared jq presets do not select or filter the preview.

Local mutation commands make no request under `--dry-run`: instead of a preview they emit one `{"dry_run":true,"local":true,"command":"…","message":"…"}` object. `select(.request)` keeps only would-be requests; `select(.local)` keeps the local no-ops.

### Machine-readable output

```bash
# JSON on stdout
blu agentes list-agents --output-format json

# Filter or reshape with a jq expression (always emits JSON, overrides --output-format)
blu agentes list-agents --jq '.'

# Print jq string results as plain text instead of JSON strings (like jq -r)
blu agentes list-agents --jq '.' --raw-output
```

`--output-format toon` emits [TOON](https://github.com/toon-format/spec), a compact line-oriented format that uses fewer tokens than JSON; it is the default in agent mode.

### Interactive mode
Required-input prompts and guided `configure` / `auth login` forms are enabled by default. Required-input prompts require an interactive terminal; off-TTY forms read line input from stdin. Use `--no-interactive` to force flag-only execution.

```bash
# Prompt for missing command inputs
blu agentes list-agents --interactive

# Open the guided configuration form
blu configure --interactive

# Explicitly launch the terminal command explorer
blu explore
```

### Agent mode and structured errors
Agent mode turns on automatically when a known agent environment is detected (`CLAUDECODE`, `CURSOR_AGENT`, `CODEX`, `AIDER`, `CLINE`, `WINDSURF_AGENT`, `GITHUB_COPILOT`, `AMAZON_Q`, `GEMINI_CODE_ASSIST`, `SRC_CODY`) or with `--agent-mode` (`--agent-mode=false` disables detection).
In agent mode interactive prompts never launch, output defaults to TOON, and every failure — API errors and CLI usage errors alike — is one JSON envelope on stderr:
Outside agent mode, explicit JSON and `--jq` preserve the compatibility envelope without classification; enable agent mode to request the classified contract.

```json
{
  "error": "...",
  "error_type": "validation_error",
  "error_reason": "CLI_VALIDATION",
  "exit_code": 2,
  "message": "human-readable message",
  "hints": ["what to try next"]
}
```

`error_type` is one of `authentication_error`, `authorization_error`, `not_found`, `validation_error`, `rate_limit_error`, `server_error`, `api_error`, `connection_error`, `protocol_error`, `runtime_error`, `unsupported_error`, `async_failed`, `async_timeout`, `async_unknown_state`. Classification derives from the HTTP status and transport evidence; `error_reason` is absent for API errors. Status-less local failures may use `CLI_VALIDATION`, `CLI_CONNECTION`, `CLI_PROTOCOL`, `CLI_RUNTIME`, `CLI_UNAVAILABLE`, `CLI_AUTHENTICATION`, the async polling reasons `CLI_ASYNC_FAILED`, `CLI_ASYNC_TIMEOUT`, and `CLI_ASYNC_UNKNOWN_STATE`, or `CLI_OUTCOME_FAILED` when a successful response reports a declared failure state; that envelope also carries `state` (the reported state) and, when the command declares an error pointer, `error` (the failure detail taken from the response). `hints` preserves server guidance first, adds the most specific local taxonomy guidance, then typed CLI and command-specific guidance, removing exact duplicates. `exit_code` is always the code for the final `error_type` shown in the envelope: 1 runtime, 2 usage, or 3 authentication/authorization.
<!-- End For AI agents [agents] -->

<!-- Start Authentication [security] -->
## Authentication

Authentication credentials can be configured in four ways (in order of priority):

### 1. Command-line flags

Pass credentials directly as flags to any command:

```bash
blu --secret-key "$BLU_SECRET_KEY" agentes list-agents
```

### 2. Environment variables

Set credentials via environment variables:

| Variable | Description |
|----------|-------------|
| `BLU_SECRET_KEY` | Autentique com uma chave secreta (`sk_live_…`) criada no painel. Cada chave tem permissões, como `inbox:read` e `inbox:write`, e pode ser restrita a alguns números. Sem a permissão do endpoint, a resposta é `403 insufficient_scope`; um número fora da chave não é encontrado (`404`). Com a assinatura da conta inativa, a resposta é `403 subscription_inactive`. |

### 3. OS Keychain (recommended for workstations)

Credentials are stored securely in your operating system's keychain when you run:

```bash
blu configure
```

Secret credentials (tokens, API keys, passwords) are automatically stored in:
- **macOS**: Keychain
- **Linux**: GNOME Keyring / KWallet (via D-Bus Secret Service)
- **Windows**: Windows Credential Locker

If no keychain is available (e.g., in CI environments), credentials fall back to the config file.

Where the keychain cannot be unlocked (e.g., headless Linux or SSH sessions with a locked GNOME Keyring), skip it entirely with `--no-keyring`, `BLU_NO_KEYRING=true`, or `no_keyring: true` in the config file. The keychain is then never read or written, and secrets are stored in the config file instead; flags and environment variables still take precedence.

### 4. Configuration file

Run the interactive `configure` command to store non-secret settings:

```bash
blu configure
```

Configuration is stored in `~/.config/blu/config.yaml`.
<!-- End Authentication [security] -->

<!-- Start Commands [operations] -->
## Commands

<details open>
<summary>Available commands</summary>

* [`conta`](docs/blu_conta.md) - A conta à qual sua chave de API pertence
  * [`get-account`](docs/blu_conta_get-account.md) - Consultar a conta
* [`numeros`](docs/blu_numeros.md) - Números de WhatsApp conectados à sua conta
  * [`list-phone-numbers`](docs/blu_numeros_list-phone-numbers.md) - Listar números
  * [`get-phone-number`](docs/blu_numeros_get-phone-number.md) - Consultar um número
* [`conversas`](docs/blu_conversas.md) - Conversas entre um dos seus números e um `wa_id`
  * [`create-chat`](docs/blu_conversas_create-chat.md) - Abrir uma conversa
  * [`list-chats`](docs/blu_conversas_list-chats.md) - Listar conversas
  * [`get-chat`](docs/blu_conversas_get-chat.md) - Consultar uma conversa
  * [`update-chat`](docs/blu_conversas_update-chat.md) - Atualizar uma conversa
* [`mensagens`](docs/blu_mensagens.md) - Mensagens enviadas e recebidas em uma conversa
  * [`list-messages`](docs/blu_mensagens_list-messages.md) - Listar as mensagens de uma conversa
  * [`send-message`](docs/blu_mensagens_send-message.md) - Enviar uma mensagem
  * [`get-message`](docs/blu_mensagens_get-message.md) - Consultar uma mensagem
* [`midia`](docs/blu_midia.md) - Arquivos usados em mensagens
  * [`create-media`](docs/blu_midia_create-media.md) - Enviar mídia
  * [`get-media`](docs/blu_midia_get-media.md) - Consultar mídia
* [`agentes`](docs/blu_agentes.md) - Agentes de IA que respondem às conversas
  * [`list-agents`](docs/blu_agentes_list-agents.md) - Listar agentes
  * [`get-agent`](docs/blu_agentes_get-agent.md) - Consultar um agente
  * [`update-agent`](docs/blu_agentes_update-agent.md) - Atualizar um agente
* [`modelos-de-mensagem`](docs/blu_modelos-de-mensagem.md) - Mensagens aprovadas pela Meta, da conta do WhatsApp Business de um número `cloud_api` ou `coexistence`
  * [`list-message-templates`](docs/blu_modelos-de-mensagem_list-message-templates.md) - Listar os modelos de mensagem de um número
  * [`create-message-template`](docs/blu_modelos-de-mensagem_create-message-template.md) - Criar um modelo de mensagem
  * [`get-message-template`](docs/blu_modelos-de-mensagem_get-message-template.md) - Consultar um modelo de mensagem
  * [`delete-message-template`](docs/blu_modelos-de-mensagem_delete-message-template.md) - Excluir um modelo de mensagem
* [`campanhas`](docs/blu_campanhas.md) - Um modelo de mensagem enviado para vários destinatários
  * [`list-broadcasts`](docs/blu_campanhas_list-broadcasts.md) - Listar campanhas
  * [`create-broadcast`](docs/blu_campanhas_create-broadcast.md) - Criar uma campanha
  * [`get-broadcast`](docs/blu_campanhas_get-broadcast.md) - Consultar uma campanha
  * [`update-broadcast`](docs/blu_campanhas_update-broadcast.md) - Atualizar uma campanha
  * [`list-broadcast-recipients`](docs/blu_campanhas_list-broadcast-recipients.md) - Listar os destinatários de uma campanha
* [`custos`](docs/blu_custos.md) - O que a Meta cobra pelas mensagens dos seus números
  * [`get-costs`](docs/blu_custos_get-costs.md) - Consultar os custos
* [`endpoints-de-webhook`](docs/blu_endpoints-de-webhook.md) - URLs que recebem os eventos de um dos seus números
  * [`list-webhook-endpoints`](docs/blu_endpoints-de-webhook_list-webhook-endpoints.md) - Listar os endpoints de webhook de um número
  * [`create-webhook-endpoint`](docs/blu_endpoints-de-webhook_create-webhook-endpoint.md) - Criar um endpoint de webhook para um número
  * [`get-webhook-endpoint`](docs/blu_endpoints-de-webhook_get-webhook-endpoint.md) - Consultar um endpoint de webhook
  * [`update-webhook-endpoint`](docs/blu_endpoints-de-webhook_update-webhook-endpoint.md) - Atualizar um endpoint de webhook
  * [`delete-webhook-endpoint`](docs/blu_endpoints-de-webhook_delete-webhook-endpoint.md) - Excluir um endpoint de webhook

</details>
<!-- End Commands [operations] -->

<!-- Start Request Body Input [stdinpiping] -->
## Request Body Input

Commands that accept a request body take it three ways, with a clear priority chain: individual field flags (highest priority), the whole body as JSON via `--body`, and JSON piped on stdin (lowest priority). Later sources never override earlier ones; a body-bearing command prints its exact request schema with `--schema`.
<!-- End Request Body Input [stdinpiping] -->

<!-- Start Server Selection [server] -->
## Server Selection

### Override Server URL

Use `--server-url` to override the server URL entirely, bypassing any named or indexed server selection:

```bash
blu --server-url https://custom-api.example.com agentes list-agents
```

**Precedence**: `--server-url` > `--server` > default
<!-- End Server Selection [server] -->

<!-- Start Output Formats [output-formats] -->
## Output Formats

Every command supports a `--output-format` flag that controls how the response is rendered to stdout.

### Available formats

| Format | Flag | Description |
|--------|------|-------------|
| Pretty | `--output-format pretty` (default) | Aligned key-value pairs with color, nested indentation. Human-readable at a glance. |
| JSON | `--output-format json` | JSON output. Passthrough when the response is already JSON (preserves original field order and numeric precision). Falls back to typed marshaling otherwise. |
| YAML | `--output-format yaml` | YAML output via standard marshaling. |
| Table | `--output-format table` | Tabular output for array responses. |
| TOON | `--output-format toon` | [Token-Oriented Object Notation](https://github.com/toon-format/spec) — a compact, line-oriented format that typically uses 30–60% fewer tokens than JSON. Well-suited for piping responses into LLM prompts. |

```bash
# Default pretty output
blu agentes list-agents

# Machine-readable JSON
blu agentes list-agents --output-format json

# TOON for LLM-friendly compact output
blu agentes list-agents --output-format toon

# Pipe JSON to jq without using --output-format
blu agentes list-agents --output-format json | jq '.'
```

### jq filtering

Use `--jq` to filter or transform the response inline using a [jq](https://jqlang.org) expression. This always outputs JSON and overrides `--output-format`:

```bash
# Extract a single field
blu agentes list-agents --jq '.'

# Reshape with any jq program; --raw-output prints string results as plain text (like jq -r)
blu agentes list-agents --jq '.' --raw-output
```

### Color control

Use `--color` to control terminal colors:

| Value | Behavior |
|-------|----------|
| `auto` (default) | Color when stdout is a TTY, plain text otherwise |
| `always` | Always colorize |
| `never` | Never colorize |

The `NO_COLOR` and `FORCE_COLOR` environment variables are also respected.

### Streaming and pagination

When using `--all` (pagination) or streaming operations, output is written incrementally as items arrive:

| Format | Streaming behavior |
|--------|-------------------|
| `json` | One compact JSON object per line ([NDJSON](https://github.com/ndjson/ndjson-spec)) |
| `yaml` | YAML documents separated by `---` |
| `toon` | One TOON-encoded object per block, separated by blank lines |
| `pretty` (default) | Pretty-printed items separated by blank lines |
<!-- End Output Formats [output-formats] -->

<!-- Start Error Handling [errors] -->
## Error Handling

The CLI uses standard exit codes to indicate success or failure:

| Exit Code | Meaning |
|-----------|---------|
| `0` | Success |
| `1` | Runtime/API failure |
| `2` | Usage or input failure |
| `3` | Authentication or authorization failure |

On success, the response data is printed to **stdout** as JSON. On failure, error details are printed to **stderr**.

```bash
# Capture output and handle errors
blu agentes list-agents --output-format json > output.json 2> error.log
if [ $? -ne 0 ]; then
  echo "Error occurred, see error.log"
fi
```
This CLI uses unclassified error rendering outside agent mode: pretty and TOON print the API error text as received, while `--output-format json` and `--jq` emit the unclassified envelope (including the configure `_hint` for HTTP 401/403) plus `exit_code`. Agent mode always emits the classified JSON envelope with `exit_code`, `error_type`, optional `error_reason`, `message`, `hints`, and optional `status_code` — see [For AI agents](#for-ai-agents).
<!-- End Error Handling [errors] -->

<!-- Start Diagnostics [diagnostics] -->
## Diagnostics

The CLI includes two diagnostic flags available on all commands:

### Dry Run

Preview what would be sent without making any network calls:

```bash
blu agentes list-agents --dry-run
```

In human output modes, stdout is empty and the `[DRY-RUN]` block goes to stderr. It includes:
- HTTP method and URL
- Request headers (sensitive values redacted)
- Request body preview (sensitive fields redacted)

In machine modes (`--agent-mode`, `--output-format json`, `yaml` or `toon`, or a caller-explicit `--jq`), stderr is silent and stdout is NDJSON: one compact preview object per would-be request, whatever the output format. The jq filter is not applied, and command-declared jq presets do not select the JSON protocol.

```json
{"dry_run":true,"request":{"method":"POST","url":"https://…","headers":{"Accept":["application/json"],…},"body":<JSON value | string | null>}}
```

JSON bodies remain structured; text bodies are strings; binary bodies are `"<bytes:N>"`; absent bodies are `null`. Headers retain all values as arrays, with credentials replaced by `[REDACTED]`. Dry-run never reads the OS keychain, but credentials supplied by flag, environment, or config file still appear redacted. The command exits successfully without contacting the API.

Local mutation commands emit one `{"dry_run":true,"local":true,"command":"…","message":"…"}` object in place of a preview; filter with `select(.request)` or `select(.local)`.

### Debug

Log request and response diagnostics while running normally:

```bash
blu agentes list-agents --debug
```

Debug output goes to stderr and includes:
- Request method, URL, headers, and body preview
- Response status, headers, and body preview
- Transport errors (if any)

The command still executes normally and produces its regular output on stdout.

### Flag Precedence

If both `--dry-run` and `--debug` are set, `--dry-run` takes precedence and no network calls are made.

### Security

Sensitive information is automatically redacted in diagnostic output:
- **Headers**: `Authorization`, `Cookie`, `Set-Cookie`, `X-API-Key`, and other security headers show `[REDACTED]`
- **Body**: JSON fields named `password`, `secret`, `token`, `api_key`, `client_secret`, etc., and fields named like sensitive headers (such as `Authorization`, `Cookie` or `X-Auth-Token`), show `[REDACTED]`. A JSON Schema stored under such a name keeps its structure, with `default`, `const`, `enum`, `example` and `examples` masked at every level
- **Binary data**: binary media and canonical base64 strings are replaced with `<bytes:N>`
- **URL query**: credential-like query parameters are replaced with `[REDACTED]`

Diagnostic output should still be treated as potentially sensitive operational data.
<!-- End Diagnostics [diagnostics] -->

<!-- Placeholder for Future Speakeasy SDK Sections -->

# Development

## Maturity

This CLI is in beta, and there may be breaking changes between versions without a major version update. Therefore, we recommend pinning usage
to a specific package version. This way, you can install the same version each time without breaking changes unless you are intentionally
looking for the latest version.

## Contributions

This CLI is generated programmatically. Edits to generated files are overwritten on regeneration. To customize it:

- **Configuration and behavior:** Use [OpenAPI overlays](https://www.speakeasy.com/docs/prep-openapi/overlays/create-overlays) in the Speakeasy workflow with `x-speakeasy-*` extensions (for example, `x-speakeasy-cli-commands`) to define commands, flags, help text, examples, authentication, and grouping.
- **Persistent code changes:** Store unified diffs as [patch files](https://www.speakeasy.com/docs/sdks/customize/code/patch-files/patch-files) at `.speakeasy/patches/<path-of-generated-file>.patch`; they are re-applied on every generation.

### CLI Created by [Speakeasy](https://www.speakeasy.com/?utm_source=github-com/blu-direct/blu-cli&utm_campaign=cli)
