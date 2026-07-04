# CLAUDE.md — EXPOUSIPA 2026 Gamification App

> **Para o Claude Code:** leia este arquivo inteiro antes de qualquer ação. Ele contém tudo para deixar o ambiente pronto e responder a qualquer pedido de alteração no app.

---

## SETUP INICIAL — execute isso na primeira vez

### 1. Autenticação GitHub

Para dar `git push`, é necessário estar autenticado no GitHub com a conta **adosilvaviana**. O Rafael deve fazer login em [github.com](https://github.com) no navegador com as credenciais desta conta antes de rodar os comandos abaixo. Na primeira vez que o terminal pedir usuário/senha, usar:

- **Usuário:** `adosilvaviana`
- **Senha:** usar um Personal Access Token (PAT) gerado em github.com → Settings → Developer settings → Personal access tokens (a senha normal do GitHub não funciona no terminal)

Se já estiver autenticado no terminal (de uma sessão anterior), pular direto para o clone.

### 2. Clonar o repositório

Se o repositório ainda não estiver clonado localmente, faça agora:

```bash
git clone https://github.com/adosilvaviana/expousipa-game.git
cd expousipa-game
```

Para rodar o servidor local de desenvolvimento:

```bash
python3 -m http.server 8765
```

Acesse em: `http://localhost:8765`

Para navegar direto para uma tela específica: `http://localhost:8765/#result`, `/#cta`, `/#choose`, etc.

Após o setup, confirme para o usuário (Rafael) que o ambiente está pronto e pergunte o que ele quer alterar.

---

## Como publicar qualquer alteração

```bash
git add index.html
git commit -m "descrição da mudança"
git push origin main
```

GitHub Pages publica automaticamente em ~1 minuto em:
**`https://adosilvaviana.github.io/expousipa-game/`**

Após o push, avisar Rafael para fazer `Ctrl+Shift+R` (ou `Cmd+Shift+R` no Mac) no navegador.

Se alterar também `admin.html`, incluir no `git add`.

---

## O que é este projeto

App de gamificação offline para o totem touchscreen de 43" (Android 11, portrait) da EXPOUSIPA 2026 — evento do Sistema Fecomércio MG em Ipatinga, 8 a 10 de julho de 2026, no complexo USIPA.

O app captura leads e oferece três mini-jogos temáticos (Sesc, Senac, Sindicato do Comércio). Funciona 100% offline; os dados são salvos em `localStorage` e sincronizados com Google Sheets via webhook quando há conexão.

---

## Credenciais e acessos

| O quê | Valor |
|---|---|
| Repositório GitHub | `https://github.com/adosilvaviana/expousipa-game` |
| GitHub Pages (URL pública) | `https://adosilvaviana.github.io/expousipa-game/` |
| Senha do painel Admin | `sesc2026` |
| Google Sheets Webhook URL | `https://script.google.com/macros/s/AKfycbzcoLNKTCMWjBNu3jiIRU4GP5kk48NYwyM-vxYdEUOahFLUaYBw3rEZ_jZih-54Jj1N1g/exec` |

O painel Admin é acessado pelo botão ⋮ discreto no canto inferior direito de qualquer tela do app.

---

## Stack e arquitetura

- **Dois arquivos principais**: `index.html` (app) e `admin.html` (painel admin) — zero dependências, zero build step.
- **Navegação**: `location.hash` (`#splash`, `#form-1`, `#choose`, `#game-sesc`, etc.).
- **Persistência**: `localStorage` (chave `nxp_leads`). Retry automático a cada 30s para leads offline.
- **Totem**: 1080×1920px. CSS `transform: scale()` adapta para qualquer tela via `scaleToFit()`.

---

## Identidade visual — regras obrigatórias

| Variável CSS | Valor | Uso |
|---|---|---|
| `--expo-blue` | `#004587` | Azul principal (títulos, botões, bordas) |
| `--expo-yellow` | `#fbba00` | Amarelo de destaque (badges, CTA) |
| Fundo cream | `#ece4d0` | Cards de resultado e conteúdo destacado |
| Fundo splash | `#ece4d0` | Tela de abertura |

**IMPORTANTE:** nunca recriar os elementos visuais como SVG ou CSS puro. Usar sempre os PNGs oficiais que estão na pasta do projeto:

| Arquivo | Uso |
|---|---|
| `logo-expousipa.png` | Logo EXPO USIPA (splash) |
| `regua-marcas.png` | Régua de marcas do Sistema Comércio (topo splash + bottom de todas as telas) |
| `deco-retangulos.png` | Decoração de retângulos concêntricos (rodapé da splash) |
| `deco-vertical.png` | Decoração vertical (disponível, uso opcional) |

A régua de marcas é injetada automaticamente no bottom de todas as telas (exceto splash) pelo JS — buscar por `brand-bar` no código.

---

## Estrutura de telas

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
| `#admin` | Painel admin — abre `admin.html` numa nova aba |

Navegação: `goTo('nome-da-tela')`. Reset completo: `resetAll()`.

---

## Principais funções JS

| Função | O que faz |
|---|---|
| `goTo(screen)` | Troca a tela ativa pelo hash |
| `resetAll()` | Limpa o estado e volta para a splash |
| `showResult(path)` | Renderiza resultado para `'sesc'`, `'senac'` ou `'sindicato'` |
| `renderCta(path)` | Renderiza o CTA final conforme o caminho |
| `scaleToFit()` | Escala o app para a tela atual |
| `syncLead(lead)` | Envia lead para Google Sheets (no-cors) |
| `saveLead(data)` | Salva lead no localStorage e tenta sync |
| `openAdmin()` | Abre o painel admin (`admin.html`) numa nova aba |
| `openEstadoPicker()` | Bottom sheet de seleção de estado |
| `openIdadePicker()` | Bottom sheet de seleção de faixa etária |
| `escHtml(str)` | Sanitiza string para inserção no DOM — usar sempre que renderizar input do usuário |

---

## Jogo SESC — pares

Todas as unidades Sesc usam o emoji 🏫 (inclusive Senac Grogotó).

```javascript
const sescPairs = [
  { id:0, tourist:'🏛️', tName:'Museu da Inconfidência',                hotel:'🏫', hName:'Sesc Ouro Preto' },
  { id:1, tourist:'💧', tName:'Fonte dos Amores',                      hotel:'🏫', hName:'Sesc Poços de Caldas' },
  { id:2, tourist:'♨️', tName:'Fonte Dona Beja',                       hotel:'🏫', hName:'Sesc Araxá' },
  { id:3, tourist:'⛪', tName:'Igrejinha da Pampulha',                 hotel:'🏫', hName:'Sesc Venda Nova (BH)' },
  { id:4, tourist:'⛪', tName:'Praça N. Sra. da Glória/Igreja Matriz', hotel:'🏫', hName:'Sesc Contagem' },
  { id:5, tourist:'🏫', tName:'Museu da Loucura',                      hotel:'🏫', hName:'Senac Grogotó (Barbacena)' },
];
```

---

## O que NÃO fazer

- Não recriar os PNGs como SVG ou CSS — usar sempre os arquivos `.png` da pasta.
- Não dividir em múltiplos arquivos HTML/JS/CSS — tudo fica em `index.html` e `admin.html`.
- Não adicionar frameworks ou dependências externas — o app precisa funcionar 100% offline.
- Toda string de usuário renderizada no DOM deve passar por `escHtml()` para evitar XSS.
- Não usar `onclick` inline no HTML para novos elementos — usar funções nomeadas já existentes.
