# Framework do Portuguese Notebook

Descrição técnica e visual do site **Portuguese Notebook** (https://filustrias.github.io/Portuguese/), para reaproveitar em outro projeto. O código completo está no repositório https://github.com/filustrias/Portuguese, na pasta `docs/.vitepress/`.

Os nomes internos (componentes, variáveis CSS, chaves) estão em português, herdados do projeto de origem. O conteúdo do site está em inglês.

---

## 1. Base técnica

| Item | Valor |
|:--|:--|
| Gerador | **VitePress 1.6** (Vue 3 + Vite), tema padrão estendido |
| Módulos | `"type": "module"` no `package.json` |
| Pasta do conteúdo | `docs/` (um `.md` por página) |
| Configuração | `docs/.vitepress/config.mts` |
| Tema | `docs/.vitepress/theme/` |
| Publicação | GitHub Pages via **GitHub Actions** (a cada push no `main`) |
| Node | 22 |

### package.json

```json
{
  "private": true,
  "type": "module",
  "scripts": {
    "dev": "vitepress dev docs",
    "build": "vitepress build docs",
    "preview": "vitepress preview docs"
  },
  "devDependencies": {
    "@fontsource-variable/fraunces": "^5.3.0",
    "@fontsource-variable/inter": "^5.3.0",
    "@fontsource-variable/literata": "^5.3.0",
    "@fontsource-variable/source-sans-3": "^5.3.1",
    "vitepress": "^1.6.4"
  }
}
```

### .gitignore

```
node_modules/
docs/.vitepress/dist/
docs/.vitepress/cache/
```

### Estrutura de pastas

```
docs/
  index.md                     página inicial (layout: home)
  pronunciation.md             guia de pronúncia
  pimsleur_portuguese_NN.md    uma página por lição
  textbook-vs-everyday.md      tabela de referência
  imagens/azulejo.svg          ilustração da capa
  .vitepress/
    config.mts
    theme/
      index.ts                 entrada do tema (fontes, slots, componentes)
      estilo.css               todas as regras visuais
      latinizado-markdown.ts   plugin markdown-it (coluna coberta + IPA)
      latinizado.ts            estado compartilhado dos trechos cobertos
      Lat.vue                  trecho coberto, revelado ao clicar
      LatTodos.vue             botão "Show/Hide all pronunciation"
      BarraAula.vue            barra fixa no topo da lição (tocador + botão)
      Tocador.vue              tocador de áudio
      audios.ts                IDs dos MP3 no Google Drive
      Leitura.vue              painel "Aa" (fonte e tamanho)
      Capa.vue                 imagem da página inicial
.github/workflows/deploy.yml
```

---

## 2. Configuração do VitePress (`config.mts`)

- `lang: 'en-US'`, `title`, `description`.
- `base: '/NomeDoRepo/'`: obrigatório no GitHub Pages de projeto. Precisa ser exatamente o nome do repositório.
- `cleanUrls: false`: as URLs terminam em `.html`, o que evita 404 no GitHub Pages.
- **Script no `head`**: aplica a fonte e o tamanho salvos no navegador antes da página aparecer, para não piscar:

```js
head: [
  ['script', {}, `try{var d=document.documentElement,f=localStorage.getItem('leitura-fonte'),t=localStorage.getItem('leitura-tamanho');if(f)d.dataset.fonte=f;if(t)d.dataset.tamanho=t}catch(e){}`]
],
```

- **Plugin markdown**: `markdown: { config: (md) => md.use(latinizadoNasTabelas) }`.
- **Lista de lições gerada por número**: o mesmo array alimenta o menu do topo e a barra lateral.

```js
const totalLicoes = 20
const licoes = Array.from({ length: totalLicoes }, (_, i) => {
  const n = String(i + 1).padStart(2, '0')
  return { text: `Lesson ${n}`, link: `/pimsleur_portuguese_${n}` }
})
```

- `themeConfig`:
  - `nav` com um menu suspenso das lições;
  - `sidebar` em três grupos (Start here, Pimsleur, Reference);
  - `socialLinks` (GitHub) e `editLink` apontando para o repositório;
  - `search: { provider: 'local' }`;
  - `outline: { level: [2, 3] }`;
  - rótulos traduzidos: `docFooter`, `sidebarMenuLabel: 'Chapters'`, `notFound`.

---

## 3. Tipografia

Todas as fontes vêm do **Fontsource** em versão *variable*. São empacotadas pelo Vite, sem chamada ao Google Fonts. Importação em `theme/index.ts`:

```ts
import '@fontsource-variable/source-sans-3'
import '@fontsource-variable/source-sans-3/wght-italic.css'
import '@fontsource-variable/literata'
import '@fontsource-variable/literata/wght-italic.css'
import '@fontsource-variable/inter'
import '@fontsource-variable/inter/wght-italic.css'
import '@fontsource-variable/fraunces'
```

| Uso | Fonte | Observação |
|:--|:--|:--|
| Texto (padrão) | **Source Sans 3** | humanista e legível; tem IPA completo |
| Texto (opção serifada) | **Literata** | escolhida no painel "Aa" |
| Texto (opção) | **Inter** | escolhida no painel "Aa" |
| Títulos (h1, h2, hero) | **Fraunces** | serifa expressiva, `letter-spacing: -0.02em` |
| Transcrições IPA | Source Sans 3, fixa | não muda com o painel, para os símbolos fonéticos nunca faltarem |

### Variáveis

```css
:root {
  --vp-font-family-base: 'Source Sans 3 Variable', ui-sans-serif, system-ui, 'Segoe UI', sans-serif;
  --fonte-titulos: 'Fraunces Variable', Georgia, 'Times New Roman', serif;
  --tamanho-texto: 17px;
}
html[data-fonte='serif'] { --vp-font-family-base: 'Literata Variable', Georgia, 'Times New Roman', serif; }
html[data-fonte='inter'] { --vp-font-family-base: 'Inter Variable', ui-sans-serif, system-ui, sans-serif; }
html[data-tamanho='p'] { --tamanho-texto: 15px; }
html[data-tamanho='g'] { --tamanho-texto: 20px; }
```

### Escala

| Elemento | Tamanho | Peso | Entrelinha |
|:--|:--|:--|:--|
| h1 | 34px (celular) / 42px (≥ 768px) | 700 | 1.2 |
| h2 | 23px, com **fio preto de 2px acima** | 600 | padrão |
| h3 | padrão do tema | 700 | padrão |
| Texto corrido | `var(--tamanho-texto)`: 15 / **17** / 20px | 400 | 1.7 |
| Tabelas | 0.92em | 400 | 1.55 |
| Frase do hero | 30px / 40px (≥ 640px) | | 1.25 |
| Botões pequenos | 12 a 13px | 600 a 700 | |

---

## 4. Cores

A identidade é **papel creme com vermelho** no modo claro e **quase preto com vermelho claro** no modo escuro. Bordas são pretas e sólidas, e as sombras são deslocadas e sem desfoque, num estilo neobrutalista suave.

### Modo claro (`:root`)

| Token | Valor | Uso |
|:--|:--|:--|
| `--vp-c-bg` | `#fbf8f1` | fundo (creme) |
| `--vp-c-bg-alt` | `#f2ede1` | barra lateral |
| `--vp-c-bg-soft` | `#ebe4d4` | blocos, linhas alternadas |
| `--vp-c-bg-elv` | `#fffdf8` | painéis flutuantes |
| `--vp-c-text-1` | `#121212` | texto principal, bordas fortes |
| `--vp-c-text-2` | `#383838` | texto secundário |
| `--vp-c-text-3` | `#5c5c5c` | texto terciário |
| `--vp-c-divider` | `#d6ccb8` | divisórias |
| `--vp-c-border` | `#bfb39b` | bordas de tabela |
| `--vp-c-gutter` | `#e3dac7` | |
| `--vp-c-brand-1` / `-3` | `#b3121f` | **vermelho de destaque** |
| `--vp-c-brand-2` | `#8f0e19` | vermelho escuro (hover) |
| `--vp-c-brand-soft` | `rgba(179, 18, 31, 0.12)` | fundos suaves |
| `--vp-button-brand-bg` | `#b3121f` (hover `#8f0e19`, ativo `#6e0a13`) | botões |
| `--lat-listra-a` / `-b` | `#e2d8c3` / `#d5c8ad` | listras do trecho coberto |

### Modo escuro (`.dark`)

| Token | Valor |
|:--|:--|
| `--vp-c-bg` | `#0b0b0c` |
| `--vp-c-bg-alt` | `#121214` |
| `--vp-c-bg-soft` | `#1b1b1e` |
| `--vp-c-bg-elv` | `#161618` |
| `--vp-c-text-1` / `-2` / `-3` | `#f5f1e8` / `#cbc6ba` / `#9c978b` |
| `--vp-c-divider` / `--vp-c-border` | `#2a2a2e` / `#3d3d42` |
| `--vp-c-brand-1` / `-2` / `-3` | `#ff5c5c` / `#ff7d7d` / `#d0202e` |
| `--vp-c-brand-soft` | `rgba(255, 92, 92, 0.16)` |
| `--vp-button-brand-bg` | `#d0202e` (hover `#e2323f`, ativo `#a8161f`) |
| `--lat-listra-a` / `-b` | `#26262b` / `#1c1c20` |

Para trocar o tema de cor em outro projeto, basta trocar os quatro valores de `brand` e as duas cores de listra; o resto é neutro.

---

## 5. Linguagem visual

- **Borda forte**: `2px solid var(--vp-c-text-1)` em tocador, botões, painel, cards da página inicial e imagem da capa.
- **Sombra deslocada em vermelho**, sem desfoque:
  - `3px 3px 0 var(--vp-c-brand-1)` nos botões e no tocador;
  - `4px 4px 0` no painel "Aa";
  - `8px 8px 0` na imagem da capa.
- **Raios pequenos**: 3 a 6px (8px na moldura do áudio; 50% no botão play).
- **Cabeçalho de tabela invertido**: fundo `--vp-c-text-1`, texto `--vp-c-bg`, peso 600.
- **h2 com fio superior** de 2px na cor do texto, separando as seções como num caderno.
- **Hover**: a borda e o texto passam para o vermelho de destaque.
- **Foco visível**: `outline: 2px solid var(--vp-c-brand-1); outline-offset: 2px`.
- **Blocos personalizados** (`::: details`) com barra esquerda de 4px.
- **Página inicial**:
  - hero com o nome em vermelho e a frase em Fraunces;
  - imagem própria (SVG) com borda e sombra vermelha, legenda em 12px;
  - cards (`features`) com borda de 2px e ícone em texto, em Fraunces vermelho (por exemplo "Aa", "01–20", "≠").
- **Animação** só no botão play enquanto o áudio carrega (pulso de opacidade), desligada com `prefers-reduced-motion`.

---

## 6. Componentes

### Trecho coberto (`Lat.vue` + `latinizado.ts` + `LatTodos.vue`)

- **O que faz**: um `<button>` que esconde o texto atrás de listras diagonais até ser clicado. Serve para treinar: tenta-se ler primeiro e depois se confere.
  - Coberto: `repeating-linear-gradient(135deg, listra-a 0 6px, listra-b 6px 12px)`, com o texto em `visibility: hidden`, que mantém o tamanho do trecho.
  - Acessibilidade: `aria-expanded` e `aria-label="Show pronunciation"` enquanto coberto.
- **Uso no texto**: `*termo* (<Lat>[IPA]</Lat> - tradução)`.
- **Mostrar tudo**: `latinizado.ts` guarda todos os trechos da página num `Set` reativo, e o botão "Show all / Hide all" (`LatTodos.vue`) abre ou fecha todos de uma vez.

### Plugin markdown (`latinizado-markdown.ts`)

- **Coluna coberta**: em qualquer tabela cuja coluna se chame **IPA** (ou **Pronunciation**), envolve cada célula do corpo em `<Lat>`. Assim a tabela continua sendo Markdown puro.
- **Acento tônico**: insere um *word joiner* (U+2060) antes de cada `ˈ` que fica no meio da palavra. Sem isso, o navegador pode quebrar a linha antes do acento e partir a palavra.

### Barra da lição (`BarraAula.vue`)

- Aparece quando a página tem `unidade: N` no frontmatter.
- Fica fixa no topo (`position: sticky; top: nav + 12px`, ou 56px abaixo de 960px) e reúne o tocador e o botão "Show all".
- No celular (≤ 640px) só o tocador fica fixo; o botão rola com o texto.

### Tocador de áudio (`Tocador.vue` + `audios.ts`)

- **Controles**: tocar/pausar (círculo vermelho de 36px), −15 s e +15 s, barra de posição, tempo `m:ss / m:ss` com números tabulares e velocidade (0.75×, 1×, 1.25×).
- **Retomada**: guarda a posição de cada lição no `localStorage` a cada 5 s e volta a ela ao reabrir.
- **Origem do áudio**: lê o MP3 em `drive.usercontent.google.com/download?id=…`. Se o arquivo for privado e falhar, troca para o tocador do próprio Drive (`/file/d/ID/preview`, num iframe recortado em 56px de altura) e mostra um link "Open in Google Drive".
- **Cadastro**: `audios.ts` mapeia o número da unidade para o ID do arquivo no Drive.

### Painel de leitura (`Leitura.vue`)

- Botão "Aa" (em Literata, com borda de 2px) na barra do topo, inserido pelo slot `nav-bar-content-after`.
- Abre um painel de 270px com três fontes (Source Sans, Literata, Inter) e três tamanhos (A−, A, A+).
- A escolha vai para `data-fonte` e `data-tamanho` no `<html>` e é salva no `localStorage`.
- Fecha ao clicar fora ou com Esc.

### Capa (`Capa.vue`)

- Imagem da página inicial, inserida pelo slot `home-hero-image`, com `alt` descritivo e legenda.
- O fundo degradê padrão do hero foi removido (`.image-bg { display: none }`).

### Ligação no tema (`theme/index.ts`)

```ts
export default {
  extends: DefaultTheme,
  Layout() {
    return h(DefaultTheme.Layout, null, {
      'home-hero-image': () => h(Capa),
      'doc-before': () => h(BarraAula),
      'nav-bar-content-after': () => h(Leitura)
    })
  },
  enhanceApp({ app }) {
    app.component('Lat', Lat)
  }
} satisfies Theme
```

---

## 7. Modelo de página de lição

```markdown
---
unidade: 11
---

# Pimsleur — Lesson 11

## Dialogue

Uma frase de contexto.

| Who | Portuguese | IPA | English |
|:--|:--|:--|:--|
| Customer | Frase original. | [transcrição &#124; com pausa ‖ fim de frase] | Tradução. |

## Concepts

**Título curto.** Explicação, com exemplos: *termo* (<Lat>[IPA]</Lat> - tradução).

## Vocabulary

| Portuguese | IPA | English |
|:--|:--|:--|

## Exercises

1. Cinco itens.

::: details Answers
1. Respostas.
:::

## Informal version

| Who | Portuguese | IPA | English |
|:--|:--|:--|:--|

**Notas.** O que se diz de verdade na rua, gíria, armadilhas.
```

### Regras de conteúdo e de IPA

- **Colchetes e acento**: transcrição entre colchetes, com `ˈ` antes da sílaba tônica.
- **Pausas**: `|` para pausa curta e `‖` para fim de frase; a pontuação original não entra na transcrição.
- **Pausa dentro de tabela**: escrever **`&#124;`**. Um `|` solto, ou mesmo `\|`, é lido como divisória de coluna e empurra a transcrição para a coluna seguinte.
- **Letra g**: no IPA usar `ɡ` (U+0261), não o `g` comum.
- **Aspas**: nunca usar « ».

---

## 8. Publicação (`.github/workflows/deploy.yml`)

No repositório, em Settings → Pages → Source, escolher **GitHub Actions**. Depois, cada push no `main` publica sozinho.

```yaml
name: Deploy site to GitHub Pages
on:
  push:
    branches: [main]
  workflow_dispatch:
permissions:
  contents: read
  pages: write
  id-token: write
concurrency:
  group: pages
  cancel-in-progress: false
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 22
          cache: npm
      - run: npm ci
      - run: npm run build
      - uses: actions/configure-pages@v5
      - uses: actions/upload-pages-artifact@v3
        with:
          path: docs/.vitepress/dist
  deploy:
    needs: build
    runs-on: ubuntu-latest
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - id: deployment
        uses: actions/deploy-pages@v4
```

O `package-lock.json` precisa estar no repositório, porque o `npm ci` depende dele.

---

## 9. Testes usados

Os testes usam **puppeteer-core** com o Edge instalado (`C:/Program Files (x86)/Microsoft/Edge/Application/msedge.exe`), em 1366px e 390px de largura. Para cada página, verificam:

- ausência de rolagem horizontal (`scrollWidth > innerWidth`);
- todos os trechos `.lat` cobertos ao abrir; um clique revela um, e "Show all" revela todos;
- o painel "Aa" troca a fonte do `body`;
- o tocador existe na página da lição e aponta para o ID de áudio certo;
- em toda tabela, cada linha tem o mesmo número de células do cabeçalho, cada célula da coluna IPA começa com `[` e termina com `]`, e a coluna English não contém símbolos IPA.

---

## 10. Para começar um projeto novo

1. Copiar `package.json`, `.gitignore`, `.github/workflows/deploy.yml` e a pasta `docs/.vitepress/` inteira.
2. Rodar `npm install` e incluir o `package-lock.json` gerado no repositório.
3. Em `config.mts`, trocar `base`, `title`, `description`, `lang`, os links do GitHub e a lista de páginas.
4. Em `audios.ts`, trocar os IDs do Drive. Se não houver áudio, basta não usar `unidade` no frontmatter.
5. Em `estilo.css`, trocar as cores `brand` e as listras, se quiser outra identidade.
6. Em `Capa.vue`, trocar a imagem e a legenda.
7. Se a coluna coberta tiver outro nome, ajustar `colunasCobertas` em `latinizado-markdown.ts` e os rótulos em `Lat.vue` e `LatTodos.vue`.
8. No GitHub, apontar o Pages para **GitHub Actions** e fazer o primeiro push.
