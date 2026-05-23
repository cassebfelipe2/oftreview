# CLAUDE.md — OFT-Review Tracker · Handoff para Claude Code

## O que é este projeto

Aplicação web single-page (um único `index.html`) para o **Felipe Augusto Casseb dos Santos**, R3 de Oftalmologia na Santa Casa de São Paulo, estudando para a prova de Título do **Conselho Brasileiro de Oftalmologia (CBO) 2026**.

O app rastreia o cronograma de revisões espaçadas do **OFT-Review Extensive Geração 7** — 39 semanas de conteúdo novo com sistema de revisões nos intervalos de 15, 45, 90 e 180 dias. Também registra trilhas de questões (feitas + acertadas) e exporta relatório em PDF para enviar ao mentor.

---

## Arquitetura

```
oft-review/
├── index.html   ← TODO o app (HTML + CSS + JS embutidos, zero build step)
├── README.md
└── CLAUDE.md    ← este arquivo
```

**Stack:** HTML5 puro, sem framework, sem bundler, sem node_modules.
- **Persistência:** `localStorage` (chaves: `oft_week`, `oft_history`)
- **PDF:** jsPDF 2.5.1 + jsPDF-AutoTable 3.8.2 via CDN (cdnjs.cloudflare.com)
- **Fontes:** Google Fonts — Syne (UI) + JetBrains Mono (dados/código)
- **Deploy:** GitHub Pages — basta fazer push do `index.html`

---

## Dados principais (constantes JS no index.html)

### `SCHEDULE` (linhas ~802–842)
```js
const SCHEDULE = {
  1: { date: "09/03/2026", module: "Órbita", topics: [...] },
  // ... semanas 1–39
};
```
Cada entrada = uma semana do extensivo, com data real da aula, nome do módulo e lista de tópicos.

### `REVISIONS` (linhas ~844–883)
```js
const REVISIONS = {
  3:  [{ name: "ÓRBITA 1", d: "15" }],
  7:  [{ name: "ÓRBITA 1", d: "45" }, { name: "CÓRNEA 3", d: "15" }],
  // ... semanas 3–40
};
```
Cada entrada = lista de revisões a fazer naquela semana, com nome canônico e intervalo em dias.
- `d: "15"` = revisão de 15 dias (vermelho)
- `d: "45"` = revisão de 45 dias (âmbar)
- `d: "90"` = revisão de 90 dias (verde)
- `d: "180"` = revisão de 180 dias (azul)

### `REV_SOURCE` (linhas ~885–900)
```js
const REV_SOURCE = {
  "ÓRBITA 1": 1,   // "ÓRBITA 1" veio da semana 1
  "CÓRNEA 1": 3,   // etc.
  // ...
};
```
Mapeia nome canônico da revisão → semana de origem, para buscar os tópicos certos no `SCHEDULE`.

### Estado persistido no localStorage
```js
let currentWeek = parseInt(localStorage.getItem('oft_week') || '8');
let history = JSON.parse(localStorage.getItem('oft_history') || '[]');
```
`history` é um array de objetos:
```js
{ revision: "ÓRBITA 1", week: 7, feitas: 30, acertos: 24, date: "23/05/2026" }
```

---

## Funcionalidades implementadas

| Feature | Status | Onde no código |
|---|---|---|
| Navegação entre semanas (setas + teclado ←→) | ✅ | `changeWeek()` |
| Tab: Revisões programadas da semana | ✅ | `renderRevisoes()` |
| Tab: Conteúdo novo da semana | ✅ | `renderConteudo()` |
| Tab: Histórico de trilhas | ✅ | `renderHistorico()` |
| Tab: Todas as revisões (visão geral) | ✅ | `renderTodas()` |
| Expandir tópicos de cada revisão | ✅ | `toggleTopics()` |
| Modal "Registrar trilha" (feitas + acertos) | ✅ | `openModal()`, `saveTrail()` |
| Preview de % em tempo real no modal | ✅ | `updateModalPreview()` |
| Deletar entrada do histórico | ✅ | `deleteHistory()` |
| Limpar todo o histórico | ✅ | `clearHistory()` |
| Stats cards (semana, revisões, trilhas, acerto médio) | ✅ | `render()` |
| Barra de progresso do extensivo | ✅ | `render()` |
| Toast de confirmação | ✅ | `showToast()` |
| Exportar relatório PDF | ✅ | `exportPDF()` |
| Persistência em localStorage | ✅ | `saveState()` |

---

## Exportação de PDF (`exportPDF`)

