# Galeria — NURBAN PRAÇA DA ÁRVORE

Galeria de imagens e vídeo para injeção via script no 3DVista, hospedada no AWS S3.

## Arquivos

| Arquivo | Descrição |
|---------|-----------|
| `index.html` | Galeria completa (auto-suficiente, vai dentro do iframe) |
| `inject.js`  | Script leve que cria o overlay no 3DVista |
| `assets/`    | Pasta com todas as imagens organizadas por categoria |

---

## Estrutura de pastas (`assets/`)

```
assets/
├── Fachada/
├── Áreas Comuns/
├── Living/
├── Plantas/
├── Pavimentos/
└── Video/          (um único arquivo .mp4, sem sub-pastas)
```

---

## Como funciona

### Dois modos de galeria

A galeria aceita um parâmetro `?mode=` na URL:

| Modo | URL | O que exibe |
|------|-----|-------------|
| `imagens` | `index.html?mode=imagens` | Categorias de imagens |
| `plantas` | `index.html?mode=plantas` | Plantas por tipologia |
| `all` | `index.html` | Tudo (uso local / testes) |

### Filtros

- **Modo imagens**: filtros principais = categorias de imagens. Categorias com sub-filtros por tipologia.
- **Modo plantas**: filtros principais = tipologias. Cada filtro mostra as plantas daquela tipologia.
- Sem botão "Todos" — galeria já abre filtrada na primeira categoria disponível.

### Card duplex (cobertura-plan)

Cards de plantas com dois pavimentos usam `type: 'cobertura-plan'` (recurso do motor da galeria, não usado nos itens atuais deste projeto):
- Abas **Pavimento Inferior / Pavimento Superior** para alternar
- Label flutuante sobre a imagem indicando o pavimento
- Botão **"Ver ambos os pavimentos"** abre lightbox lado a lado

### Card de vídeo

Itens com `type: 'video'` mostram a primeira cena do arquivo como preview (via `<video preload="metadata">`) com um botão de play sobreposto. Como só existe um vídeo, não há sub-pastas nem múltiplos nomes — basta um item apontando para o `.mp4` dentro de `assets/Video/`. Ao clicar, abre no lightbox com os controles nativos do navegador (sem zoom/pan, que são exclusivos de imagem).

### Lightbox

- Navegação por setas (desktop) ou swipe horizontal (mobile/touch) — apenas entre imagens
- Tecla `Esc` fecha; setas do teclado navegam
- Dual-view no mobile empilha verticalmente
- Zoom com scroll/pinch e pan com drag (imagens) — vídeo usa o player nativo, sem zoom

### Responsividade

| Tela | Colunas na grade |
|------|-----------------|
| > 1100px | 4 colunas |
| 701–1100px | 3 colunas |
| 421–700px | 2 colunas |
| ≤ 420px | 1 coluna |

---

## Configurar as imagens

Abra `index.html` e edite o objeto `GALLERY_CONFIG`.

### Categorias

```js
categories: [
  { id: 'fachada',      label: 'Fachada' },
  { id: 'areas-comuns', label: 'Áreas Comuns' },
  { id: 'living',       label: 'Living' },
  { id: 'plantas',      label: 'Plantas' },
  { id: 'pavimentos',   label: 'Pavimentos' },
  { id: 'video',        label: 'Vídeo' },
],
```

### Itens — imagem simples

```js
{ id:1, type:'image', category:'fachada', title:'Fachada Principal',
  src: 'assets/Fachada/NOME-DO-ARQUIVO.jpg' },
```

Use `isPlant:true` em itens de `plantas`/`pavimentos` para fundo branco + `object-fit:contain`.

### Item — vídeo

```js
{ id:90, type:'video', category:'video', title:'Vídeo',
  src: 'assets/Video/NOME-DO-ARQUIVO.mp4' },
```

### Itens — planta com dois pavimentos (opcional, não usado neste projeto)

```js
{ id:30, type:'cobertura-plan', category:'plantas', subCategory:'pl-tipologia-1',
  title:'Tipologia 1', area:'', floors:[
    { label:'Pavimento Inferior', src:'assets/plantas/tipologia-1/1/PAVIMENTO INFERIOR.jpg' },
    { label:'Pavimento Superior', src:'assets/plantas/tipologia-1/1/PAVIMENTO SUPERIOR.jpg' },
]},
```

---

## Deploy na AWS S3

### URL do bucket

```
s3://skylineip/Tour Virtual/vitaurbana/galeria-praca-da-arvore/
```

### Upload completo (primeira vez)

```bash
aws s3 sync . "s3://skylineip/Tour Virtual/vitaurbana/galeria-praca-da-arvore/" \
  --exclude ".git/*" --exclude "*.md" \
  --content-type "text/html" --include "*.html"

aws s3 sync assets/ "s3://skylineip/Tour Virtual/vitaurbana/galeria-praca-da-arvore/assets/"
```

### Atualizar só os arquivos principais

```bash
aws s3 cp index.html "s3://skylineip/Tour Virtual/vitaurbana/galeria-praca-da-arvore/index.html" \
  --content-type "text/html"

aws s3 cp inject.js "s3://skylineip/Tour Virtual/vitaurbana/galeria-praca-da-arvore/inject.js" \
  --content-type "application/javascript"
```

### URLs resultantes

```
https://skylineip.s3.amazonaws.com/Tour%20Virtual/vitaurbana/galeria-praca-da-arvore/index.html
https://skylineip.s3.amazonaws.com/Tour%20Virtual/vitaurbana/galeria-praca-da-arvore/inject.js
```

---

## Integração 3DVista

### Passo 1 — Carregar o script (Custom HTML no Skin Editor)

```html
<script>
(() => {
  const scriptUrl = 'https://skylineip.s3.amazonaws.com/Tour%20Virtual/vitaurbana/galeria-praca-da-arvore/inject.js';
  if (document.querySelector(`script[src="${scriptUrl}"]`)) return;
  const s = document.createElement('script');
  s.src = scriptUrl;
  document.head.appendChild(s);
})();
</script>
```

### Passo 2 — Acionar a galeria nos hotspots

```js
// Abre galeria de imagens
setTimeout(() => { GaleriaImagens(1); }, 300);

// Fecha galeria de imagens
GaleriaImagens(0);

// Abre galeria de plantas
setTimeout(() => { GaleriaPlantas(1); }, 300);

// Fecha galeria de plantas
GaleriaPlantas(0);
```

> O `setTimeout` garante que o script já foi carregado antes de chamar a função.

---

## Cores e tipografia

| Token | Valor |
|-------|-------|
| Background | `#E4E4E4` |
| Texto / Secundária | `#4C1F24` |
| Acento / Principal | `#F18F58` |
| Fonte títulos | Cormorant Garamond |
| Fonte UI | Inter |
