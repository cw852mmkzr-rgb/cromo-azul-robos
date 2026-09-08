# Cromo Azul Robôs

Monitor web de células robóticas da fábrica **Cromo Azul** — 6 células, robôs FANUC 800–805.

Aplicação single-file (`index.html`, HTML + CSS + JS puro) com backend Supabase e estética
glassmorphism neon azul/roxo. Deploy automático via Netlify + GitHub Pages no push da `main`.

## Acesso

- **HOST (edição)**: senha na tela inicial
- **Visualizador**: somente leitura, sem senha
- **Dashboard TV**: grade de todas as células com relógio, para telão de fábrica

## Rodar localmente

Qualquer servidor estático serve o `index.html`, por exemplo:

```bash
npx http-server . -p 8791 -c-1
```

Depois abra `http://localhost:8791`.

## Documentação

Contexto completo de arquitetura, dados e padrões em [`CLAUDE.md`](CLAUDE.md).
