---
name: instalar-omc
description: Instala e configura o oh-my-claudecode (OMC) completo na sessão atual — plugin, setup do CLAUDE.md, CLI omc, HUD, regras e agent teams. Use quando o usuário pedir para instalar, reinstalar ou reparar o OMC, ou chamar /omc-kit:instalar-omc.
argument-hint: "[local|global] [--sem-regras] [--sem-teams]"
disable-model-invocation: true
---

# Instalar oh-my-claudecode completo

Execute os passos abaixo em ordem, relatando ao usuário (em português do Brasil) o resultado de cada um. Cada passo é idempotente: verifique antes e pule o que já estiver feito. Nunca invente resultado — se um passo falhar, mostre o erro e siga para o próximo quando ele não depender do anterior.

## Argumentos

- `local` (padrão dentro de um repositório git): configura o projeto atual (`.claude/`), versionável.
- `global` (padrão fora de um repositório git): configura `~/.claude/`, vale para todos os projetos da máquina.
- `--sem-regras`: não copia os templates de regras.
- `--sem-teams`: não ativa os agent teams.

Defina `ALVO` a partir dos argumentos. Se não houver argumento, use `local` quando `git rev-parse --is-inside-work-tree` der certo e `global` caso contrário. Defina `CFG="${CLAUDE_CONFIG_DIR:-$HOME/.claude}"` e `SETTINGS` como `.claude/settings.json` (local) ou `$CFG/settings.json` (global).

## 1. Plugin oh-my-claudecode

Verifique com `claude plugin list`. Se `oh-my-claudecode@omc` não aparecer:

```bash
claude plugin marketplace add Yeachan-Heo/oh-my-claudecode
claude plugin install oh-my-claudecode@omc --scope <project se ALVO=local, user se ALVO=global>
```

Em ALVO=local, garanta também que `SETTINGS` declare o marketplace e o plugin (mescle com `jq`, preservando o resto):

```json
{
  "extraKnownMarketplaces": { "omc": { "source": { "source": "github", "repo": "Yeachan-Heo/oh-my-claudecode" } } },
  "enabledPlugins": { "oh-my-claudecode@omc": true }
}
```

Defina `OMC_ROOT` como o diretório de versão mais alta em `$CFG/plugins/cache/omc/oh-my-claudecode/` (`ls -d .../[0-9]* | sort -V | tail -1`). Se não existir, pare e peça ao usuário para reiniciar o Claude Code e rodar o comando de novo.

## 2. Setup do CLAUDE.md (coordenador oficial do OMC)

Não escreva o CLAUDE.md à mão. Rode:

```bash
bash "$OMC_ROOT/scripts/setup-claude-md.sh" <ALVO>
```

Confirme que o arquivo gerado (`.claude/CLAUDE.md` ou `$CFG/CLAUDE.md`) contém `OMC:START` e `OMC:END`. Se o global já existir sem os marcadores, pergunte ao usuário se deve sobrescrever (`overwrite`) ou preservar (`preserve`) antes de rodar.

## 3. CLI `omc`

```bash
command -v omc || npm install -g oh-my-claude-sisyphus
omc --version
```

## 4. HUD (barra de status)

```bash
mkdir -p "$CFG/hud/lib"
cp "$OMC_ROOT/scripts/lib/hud-wrapper-template.txt" "$CFG/hud/omc-hud.mjs"
cp "$OMC_ROOT/scripts/lib/config-dir.mjs" "$CFG/hud/lib/config-dir.mjs"
chmod 755 "$CFG/hud/omc-hud.mjs"
```

Mescle em `$CFG/settings.json` (sempre o do usuário, nunca o do projeto), preservando o resto:

```json
{ "statusLine": { "type": "command", "command": "node ${CLAUDE_CONFIG_DIR:-$HOME/.claude}/hud/omc-hud.mjs" } }
```

Se já existir um `statusLine` que não seja do OMC, pergunte antes de substituir.

## 5. Agent teams (pule com `--sem-teams`)

Mescle em `SETTINGS`: `{"env": {"CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"}}`.
Mescle em `$CFG/.omc-config.json`: `{"team": {"ops": {"maxAgents": 3, "defaultAgentType": "claude", "monitorIntervalMs": 30000, "shutdownTimeoutMs": 15000}}}`.

## 6. Regras (pule com `--sem-regras`)

Copie `$OMC_ROOT/templates/rules/*.md` (exceto `README.md`) para `.claude/rules/` (local) ou `$CFG/rules/` (global). Não sobrescreva arquivos que já existam com conteúdo diferente sem perguntar.

## 7. Marcar setup como concluído

```bash
V=$(grep -m1 'OMC:VERSION:' <CLAUDE.md do passo 2> | sed -E 's/.*OMC:VERSION:([^ ]+).*/\1/')
bash "$OMC_ROOT/scripts/setup-progress.sh" complete "$V"
```

## 8. Relatório final

Liste em uma tabela curta: plugin (versão), CLAUDE.md, CLI omc (versão), HUD, agent teams, regras — cada um com ✅ feito, ⏭️ já existia ou ❌ falhou (com o erro). Diga que é preciso reiniciar o Claude Code para o HUD e os agent teams entrarem em vigor. Em ALVO=local, liste os arquivos alterados que podem ser commitados; não faça commit sem o usuário pedir.
