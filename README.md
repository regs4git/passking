# passking
Gerador e analisador de passwords PWA (offline, sem dependências externas).

- Tema claro/escuro (Material You, semente âmbar) guardado em localStorage (`pk-theme`); a password nunca é guardada.
- Lista de palavras: EFF Large Wordlist (7776) em `wordlist.js`; substituível por outra lista com >=7776 palavras únicas.
- CSP restrita por `<meta>`; único pedido de rede permitido: `api.pwnedpasswords.com` (opcional, k-anonymity).
- Ao alterar ficheiros da app, incrementar `CACHE_NAME` em `service-worker.js`.
