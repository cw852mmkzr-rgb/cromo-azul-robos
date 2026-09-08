# Cromo Azul Robôs

Sistema web de monitoramento de células robóticas para a fábrica **Cromo Azul**.

Projeto enxuto, inspirado na arquitetura do **Células Robóticas** (`cw852mmkzr-rgb/celulas-roboticas`),
mas com estética totalmente diferente (glassmorphism neon azul/roxo) e escopo reduzido.

## Idioma

Sempre responder e escrever textos em **português brasileiro**.

## Arquitetura

- **Frontend**: `index.html` — single-file, HTML + CSS + JS puro (sem build, sem framework)
- **Backend**: Supabase — **projeto próprio e separado** do Células Robóticas
- **Deploy**: push na branch `main` → Netlify + GitHub Pages atualizam automaticamente
- **Repositório**: `cw852mmkzr-rgb/cromo-azul-robos` (público)

## Backend — Supabase

- **Projeto**: `cromo-azul-robos` — ref `zgetqjplwueeychpnshv` (região `sa-east-1`)
- **URL**: `https://zgetqjplwueeychpnshv.supabase.co`
- Chave **anon** (pública) embutida no `index.html` — é a chave de cliente, pode ficar no front

### Padrão "linha única com colunas JSON"

- Tabela `dados`, linha `id = 'main'`
- Cada funcionalidade vive numa coluna `jsonb` própria
- Coluna atual: **`conteudo`** — objeto com todas as células, chaveado pelo número do robô

```
dados
├── id            text  (PK)      → sempre 'main'
├── conteudo      jsonb           → { "800": {...célula}, "801": {...}, ... }
├── imagens       jsonb           → reservado p/ fotos futuras (base64), fora do conteudo
└── atualizado_em timestamptz
```

- **RLS** ativado. Política única `anon full access dados` (SELECT/INSERT/UPDATE/DELETE para o role `anon`) —
  padrão de app público interno, sem login de usuário Supabase.
- Leitura/escrita via **REST** (`/rest/v1/dados?id=eq.main`), com `HEADERS_SB` (apikey + Bearer anon).

### Modelo de dados de uma célula

```js
{
  nome, modelo, celula, cliente,
  status,            // Produzindo | Setup | Aguardando | Manutenção | Parado
  peca, codigo_peca, operador, turno, meta_hora, tempo_peca,
  eletrodo, tipo_eletrodo, corrente, tempo_solda, pre_pressao, pontos_solda,
  membro1, membro2, membro3, obs,
  ultima_troca, historico_eletrodo: [ { data, eletrodo, tipo } ]
}
```

## Dados da fábrica

- **6 células robóticas**, robôs numerados de **800 a 805**
- Todos **FANUC** — o campo `modelo` é editável no sistema (definir modelo real quando confirmado)

## Escopo (só isto)

- **Módulo Células Robóticas** — cadastro/config das 6 células e monitoramento
- **Tela de intro/login** animada (glassmorphism neon)
- **Backup / Restaurar** backup (JSON local)
- **Modo HOST** (edição) + **Visualizador** (somente leitura) + **Dashboard TV**

### NÃO incluir (por enquanto)

Mapa de Produção · Gabaritagem · Controle de Componentes · Equipes · Relatórios ·
Metas Diárias · Base de conhecimento IA / MR.ROBOT · Bot WhatsApp / Edge Functions · Armário de Gabaritos.

## Acessos

- **HOST / Admin**: senha `cromo3102` (constante `ADMIN_PW` no `index.html`) — edita tudo, cria/exclui células, backup
- **Visualizador**: entra sem senha, somente leitura
- **Dashboard TV**: entra sem senha, grade de todas as células com relógio ao vivo (para TV de chão de fábrica)

## Estética — Glassmorphism Neon Azul/Roxo

- Fundo: gradiente escuro azul-marinho → roxo, com **orbs desfocados** dando o glow neon
- Cards: glass real — `backdrop-filter: blur(20px)`, fundo branco 5–9%, borda branca ~16%, sombra suave
- Paleta (variáveis CSS em `:root`):
  - Azul elétrico `#4facfe` / `#00c6ff` · Roxo neon `#a855f7` / `#7c3aed` · Ciano `#22d3ee`
  - Gradiente padrão: `linear-gradient(135deg, #4facfe, #a855f7)`
- Tipografia: **Space Grotesk** (títulos, uppercase, letter-spacing) + **Inter** (corpo)
- Botões pill com gradiente neon e glow no hover; ícones outline finos; transições `cubic-bezier`
- Cores de status: Produzindo=verde · Setup=azul · Aguardando=roxo · Manutenção=âmbar · Parado=vermelho

## Padrões de código

1. **Single-file**: todo código novo vai no `index.html` — não criar arquivos separados sem confirmar
2. **Padrão visual**: manter as variáveis CSS, classes `.glass`, `.btn`, `.card`, `.field` e os tokens de cor
3. **Idioma**: todos os textos em português
4. **Salvar**: edições do HOST salvam com `saveDebounced()` (700 ms); ações pontuais chamam `saveToAPI()` direto
5. **Polling**: `loadFromAPI` a cada 5 s, com debounce de 2 s após qualquer ação do usuário e guarda de foco/modal
6. **Anti-flicker**: `renderDetail` compara uma assinatura (`_lastRenderSig`) e só re-renderiza se mudou
7. **Confirmação antes de excluir** qualquer célula
8. **Cache local**: `localStorage['cromo_cache']` para resposta instantânea offline

## ⚠️ Bugs visuais — vistoria obrigatória antes do commit

Estes bugs não podem entrar em produção. Antes de `git push`, verifique:

1. **Scroll subindo sozinho** — botões sempre `type="button"`; nunca `href="#"`; sem foco fora da viewport
2. **Tela piscando** — não re-renderizar o DOM inteiro sem necessidade (usar `_lastRenderSig`); não sobrescrever
   valores enquanto um input está focado (o polling já respeita isso)
3. **Scroll dentro de modal** — `body{overflow:hidden}` ao abrir; modal com scroll interno próprio (`max-height`)
4. **Modal não fecha** — deve fechar no X, no backdrop e no `Esc` (já implementado; manter)
5. **Body com scroll horizontal** — nunca; conteúdo largo scrolla dentro do próprio container (`overflow-x:auto`)
6. **Layout quebrando** — testar desktop, tablet e mobile (a lista de células vira faixa horizontal no mobile)
7. **Toast empilhando infinito** — limite de 3 e auto-dismiss (já implementado)

### Checklist rápido

- [ ] Abrir e navegar a aba/modal alterado; abrir/fechar modais várias vezes
- [ ] Testar desktop **e** mobile (a faixa de células rola na horizontal sem estourar a página)
- [ ] Rolar até o fim e voltar; clicar em vários botões — a tela pisca? Sobe sozinho?
- [ ] Console do navegador sem erros/warnings novos; nenhum `console.log` de debug esquecido
- [ ] Confirmar no Supabase (MCP) que os saves realmente chegaram no banco

## Fluxo Git

1. Rodar o checklist de bugs visuais acima
2. Commit com mensagem descritiva em português
3. Push na branch `main` → Netlify + GitHub Pages fazem deploy automático

## Como abordar tarefas

1. Ler o `index.html` antes de editar
2. Reaproveitar funções/estilos existentes antes de criar algo novo
3. Testar no navegador (login, HOST, viewer, dashboard) e confirmar persistência no Supabase
4. Rodar o checklist de bugs visuais antes do commit
