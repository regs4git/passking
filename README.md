# PassKing

Gerador e avaliador de passwords em PWA: HTML, CSS e JavaScript simples, sem dependências nem passo de build. Instala-se no telemóvel e funciona offline (excepto a verificação HIBP, opcional).

**Demo:** https://regs4git.github.io/passking/

## Funcionalidades

**Gerador** — 4 modos, todos com `crypto.getRandomValues()` e rejeição de amostragem (sem viés de módulo):

| Modo | Por omissão | Intervalo | Entropia por omissão |
|---|---|---|---|
| Personalizada | 20 caracteres, 4 conjuntos activos | 8–64 | ≈126 bits |
| Passphrase | 6 palavras, separador `-` | 4–10 palavras | 77,5 bits |
| Pronunciável | 4 palavras × 3 sílabas CV + sufixo 10–99 | 2–8 palavras, 1–5 sílabas | 83,4 bits |
| Wi‑Fi | chave hexadecimal de 64 caracteres | hex 8–64; palavras 4–9 | 256 bits |

- Passphrase: lista EFF Large Wordlist (7776 palavras, 12,92 bits/palavra); números opcionais entre palavras; separadores `-` `.` `_` espaço.
- Wi‑Fi: 64 hex = PSK WPA2 de 256 bits usada directamente; em modo palavras, o resultado nunca excede 63 caracteres.
- Acções: copiar (o clipboard é limpo ao fim de 60 s), imprimir cartão de emergência, limpar.

**Avaliação de segurança** — estima a entropia por modelo de segmentos (palavras de dicionário, anos, sequências, repetições, teclado, leet simples, resto por força bruta), mostra o tempo médio até encontrar sob 4 perfis de ataque e lista os padrões detectados. Verificação opcional no Have I Been Pwned, só a pedido.

**Badge de segurança** junto de cada caixa de password (gerador e avaliação):

| Estado | Bits |
|---|---|
| 🟢 Segura | ≥ 60 |
| 🟡 Rever | 40 a < 60 |
| 🔴 Insegura | < 40 |

Clicar no badge do gerador abre a aba de avaliação com essa password carregada.

**Interface** — Material You (semente âmbar), tema claro/escuro (segue o sistema na primeira abertura; escolha guardada), barra de navegação inferior em ecrãs ≤ 860 px, opções avançadas e notas em caixas colapsáveis.

## Privacidade

- A password gerada ou escrita não é guardada (sem localStorage, URL ou histórico).
- O único dado persistido é a preferência de tema (`pk-theme`).
- Único pedido de rede da app: `api.pwnedpasswords.com`, apenas ao carregar em "Verificar". Envia só os 5 primeiros caracteres do SHA‑1 com `Add-Padding` (k‑anonymity).
- CSP por `<meta>`: `default-src 'none'`; scripts só de `'self'`.

## Estrutura

| Ficheiro | Função |
|---|---|
| `index.html` | Estrutura, estilos (tokens claro/escuro) e CSP |
| `app.js` | Geradores, analisador, HIBP, UI, tema, registo do service worker |
| `wordlist.js` | EFF Large Wordlist (7776 palavras) |
| `service-worker.js` | Cache stale‑while‑revalidate (`passking-v4`), só mesmo domínio |
| `manifest.json` | PWA (ícones `any` e `maskable`) |
| `icon-*.png`, `apple-touch-icon.png` | Ícones |
| `SPEC.md` | Especificação funcional |

## Publicar (GitHub Pages)

1. Fazer commit de todos os ficheiros na raiz do repositório.
2. Settings → Pages → *Deploy from a branch* → `main` / root.
3. **Ao alterar qualquer ficheiro da app, incrementar `CACHE_NAME` em `service-worker.js`**; caso contrário, os utilizadores ficam com a versão antiga em cache.

Para servir localmente: `python3 -m http.server 8080` (o service worker e o HIBP exigem `https` ou `localhost`).

## Limitações

- A estimativa de entropia é um modelo, não uma garantia: palavras fora do dicionário (≈8 mil entradas) contam como aleatórias.
- Em passwords aleatórias o analisador subestima 10–15% (trata substrings casuais como palavras).
- Os tempos de ataque são ilustrativos (100 mil milhões/s, 500 000/s, 10 000/s, 1 000/s) e variam com hardware e algoritmo.
- A lista de palavras da passphrase é em inglês.
