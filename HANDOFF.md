# Cases Mói — Protótipo navegável

Handoff do projeto para continuar em outra conta Claude. Cole este arquivo (ou o conteúdo dele) na nova conversa e o Claude terá todo o contexto pra seguir de onde parou.

## O que é

Um protótipo HTML navegável do board "Cases Moi" do Figma (portfólio do Mói Studio), com 49 telas exportadas como imagens, organizadas em categorias (Intro, Apps, Sites, Sistemas Web, Jogos Educativos, Branding, Fim). Funciona como uma apresentação clicável: setas/clique para avançar, um menu com grade de miniaturas, hotspots na tela de índice, e links externos clicáveis sobre algumas telas de sites.

Origem: board do Figma **"UI Design - Portifolio Webflow"**, arquivo `V6sEOw7NngaPuJF0AowIL4`, canvas "Cases Moi" (node `4531:196`). As imagens não vêm mais direto do Figma via MCP — a dona do projeto (Sthefany, Mói Studio) exporta os frames manualmente e coloca na pasta `imgs-new/`.

## Onde está tudo

Pasta local: `~/Downloads/cases-moi-prototype/`

```
cases-moi-prototype/
├── index.html          ← versão standalone (abre direto no navegador, arquivo completo com <html><head><body>)
├── artifact.html        ← versão pra publicar no Claude Artifact (sem <!DOCTYPE>/<html>/<head>/<body> — a plataforma envolve isso automaticamente)
├── img/                 ← 49 imagens já renomeadas e em uso pelo site (nomenclatura própria, ex: 01-capa.png)
├── imgs-new/             ← pasta onde a dona do projeto larga os exports novos/atualizados do Figma (nome original "Frame N.png")
├── fonts/                ← fontes originais completas (PP Migra + PP Monument Extended, todos os pesos) — não usadas diretamente
├── fonts-web/            ← só os 2 arquivos woff2 realmente usados no site (Regular + Black da PP Monument Extended)
└── HANDOFF.md            ← este arquivo
```

**IMPORTANTE:** `index.html` e `artifact.html` têm o mesmo conteúdo/lógica, só diferem no wrapper HTML. Toda alteração de estrutura/estilo/JS precisa ser feita **nos dois arquivos**.

## Artifact publicado

- URL: **https://claude.ai/artifact/3iHgiJo8xGUXvLdaK8oQQb** (dono: conta atual, sthefany@moistudio.com.br)
- Já compartilhado como "anyone with the link", mas o compartilhamento **fixa (pina) uma versão** — republicar não atualiza automaticamente o que quem já tem o link vê. É preciso reabrir o Share e atualizar a versão fixada manualmente pela interface (não existe isso via API/ferramenta).
- Se for publicar numa **conta nova**, o caminho é: nessa nova conta, usar a ferramenta de Artifact passando `file_path: artifact.html` (sem `url`, já que é um artifact novo) + o parâmetro `files` apontando pra pasta `img/` e `fonts-web/` pra subir as imagens e fontes junto. Ver comandos usados abaixo.

## Como o código funciona

Tudo em um único `<script>` no fim do arquivo.

### 1. `CATS` — estrutura de conteúdo
Array de categorias, cada uma com `key`, `label` e `slides` (lista de `{f: 'arquivo.png', label: 'Nome exibido'}`). A ordem no array = ordem de navegação. Estado atual:

- **Intro** (7 telas): capa, hi, quem somos, números importam, parceiros, processos, índice
- **Apps** (11 telas): capa + S3 Bank, GNDI, Drop-X, Eu Protegido, **Ipiranga** (01/02 cada)
- **Sites** (13 telas): capa + FTD, Banco BS2, Beegin, Keynapse, Fintecs, **Aurora Inova** (01/02 cada)
- **Sistemas Web** (7 telas): capa + MPWeek, Giga Consult, Saúde Controle (01/02 cada)
- **Jogos Educativos** (4 telas): capa + Fábrica de Apps, Microscópio, George Orwell
- **Branding** (6 telas): capa + SeuFit, Fintecs, Xipp, Gigaconsult, Aurora
- **Fim** (1 tela)

⚠️ **Case trocado:** o que era "Kikkin" nos Frames 21/22 virou **Ipiranga** — a dona do projeto re-exportou o Figma com esse case novo. Os arquivos já estão como `17-apps-ipiranga-01.png` / `18-apps-ipiranga-02.png`.

