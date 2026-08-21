# Ollama

The Compose stack exposes Ollama through a Caddy reverse proxy that requires an
API key. Ollama itself does not provide authentication for its local API.

## Start

Copy `.env.example` to `.env`, generate a secure key as described below, then
start the stack:

```powershell
Copy-Item .env.example .env
docker compose up -d
```

## Generate an authentication key

Use a cryptographically secure random number generator. A 32-byte key is a
good minimum for this local proxy. In PowerShell, run the following from the
repository directory after copying `.env.example`:

```powershell
$rng = [System.Security.Cryptography.RandomNumberGenerator]::Create()
$bytes = New-Object byte[] 32
$rng.GetBytes($bytes)
$rng.Dispose()
$key = [Convert]::ToBase64String($bytes)
(Get-Content .env) -replace '^OLLAMA_API_KEY=.*$', "OLLAMA_API_KEY=$key" | Set-Content .env
```

On Linux or macOS, the equivalent is:

```bash
printf 'OLLAMA_API_KEY=%s\n' "$(openssl rand -hex 32)" > .env
```

Do not use an easily guessed value, reuse a key from another service, commit
`.env`, or include the key in a URL. Treat the key like a password. The `.gitignore`
file excludes `.env` from Git, but verify it with `git status` before pushing.

To rotate the key, generate a new value in `.env` and recreate the proxy:

```powershell
docker compose up -d --force-recreate auth-proxy
```

Keep `.env` private. Requests must include the key in this form:

```http
Authorization: Bearer replace-with-your-key
```

For example:

```powershell
Invoke-RestMethod http://localhost:11434/api/tags -Headers @{ Authorization = "Bearer $env:OLLAMA_API_KEY" }
```

The Ollama container is not published directly; only the authenticated proxy
is exposed on port `11434`.