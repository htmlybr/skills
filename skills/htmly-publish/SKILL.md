---
name: htmly-publish
description: Publica sites HTML estáticos no HTMLy (htmly.com.br) e devolve a URL ao vivo. Use quando o usuário pedir para publicar, hospedar, colocar no ar, subir, compartilhar como link ou atualizar um site, uma landing page, um portfólio, um cardápio, um currículo ou qualquer HTML/CSS/JS gerado na conversa. Also use when the user asks to publish, host, deploy or share an HTML page as a live link, or to update a site already on HTMLy.
license: MIT
compatibility: Requires the HTMLy MCP server connected (https://htmly.com.br/api/mcp, OAuth) or an HTMLy API key. Works in Claude, Claude Code, Cursor, Codex and Gemini CLI.
metadata:
  author: htmlybr
  version: "1.0"
  homepage: https://htmly.com.br/mcp
---

# Publicar no HTMLy

O HTMLy hospeda sites estáticos em `{slug}.htmly.com.br`. Esta skill orienta o uso das 4 tools do servidor MCP. Se as tools `publish_site`, `list_sites`, `get_site` e `read_file` não aparecerem, o servidor não está conectado: siga "Conectar" no fim.

## Regras que evitam estrago

1. **Chame `list_sites` antes de publicar.** Ela diz o plano, quantos sites ainda cabem e o limite de armazenamento por site. Não crie site novo sem saber se há vaga.
2. **`publish_site` é MERGE por padrão.** Arquivo enviado sobrescreve; arquivo não enviado FICA. Nada é apagado por acidente. Para remover, liste em `delete_paths`. Só use `mode: "replace"` quando o usuário quiser reconstruir o site do zero, e nesse caso `expected_file_count` é obrigatório e precisa bater com o número de arquivos enviados.
3. **Confira `files_kept` na resposta.** É a lista do que já existia e não foi enviado. Se aparecer algo que o usuário não queria manter, faça uma segunda chamada só com `delete_paths`.
4. **Uma chamada não deve passar de ~90 KB de conteúdo.** Acima disso, publique em lotes: primeira chamada cria o site com `index.html`, as seguintes fazem merge com o restante. Imagens grandes vão por URL externa ou comprimidas em WebP.
5. **Nunca invente slug de terceiro.** Slug que pertence a outra conta devolve erro; escolha outro ou omita para gerar um aleatório.
6. **Rascunho ou preview: `is_indexable: false`.** O site fica fora dos buscadores até o usuário pedir para indexar.

## Fluxo A: site novo gerado na conversa

1. `list_sites` para conferir vaga e limite.
2. Gerar os arquivos. `index.html` é obrigatório na raiz. CSS e JS podem ser inline ou arquivos separados.
3. `publish_site` com:
   - `files`: lista de `{path, content}` para texto e `{path, content_base64}` para binário (exatamente um dos dois por arquivo).
   - `slug`: opcional, 3 a 63 caracteres, minúsculas, números e hífens.
   - `title`: nome do projeto para o painel (não aparece no site).
   - `meta_description`: até 160 caracteres, usada como description e og:description.
   - `source`: qual IA gerou o HTML (`claude`, `chatgpt`, `cursor`, `other_ai`).
4. Responder ao usuário com a URL retornada e o resumo de `files_written` e `storage_bytes`.

## Fluxo B: editar site que já está no ar

1. `get_site` com o slug para ver os arquivos que existem.
2. `read_file` em cada arquivo que vai mudar. Não leia mídia pesada sem necessidade.
3. Fazer a alteração e chamar `publish_site` só com os arquivos alterados. O merge mantém o resto.
4. Para renomear um arquivo: enviar o novo e listar o antigo em `delete_paths` na mesma chamada.

## Fluxo C: apagar arquivos sem mudar nada

`publish_site` com `slug` e apenas `delete_paths`. `index.html` não pode ser apagado. Path inexistente falha a chamada inteira e nada é aplicado.

## O que o HTMLy injeta sozinho

Não é preciso adicionar: canonical, `og:url`, pixel de analytics (server-side, sem cookie), e o selo "Made with HTMLy" no plano Free. Meta tags preenchidas no painel vencem as do HTML; campo vazio mantém as do HTML.

## Limites e validações (evite o erro antes de acontecer)

- Extensões aceitas: html, css, js, json, xml, svg, png, jpg, jpeg, gif, webp, ico, woff, woff2, ttf, eot, otf, pdf, mp4, webm, ogg, mp3, wav, txt, csv. `.webmanifest` NÃO é aceito: use `manifest.json`.
- O tipo real do arquivo é conferido, não só a extensão. Um `.js` vazio ou um `.png` que é HTML são recusados.
- Sem PHP e sem execução no servidor. Formulário precisa de serviço externo.
- Armazenamento por site depende do plano (Free 10 MB). O erro de limite vem com o link de upgrade; repasse ao usuário, não tente contornar.
- Plano Free: 1 site e 3 atualizações por dia. Se o erro do dia mencionar um bônus disponível, pergunte ao usuário antes de reenviar com `use_bonus: true`; ele é único por conta.
- Site suspenso não aceita escrita. Oriente a contestar pelo suporte.

## Erros comuns e o que fazer

| Erro | Ação |
|---|---|
| slug já em uso / reservado | escolher outro ou omitir |
| bundle sem `index.html` na criação | adicionar `index.html` na raiz |
| `expected_file_count` não bate | recontar e reenviar; nada foi alterado |
| limite de sites do plano | dizer ao usuário quantos cabem e sugerir atualizar um existente |
| 401 | reconectar (OAuth) ou conferir a API key |
| 403 "closed beta" | a conta ainda não está liberada; pedir acesso em htmly.com.br/mcp |

## Conectar

- **claude.ai / Claude Desktop:** Configurações → Conectores → Adicionar conector personalizado → URL `https://htmly.com.br/api/mcp` → aprovar no HTMLy.
- **Claude Code:** `claude mcp add --transport http htmly https://htmly.com.br/api/mcp` e depois `/mcp` para autorizar.
- **Cursor:** em `mcp.json`: `{"mcpServers":{"htmly":{"url":"https://htmly.com.br/api/mcp"}}}`.
- **Codex:** `codex mcp add htmly --url https://htmly.com.br/api/mcp` e `codex mcp login htmly`.
- **CI ou sem navegador:** o mesmo endpoint aceita `Authorization: Bearer <api_key>`; a chave fica em htmly.com.br/profile.

Documentação completa: https://htmly.com.br/docs/mcp
