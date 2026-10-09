# PassKing — Especificação funcional

Versão da especificação: 2026‑10‑08 (cache `passking-v4`).

## 1. Objectivo e âmbito
PWA estática para gerar passwords e avaliar a sua robustez. Sem backend, sem contas, sem dependências de terceiros em execução. Prioridade ao telemóvel; em desktop mostra todo o conteúdo se couber.

Fora de âmbito: gestor de passwords, sincronização, armazenamento de segredos.

## 2. Requisitos funcionais

### 2.1 Aleatoriedade
- RF‑1 Toda a escolha aleatória usa `crypto.getRandomValues()` (Uint32).
- RF‑2 `secureRandomInt(max)` usa rejeição de amostragem: rejeita valores ≥ `2³² − (2³² mod max)`.
- RF‑3 O embaralhamento é Fisher‑Yates com `secureRandomInt`.
- RF‑4 A escolha de símbolos itera por pontos de código (`Array.from`), não por unidades UTF‑16.

### 2.2 Modos de geração

| Modo | Parâmetros (intervalo, omissão) | Saída | Entropia reportada |
|---|---|---|---|
| Personalizada | comprimento 8–64 (20); maiúsculas, minúsculas, números, símbolos (todos activos); símbolos `!@#$%^&*_-+=?.,:;`; evitar ambíguos (desligado; conjunto `0Oo1lIL|/'"` `` ` ``) | 1 carácter de cada conjunto activo + restantes do conjunto total, embaralhados | `len × log₂(tamanho do conjunto)` |
| Passphrase | palavras 4–10 (6); separador `-` `.` `_` espaço (`-`); números entre palavras (desligado, 10–99) | palavras capitalizadas | `n × log₂(7776)` + `(n−1) × log₂(90)` se números |
| Pronunciável | palavras 2–8 (4); sílabas 1–5 (3) | sílabas CV (17 consoantes × 5 vogais = 85), capitalizadas, unidas por `-`, sufixo `-NN` (10–99) | `sílabas × log₂(85) + log₂(90)` |
| Wi‑Fi (hex) | comprimento 8–64 (64) | caracteres 0–9 A–F | `4 × comprimento` bits; 64 = PSK de 256 bits |
| Wi‑Fi (palavras) | palavras 4–9 (8) | palavras capitalizadas unidas por `-`, ≤ 63 caracteres | `n × log₂(7776)` ajustada pela probabilidade de aceitação (≤ 63) |

- RF‑5 Wi‑Fi (palavras): regenerar até comprimento ≤ 63 (limite WPA2). O slider vai só até 9 palavras.
- RF‑6 Se nenhum conjunto estiver activo no modo personalizado, a saída é vazia.
- RF‑7 Valores por omissão de entropia: personalizada ≈126,1 bits; passphrase 77,5; pronunciável 83,4; Wi‑Fi hex 256; Wi‑Fi palavras (8) 102,5.

### 2.3 Acções
- RF‑8 **Gerar nova**: nova password no modo activo.
- RF‑9 **Copiar**: escreve no clipboard; ao fim de 60 s tenta escrever string vazia (não verifica se o conteúdo ainda é a password).
- RF‑10 **Imprimir cartão**: preenche um cartão A4 (password, data, tipo) e chama `window.print()`; o cartão é apagado no evento `afterprint` e em "Limpar".
- RF‑11 **Limpar**: apaga password gerada, contexto, cartão de impressão, badge e análise.

### 2.4 Avaliação de segurança
- RF‑12 Entrada: campo de password (oculto por omissão, com botão mostrar/ocultar) ou a password gerada via badge.
- RF‑13 Pesquisa por programação dinâmica sobre segmentos; o custo total é a soma dos custos dos segmentos (bits):
  - palavra de dicionário (≥ 3 letras, lista com ≈8 mil entradas: passwords comuns, nomes, vocabulário PT/EN, EFF): `log₂(posição)` + 1 bit por substituição leet + custo de capitalização (1 bit se tudo maiúsculas ou só inicial; senão até 4 bits);
  - dígitos: `n × log₂(10)`; ano 1900–2099: `log₂(130)`; sequência crescente/decrescente: máx. `log₂(200)`;
  - repetição de carácter (≥ 3): `log₂(pool × n)`;
  - sequência de teclado (≥ 4): `log₂(1000)`;
  - separador comum: `log₂(14)`;
  - restante: `log₂(pool)` por carácter (pool = 26 minúsculas, 26 maiúsculas, 10 dígitos, 32 símbolos, conforme presentes).
- RF‑14 Password exactamente na lista de comuns (ou leet trivial): ≈3,3 bits.
- RF‑15 Cadeia de ≥ 16 caracteres hexadecimais: 4 bits por carácter, sem análise de padrões.
- RF‑16 Passwords geradas pela app (contexto de geração) usam a entropia teórica do modo, não o analisador.
- RF‑17 Perfis de ataque (tempo médio = metade do espaço de pesquisa ÷ taxa):

| Perfil | Taxa (tentativas/s) |
|---|---|
| Hash rápido offline (omissão) | 10¹¹ |
| Router/Wi‑Fi WPA2 | 5 × 10⁵ |
| Hash lento offline | 10⁴ |
| Online com rate limiting | 10³ |

- RF‑18 Escalões de classificação (anel e barra): < 18 Comprometida; < 32 Muito fraca; < 50 Fraca; < 70 Média; < 90 Forte; ≥ 90 Muito forte.
- RF‑19 Alertas por nível (vermelho, laranja, amarelo) com lista de altura limitada e scroll interno; "Detalhe da estimativa" em caixa colapsável.

### 2.5 Badge de segurança de 3 estados

| Estado | Condição | Rótulo | Cores (fixas nos 2 temas) |
|---|---|---|---|
| Verde | bits ≥ 60 | Segura | fundo `#4ade80`, texto `#052e16` |
| Amarelo | 40 ≤ bits < 60 | Rever | fundo `#facc15`, texto `#422006` |
| Vermelho | bits < 40 | Insegura | fundo `#f87171`, texto `#450a0a` |
| Neutro | sem password | — | cinzento |

- RF‑20 Aparece à direita da caixa de password no gerador e na avaliação.
- RF‑21 No gerador é um botão: copia a password gerada para o campo da avaliação, mantém‑no oculto, limpa o resultado HIBP e muda para a aba de avaliação.
- RF‑22 Os cortes 40/60 seguem a leitura corrente de NIST SP 800‑63B; a norma não fixa limiares em bits, por isso são uma convenção desta app.
- RF‑23 O badge do gerador reflecte a entropia teórica do modo; o da avaliação usa o analisador. A mesma password pode ter estados diferentes nas duas.

### 2.6 Have I Been Pwned (opcional)
- RF‑24 Desligado por omissão; só activo após interruptor + botão "Verificar".
- RF‑25 Nunca há pedido enquanto se escreve.
- RF‑26 Pedido a `https://api.pwnedpasswords.com/range/<5 hex>` com cabeçalho `Add-Padding: true`; SHA‑1 via `crypto.subtle`. A comparação do sufixo é feita no browser; linhas de padding (contagem 0) ignoradas.
- RF‑27 Respostas obsoletas (password alterada durante o pedido) são descartadas.
- RF‑28 Em falha de rede mostra aviso; a avaliação restante não é afectada.

## 3. Interface

### 3.1 Estrutura
- Cabeçalho: logótipo, nome, estado de ligação (oculto em ≤ 860 px), botão de tema.
- 2 vistas: **Gerador** e **Avaliação de segurança**.
- Navegação: barra inferior fixa em ≤ 860 px (ícone + rótulo); abas no topo acima disso. Ambas sincronizadas.
- Gerador: controlo segmentado dos 4 modos → caixa de password + badge → 4 acções → "⚙️ Opções avançadas" (fechado) com os controlos do modo e nota informativa colapsável.
- Avaliação: duas colunas em desktop (entrada, tipos de caracteres, HIBP | medidor, perfil, tempo, alertas, detalhe); coluna única em telemóvel.
- Rodapé: "Piquinho Software — Crafted with altitude. Built with joy."

### 3.2 Tema
- Material You aproximado: papéis de cor (`surface`, `surface2`, `surface3`, `brass`/`brass2` como primário), cantos 12–18 px, superfícies tonais.
- Escuro por omissão no CSS; claro por `data-theme="light"`. Na primeira abertura segue `prefers-color-scheme`; depois, a escolha do utilizador (`localStorage: pk-theme`). `theme-color` e `color-scheme` actualizados.
- Contraste: texto sobre superfícies e badges com pares escuro‑sobre‑claro ou claro‑sobre‑escuro; alertas com cores de texto próprias no tema claro. Rácios AA não foram medidos formalmente.

### 3.3 Densidade
- Alvos de toque ≥ 38 px. Notas longas e detalhes em `<details>` fechados por omissão. Lista de alertas com `max-height` 190 px.
- Não é garantido zero scroll em todos os ecrãs; a vista de avaliação pode exigir scroll em telemóveis pequenos.

## 4. Segurança e privacidade
- SEG‑1 Nenhuma password é guardada em localStorage, sessionStorage, URL ou histórico. Persistido apenas: `pk-theme`.
- SEG‑2 CSP por `<meta>`: `default-src 'none'; script-src 'self'; style-src 'self' 'unsafe-inline'; img-src 'self' data:; connect-src https://api.pwnedpasswords.com; manifest-src 'self'; worker-src 'self'; base-uri 'none'; form-action 'none'`.
- SEG‑3 Sem scripts, fontes ou estilos de terceiros. O service worker não intercepta pedidos de outra origem (o HIBP vai directo à rede).
- SEG‑4 Limites conhecidos: a password permanece em memória JS e no DOM enquanto visível; o clipboard é limpo após 60 s sem verificação do conteúdo; extensões do browser com acesso à página podem lê‑la.

## 5. PWA
- `manifest.json`: `id` e `scope` `./`, `display: standalone`, `lang: pt-PT`, ícones 192 e 512 (`any`) e 512 (`maskable`).
- Service worker `passking-v4`: pré‑cache de `index.html`, `app.js`, `wordlist.js`, manifest e ícones; estratégia stale‑while‑revalidate; `skipWaiting` + `clients.claim`; limpeza de caches antigas na activação.
- Ao alterar ficheiros da app, incrementar `CACHE_NAME`.
- Requer `https` ou `localhost` (service worker, `crypto.subtle`, clipboard).

## 6. Critérios de aceitação
Verificados em jsdom e Node (lógica); a interface visual e o offline em dispositivo real não foram testados.

| # | Critério | Estado |
|---|---|---|
| A1 | Passphrase por omissão ≥ 77 bits (lista de 7776) | cumprido: 77,5 |
| A2 | Wi‑Fi (palavras) nunca > 63 caracteres | cumprido: máx. 63 em 3000 amostras por tamanho (4, 8, 9) |
| A3 | Chave hex de 64 caracteres avaliada como 256 bits | cumprido |
| A4 | Ano ou sequência no fim não anula o resto da password | cumprido: `Xk9#mPq2$vLw2024` → 73,7 bits |
| A5 | `password`, `Maria1985` → Insegura; `Tr0ub4dor&3` → Rever; `Xk9#mPq2$vLw2024` → Segura | cumprido |
| A6 | Badge do gerador abre a avaliação com a password carregada | cumprido |
| A7 | Navegação inferior e superior sincronizadas | cumprido |
| A8 | Disclosures fechadas por omissão; sem texto sobre processamento local | cumprido |
| A9 | Tema persistido e aplicado ao recarregar | cumprido (teste de escrita em localStorage) |
| A10 | Funciona offline como PWA instalada | por verificar em dispositivo |

## 7. Limitações conhecidas
- Dicionário do analisador ≈ 8 mil entradas; palavras fora dele contam como aleatórias.
- Em passwords aleatórias o analisador subestima 10–15% (em média 66,3 bits para 12 caracteres, teórico 75,6; 111,3 para 20, teórico 126,1).
- Geração personalizada: incluir 1 carácter de cada conjunto torna a distribuição ligeiramente não uniforme; o impacto é desprezável a 20 caracteres e maior a 8.
- Lista de palavras da passphrase em inglês; palavras portuguesas só no dicionário do analisador.
- Taxas de ataque ilustrativas; hardware multi‑GPU excede 10¹¹/s para hashes rápidos.
- Badge do gerador (entropia teórica) e da avaliação (modelo de padrões) podem divergir.

## 8. Evolução possível
- Acções do gerador como ícones numa só linha (≈ 50 px poupados).
- Lista de palavras em português (≥ 7776 entradas únicas) como opção.
- Dicionário maior ou carregado sob pedido para o analisador.
- Teste automático (Node/jsdom) no repositório.