Pra adicionar uma tela nova no meio de uma categoria (feito uma vez com Aurora Inova, inserida depois de Fintecs): não precisa renumerar os arquivos seguintes — só nomear o novo arquivo com um sufixo (`29a-`, `29b-`) e inserir no ponto certo do array `slides`. A ordem de exibição é 100% definida pela ordem no array, não pelo nome do arquivo.

### 2. `INDEX_HOTSPOTS` — áreas clicáveis na tela de índice
A tela "07-index.png" tem uma lista (apps/sites/sistemas web/jogos educativos/branding) que funciona como menu — cada linha é uma área clicável (`top/left/width/height` em % da imagem) que pula pra capa daquela categoria. As posições foram medidas pixel a pixel direto na imagem real com Python/PIL (não adivinhadas), porque a versão exportada pela Sthefany tem proporções diferentes do print original do Figma.

### 3. `SLIDE_LINKS` — links externos abrindo em nova aba
Mapa `{ 'nome-do-arquivo.png': [{ top, left, width, height, url }] }`. Cada entrada cria um link clicável (`<a target="_blank">`) sobre a URL impressa na própria imagem do case. Estado atual:

```js
const SLIDE_LINKS = {
  '20-sites-ftd-01.png':      [{ url: 'https://portalconteudoaberto.com.br/' }],
  '26-sites-keynapse-01.png': [{ url: 'https://www.keynapse.com.br' }],
  '28-sites-fintecs-01.png':  [{ url: 'https://fintecs.com.br/' }],
  '29a-sites-aurora-01.png':  [{ url: 'https://aurorainova.com/' }],
  '47-fim.png':               [{ url: 'https://moistudio.com.br' }],
};
```

**Processo pra adicionar um novo link** (a dona do projeto vai mandando "Frame N: url" um a um):
1. Descobrir o nome do arquivo correspondente ao "Frame N" (ver tabela de mapeamento mais abaixo, ou perguntar).
2. Medir a posição do texto do link na imagem com Python/PIL (scan de pixels verdes/brilhantes na região onde a URL aparece — sempre perto do rodapé esquerdo do frame).
3. Adicionar a entrada em `SLIDE_LINKS` **nos dois arquivos** (`index.html` e `artifact.html`).
4. Validar desenhando a caixa por cima da imagem com PIL antes de publicar (evita depender do preview do navegador, que tem se mostrado instável nesta sessão).

### 4. Navegação
- Clique na metade esquerda/direita da imagem, ou setas `←`/`→` (agora quadradas, não circulares).
- Botão **"⣩ INDEX"** no topo direito (substituiu o antigo ícone de hambúrguer + a pill verde "index" que existia embaixo, ambos removidos a pedido) abre o menu com grade de miniaturas de tudo.
- Atalhos de teclado: `←`/`→` navegam, `M` abre o menu, `Esc` fecha.

## Decisões de design / gotchas já resolvidos

1. **`position:fixed` quebra dentro do Artifact.** A plataforma do Claude Artifact parece renderizar o conteúdo dentro de um contêiner escalado/transformado (pra permitir zoom na preview), o que muda a referência de `position:fixed`. Solução: tudo usa um wrapper `#app` com `position:relative` e os elementos internos (`#stage`, `#topbar`, `#bottombar`, `#menu`) usam `position:absolute` contra esse wrapper, não contra o viewport real.
2. **Faixa branca nos slides** — inicialmente pensei ser bug de CSS (testei remover `box-shadow`, não era isso). A causa real: a própria imagem exportada tinha a faixa (era problema no arquivo, não no código). Resolvido quando a dona reexportou os frames.
3. **Hotspots do índice desalinhados** — aconteceu porque as % foram medidas em cima do print de baixa resolução do Figma (1400×788), e a imagem final exportada tem proporções diferentes (painel mais estreito). Sempre medir em cima da imagem **final** que está de fato em uso, com PIL, nunca de cabeça.
4. **Link compartilhado não atualiza sozinho** — ver seção "Artifact publicado" acima.
5. **Fonte PP Monument Extended** — a Sthefany tinha a pasta de fontes completa (`fonts/Monument Extended/*.woff2`, todos os pesos). Só usamos Regular (400) e Black (800) — os outros pesos (`font-weight:600`/`700` que existiam antes) foram normalizados pra 800 porque só temos essas duas variações carregadas.
6. **Hook `impeccable` sinaliza `broken-image` recorrente** no `<img id="frame-img" src="">` — é falso positivo, esse `src` é preenchido via JS assim que a página carrega (é o elemento principal do visualizador de slides). Não precisa mexer.
7. **Preview do Claude Code (Browser pane) trava/esconde às vezes** durante testes locais — quando acontece, uso `javascript_tool` pra ler `getBoundingClientRect()` dos elementos, ou desenho os hotspots com Python/PIL direto em cima da imagem real pra validar sem depender do navegador.

