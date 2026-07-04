# CLAUDE.md — EXPOUSIPA 2026 Gamification App

Instruções para o Claude Code trabalhar neste projeto.

---

## O que é este projeto

App de gamificação offline para o totem touchscreen de 43" (Android 11, portrait) da EXPOUSIPA 2026 — evento do Sistema Fecomércio MG em Ipatinga, 8 a 10 de julho de 2026, no complexo USIPA.

O app captura leads e oferece três mini-jogos temáticos (Sesc, Senac, Sindicato do Comércio). Funciona 100% offline; os dados são salvos em `localStorage` e sincronizados com Google Sheets via webhook quando há conexão.

---

## Stack e arquitetura

- **Um único arquivo**: `index.html` — zero dependências, zero build step.
- **Navegação**: `location.hash` (`#splash`, `#form-1`, `#choose`, `#game-sesc`, etc.).
- **Persistência**: `localStorage` (chave `nxp_leads`). Retry automático a cada 30s para leads offline.
- **Totem**: 1080×1920px. CSS `transform: scale()` adapta para qualquer tela via `scaleToFit()`.
- **Deploy**: GitHub Pages — `git push origin main` publica em ~1 min em `https://adosilvaviana.github.io/expousipa-game/`.

---

## Como rodar localmente

```bash
cd expousipa-game
python3 -m http.server 8765
# Abrir: http://localhost:8765
```

Para ver uma tela específica direto: `http://localhost:8765/#result`, `/#cta`, `/#choose`, etc.

---

## Como publicar alterações

```bash
git add index.html
git commit -m "descrição da mudança"
git push origin main
```

GitHub Pages atualiza em ~1 minuto. Hard refresh no navegador: `Cmd+Shift+R`.

---

## Identidade visual

| Variável CSS | Valor | Uso |
|---|---|---|
| `--expo-blue` | `#004587` | Azul principal (títulos, botões, bordas) |
| `--expo-yellow` | `#fbba00` | Amarelo de destaque (badges, CTA) |
| Fundo cream | `#ece4d0` | Cards de resultado e conteúdo destacado |
| Fundo splash | `#ece4d0` | Tela de abertura |

**Regra**: nunca recriar os elementos visuais como SVG ou CSS. Sempre usar os PNGs oficiais:

| Arquivo | Uso |
|---|---|
| `logo-expousipa.png` | Logo EXPO USIPA (splash) |
| `regua-marcas.png` | Régua de marcas do Sistema Comércio (topo splash + bottom de todas as telas) |
| `deco-retangulos.png` | Decoração de retângulos concêntricos (rodapé da splash) |
| `deco-vertical.png` | Decoração vertical (disponível, uso opcional) |

A régua de marcas é injetada automaticamente no bottom de todas as telas (exceto splash) pelo JS — buscar por `brand-bar` no código.

---

## Estrutura de telas (hash → screen id)

| Hash | Tela |
|---|---|
| `#splash` | Abertura — toque para começar |
| `#form-1` a `#form-4` | Formulário de lead (nome, CPF, telefone, email, cidade, estado, faixa etária) |
| `#choose` | Escolha do jogo (Sesc / Senac / Sindicato) |
| `#game-sesc` | Jogo de pares Sesc (associar destino turístico → unidade Sesc) |
| `#game-senac` | Quiz Senac (múltipla escolha) |
| `#game-sindicato` | Quiz Sindicato / Fecomércio MG |
| `#result` | Resultado do jogo |
| `#cta` | Call to action final (Credencial Sesc / Senac / Fecomércio) |
| `#admin` | Painel admin (lista de leads, exportação CSV) |

Navegação: `goTo('nome-da-tela')`. Reset completo: `resetAll()`.

---

## Principais funções JS

| Função | O que faz |
|---|---|
| `goTo(screen)` | Troca a tela ativa pelo hash |
| `resetAll()` | Limpa o estado e volta para a splash |
| `showResult(path)` | Renderiza a tela de resultado para `'sesc'`, `'senac'` ou `'sindicato'` |
| `renderCta(path)` | Renderiza o CTA final conforme o caminho |
| `scaleToFit()` | Escala o app para a tela atual |
| `syncLead(lead)` | Envia lead para Google Sheets (no-cors) |
| `saveLead(data)` | Salva lead no localStorage e tenta sync |
| `openAdmin()` | Abre o painel admin com senha |
| `openEstadoPicker()` | Bottom sheet de seleção de estado |
| `openIdadePicker()` | Bottom sheet de seleção de faixa etária |

---

## Jogo SESC — pares

Associa pontos turísticos de MG a unidades Sesc. Todas as unidades usam o emoji 🏫.

```javascript
const sescPairs = [
  { id:0, tourist:'🏛️', tName:'Museu da Inconfidência',           hotel:'🏫', hName:'Sesc Ouro Preto' },
  { id:1, tourist:'💧', tName:'Fonte dos Amores',                 hotel:'🏫', hName:'Sesc Poços de Caldas' },
  { id:2, tourist:'♨️', tName:'Fonte Dona Beja',                  hotel:'🏫', hName:'Sesc Araxá' },
  { id:3, tourist:'⛪', tName:'Igrejinha da Pampulha',            hotel:'🏫', hName:'Sesc Venda Nova (BH)' },
  { id:4, tourist:'⛪', tName:'Praça N. Sra. da Glória/Igreja Matriz', hotel:'🏫', hName:'Sesc Contagem' },
  { id:5, tourist:'🏫', tName:'Museu da Loucura',                 hotel:'🏫', hName:'Senac Grogotó (Barbacena)' },
];
```

---

## Google Sheets

Webhook configurado no Google Apps Script. O arquivo `google-apps-script.gs` na raiz do projeto contém o código do script.

```javascript
const SHEETS_WEBHOOK_URL = 'https://script.google.com/macros/s/AKfycbzcoLNKTCMWjBNu3jiIRU4GP5kk48NYwyM-vxYdEUOahFLUaYBw3rEZ_jZih-54Jj1N1g/exec';
```

Envio com `mode: 'no-cors'` e `redirect: 'follow'` para funcionar offline-first (sem bloquear a UX).

---

## Painel Admin

Acesso: botão ⋮ discreto no canto inferior direito de qualquer tela → "Área Admin". Senha configurada na variável `ADMIN_PASSWORD` no JS.

Funcionalidades:
- Listagem de todos os leads capturados
- Soft delete (marca como deletado, não apaga do storage)
- Exportação CSV
- Indicador de leads pendentes de sync

---

## O que NÃO fazer

- Não recriar os PNGs como SVG ou CSS — usar sempre os arquivos `.png` da pasta.
- Não dividir em múltiplos arquivos HTML/JS/CSS — tudo fica em `index.html`.
- Não adicionar frameworks ou dependências externas — o app precisa funcionar offline.
- Não usar `onclick` inline no HTML para novos elementos — usar event listeners ou funções nomeadas já existentes.
- Toda string de usuário renderizada no DOM deve passar por `escHtml()` para evitar XSS.