Localização: final do `index.html`, após o `</script>` principal, em um segundo bloco `<script>`.

O PDF é gerado 100% no browser via jsPDF. Estrutura:
1. **Header escuro** com nome do app, data de geração e semana atual
2. **Seção 1 — Progresso Geral:** 4 stat boxes + barra de progresso dourada
3. **Seção 2 — Revisões por Semana:** tabela agrupada (semana → revisões → %)
4. **Seção 3 — Desempenho por Módulo:** tabela com % por módulo + gráfico de barras em texto
5. **Seção 4 — Histórico Completo:** todas as trilhas individualmente
6. **Footer** em todas as páginas com data e paginação

Cores do PDF:
- `#0f0f11` fundo escuro, `#c9a84c` dourado, `#e8c97a` dourado claro
- Verde `[74,156,110]` ≥80%, Âmbar `[212,147,58]` ≥60%, Vermelho `[224,82,82]` <60%

Filename gerado: `OFT-Review_Relatorio_S08_23-05-2026.pdf`

---

## Design System (CSS vars)

```css
--bg:       #0a0a0b   /* fundo principal */
--bg2:      #111113   /* cards */
--bg3:      #18181c   /* inputs, items secundários */
--border:   rgba(255,255,255,0.07)
--border2:  rgba(255,255,255,0.13)
--text:     #f0ede8   /* texto principal */
--text2:    #888580   /* texto secundário */
--text3:    #555250   /* texto terciário / labels */
--gold:     #c9a84c   /* accent principal */
--gold2:    #e8c97a   /* accent claro */
--gold-dim: rgba(201,168,76,0.12)
--red:      #e05252   /* intervalo 15d */
--amber:    #d4933a   /* intervalo 45d */
--green:    #4a9c6e   /* intervalo 90d */
--blue:     #4a82c4   /* intervalo 180d */
--radius:    10px
--radius-lg: 16px
```

Fontes: `'Syne'` para UI/títulos, `'JetBrains Mono'` para dados numéricos e labels técnicas.

---

## O que falta / próximos passos sugeridos

Estas são melhorias que o Felipe pode querer adicionar — Claude Code pode implementar qualquer uma:

### Alta prioridade
- [ ] **Campo "nome do residente"** nas configurações → aparece no PDF como "Residente: Felipe Casseb"
- [ ] **Input de nome do mentor** → aparece no rodapé do PDF como "Destinatário: Prof. Dr. ..."
- [ ] **Marcar revisão como concluída** (checkbox por revisão, persiste no localStorage) — diferente de registrar trilha
- [ ] **Notificação / badge** na tab quando há revisões pendentes na semana atual

### Média prioridade
- [ ] **Gráfico de evolução** do % de acerto ao longo das semanas (linha temporal)
- [ ] **Filtro por módulo** no histórico
- [ ] **Modo PWA** (manifest.json + service worker) para instalar no iPhone como app
- [ ] **Campo de notas livres** por revisão ("lembrar de revisar síndromes pupilares")

### Baixa prioridade / nice to have
- [ ] **Exportar/importar histórico** como JSON (backup entre dispositivos)
- [ ] **Semana automática** baseada na data real (mapear semana 1 = 09/03/2026 e calcular a atual)
- [ ] **Modo claro** (toggle)
- [ ] **Animação de streak** quando acerto ≥ 80% consecutivo

---

## Como rodar localmente

```bash
# Sem servidor (abre direto no browser):
open index.html

# Com servidor local (opcional, para testar PWA):
python3 -m http.server 8080
# → acessar http://localhost:8080
```

## Como fazer deploy no GitHub Pages

```bash
git init
git add index.html README.md CLAUDE.md
git commit -m "feat: OFT-Review tracker inicial"
git remote add origin https://github.com/SEU_USUARIO/oft-review.git
git push -u origin main
# Depois: Settings → Pages → Branch: main → / (root) → Save
# URL: https://SEU_USUARIO.github.io/oft-review
```

---

## Contexto do usuário (para personalização)

- **Nome:** Felipe Augusto Casseb dos Santos
- **Instituição:** Irmandade da Santa Casa de Misericórdia de São Paulo (ISCMSP)
- **Residência:** R3 Oftalmologia — concluindo em 2026
- **Objetivo:** Aprovação na prova de Título CBO 2026
- **Curso:** OFT-Review Extensive Geração 7 (39 semanas, março–novembro 2026)
- **Semana atual no momento da criação:** Semana 8 (27/04/2026) — Módulo Cristalino
- **Interesse de subespecialidade:** Segmento anterior (catarata premium, córnea, refrativa)
