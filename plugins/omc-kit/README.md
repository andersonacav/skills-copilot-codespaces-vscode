# omc-kit

Instala e configura o [oh-my-claudecode](https://github.com/Yeachan-Heo/oh-my-claudecode) completo com um único comando: plugin, CLAUDE.md, CLI `omc`, HUD, agent teams e regras.

## Instalar (Claude Code no terminal, na web ou no app para Mac)

```
/plugin marketplace add andersonacav/skills-copilot-codespaces-vscode
/plugin install omc-kit@anderson
```

## Usar

```
/omc-kit:instalar-omc            # projeto atual (ou global, fora de um repositório git)
/omc-kit:instalar-omc global     # vale para todos os projetos da máquina
/omc-kit:instalar-omc local --sem-regras --sem-teams
```

Reinicie o Claude Code depois para o HUD e os agent teams entrarem em vigor.
