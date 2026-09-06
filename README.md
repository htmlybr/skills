# Hospedagem de site grátis para agentes de IA

Hospedagem de site grátis para agentes de IA: o Claude, o Cursor, o Codex ou o Gemini CLI publicam sites HTML estáticos no HTMLy em segundos, direto da conversa, sem deploy manual. Este repositório reúne a **skill `htmly-publish`** (formato aberto [Agent Skills](https://agentskills.io)), o **plugin do Claude Code** e a **extensão do Gemini CLI**, todos apontando para o servidor MCP do [HTMLy](https://htmly.com.br).

O HTMLy é a hospedagem brasileira para HTML gerado por IA: conta gratuita, PIX, suporte em português. O servidor MCP entrega 4 ferramentas ao agente: `publish_site`, `list_sites`, `get_site` e `read_file`. Publicar é merge por padrão, então uma resposta truncada da IA nunca apaga página nenhuma.

*English: HTMLy is a Brazilian static hosting for AI-generated HTML. This repo ships the `htmly-publish` Agent Skill, a Claude Code plugin and a Gemini CLI extension, all wired to the HTMLy remote MCP server (OAuth). See the [English section](#english) below.*

## Conectar o servidor MCP (sem instalar nada)

O endpoint é `https://htmly.com.br/api/mcp`, com login OAuth no navegador.

| Cliente | Como |
|---|---|
| claude.ai / Claude Desktop | Configurações → Conectores → Adicionar conector personalizado → colar a URL → aprovar |
| Claude Code | `claude mcp add --transport http htmly https://htmly.com.br/api/mcp` e depois `/mcp` |
| Cursor | `mcp.json`: `{"mcpServers":{"htmly":{"url":"https://htmly.com.br/api/mcp"}}}` |
| Codex | `codex mcp add htmly --url https://htmly.com.br/api/mcp` e `codex mcp login htmly` |
| CI / headless | mesmo endpoint com `Authorization: Bearer <api_key>` (chave em htmly.com.br/profile) |

## Instalar a skill

A skill ensina o agente a usar as ferramentas com segurança: conferir vaga antes de publicar, editar só o que mudou, publicar em lotes quando o site é grande, marcar rascunho como não indexável.

```bash
npx skills add htmlybr/skills
```

Ou só no Claude Code:

```bash
mkdir -p ~/.claude/skills/htmly-publish
curl -sL https://raw.githubusercontent.com/htmlybr/skills/main/skills/htmly-publish/SKILL.md \
  -o ~/.claude/skills/htmly-publish/SKILL.md
```

## Instalar como plugin do Claude Code

O plugin junta o servidor MCP e a skill numa instalação só.

```bash
claude plugin marketplace add htmlybr/skills
claude plugin install htmly@htmlybr-skills
```

Depois rode `/mcp` para autorizar o HTMLy. Quando o plugin estiver no marketplace da comunidade da Anthropic, o comando passa a ser `/plugin install htmly@claude-community`.

## Instalar no Gemini CLI

```bash
gemini extensions install https://github.com/htmlybr/skills
```

## Estrutura

```
skills/htmly-publish/SKILL.md   # a skill (Agent Skills)
.claude-plugin/plugin.json      # manifesto do plugin do Claude Code
.mcp.json                       # servidor MCP do plugin
gemini-extension.json           # manifesto da extensão do Gemini CLI
```

## Limites que valem a pena saber

- Site leve cabe numa chamada (até ~90 KB de conteúdo); site grande vai em lotes de merge ou por ZIP no painel.
- Armazenamento por site depende do plano (Free 10 MB). Imagens pesadas: WebP ou URL externa.
- Sem PHP e sem execução no servidor. Formulário precisa de serviço externo.
- Extensões aceitas e erros comuns estão na skill e em https://htmly.com.br/docs/mcp.

## Contribuir

Issues e PRs são bem-vindos. Mudança na skill vale para todos os clientes de uma vez; teste com `skills-ref validate skills/htmly-publish` antes de abrir o PR.

---

## English

Free static site hosting for AI agents. **HTMLy** hosts static HTML sites at `{slug}.htmly.com.br` in seconds. The remote MCP server (`https://htmly.com.br/api/mcp`, OAuth) gives any agent four tools: `publish_site` (merge by default, so a truncated bundle never deletes pages), `list_sites`, `get_site` and `read_file`.

- Connect in claude.ai: Settings → Connectors → Add custom connector → paste the URL.
- Claude Code: `claude mcp add --transport http htmly https://htmly.com.br/api/mcp`, then `/mcp`.
- Skill: `npx skills add htmlybr/skills`.
- Plugin: `claude plugin marketplace add htmlybr/skills` then `claude plugin install htmly@htmlybr-skills`.
- Gemini CLI: `gemini extensions install https://github.com/htmlybr/skills`.

Docs: https://htmly.com.br/docs/mcp. License: MIT.