## Publicando num servidor próprio (Hostinger)

Estava em andamento: a dona quer publicar em hospedagem própria (Hostinger) além do/em vez do Artifact. Precisa dela: host/IP, usuário, senha, porta (FTP geralmente 21, SFTP no Hostinger geralmente 65002) e pasta de destino (`public_html/` ou uma subpasta). Depois disso, subir a pasta inteira `index.html` + `img/` + `fonts-web/` (não precisa `imgs-new/`, `fonts/` nem `artifact.html`).

## Checklist antes de qualquer novo deploy

1. Rodar a checagem de hash (`imgs-new/Frame N.png` vs `img/nome-final.png`) pra achar imagens que a dona atualizou sem avisar explicitamente — já aconteceu uma vez (Kikkin virou Ipiranga silenciosamente).
2. Testar localmente (`python3 -m http.server` na pasta do projeto) antes de publicar.
3. Publicar `artifact.html` com `Artifact` tool, passando `url` do artifact existente pra atualizar o mesmo link (ou sem `url` se for criar um novo, como no caso da conta nova).
4. Avisar que o link compartilhado pode precisar ser reatualizado manualmente pela interface (ver gotcha #4).

## Tabela de mapeamento Frame (Figma) → arquivo final

| Frame Figma | Arquivo em `img/` | Case |
|---|---|---|
| Frame 1 | 01-capa.png | Capa |
| Frame 2 | 02-hi.png | Hi |
| quem somos | 03-quemsomos.png | Quem somos |
| Frame 3 | 04-bignumbers.png | Números importam |
| Frame 4 | 05-partners.png | Parceiros |
| Frame 5 | 06-processos.png | Processos |
| Frame 11 | 07-index.png | Índice |
| Frame 12 | 08-apps-cover.png | Capa Apps |
| Frame 13/14 | 09/10-apps-s3bank | S3 Bank |
| Frame 15/16 | 11/12-apps-gndi | GNDI |
| Frame 17/18 | 13/14-apps-dropx | Drop-X |
| Frame 19/20 | 15/16-apps-euprotegido | Eu Protegido |
| Frame 21/22 | 17/18-apps-ipiranga | **Ipiranga** (era Kikkin) |
| Frame 23 | 19-sites-cover.png | Capa Sites |
| Frame 24/25 | 20/21-sites-ftd | FTD |
| Frame 26/27 | 22/23-sites-bancobs2 | Banco BS2 |
| Frame 28/29 | 24/25-sites-beegin | Beegin |
| Frame 30/31 | 26/27-sites-keynapse | Keynapse |
| Frame 32/33 | 28/29-sites-fintecs | Fintecs |
| Frame 33.1/33.2 | 29a/29b-sites-aurora | Aurora Inova (inserida depois do Fintecs) |
| Frame 34 | 30-sistemas-cover.png | Capa Sistemas |
| Frame 35/36 | 31/32-sistemas-mpweek | MPWeek |
| Frame 37/38 | 33/34-sistemas-giga | Giga Consult |
| Frame 39/40 | 35/36-sistemas-saude | Saúde Controle |
| Frame 41 | 37-games-cover.png | Capa Jogos |
| Frame 42 | 38-games-fabricadeapps.png | Fábrica de Apps |
| Frame 43 | 39-games-microscopio.png | Microscópio |
| Frame 44 | 40-games-george.png | George Orwell |
| brading | 41-branding-cover.png | Capa Branding |
| Frame 9 | 42-branding-seufit.png | SeuFit |
| Frame 8 | 43-branding-fintecs.png | Fintecs (branding) |
| Frame 10 | 44-branding-xipp.png | Xipp |
| Frame 6 | 45-branding-gigaconsult.png | Gigaconsult |
| Frame 7 | 46-branding-aurora.png | Aurora (branding) |
| Frame 45 | 47-fim.png | Fim |
