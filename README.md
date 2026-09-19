# Manual da Casa Orlando

Página única, estática, com modo apresentação sincronizado. A TV só abre um endereço:
nada instalado, nenhum login.

---

## Parte 1 — Publicar no GitHub Pages (5 minutos)

Não precisa de terminal. Dá pra fazer tudo pelo site do GitHub.

1. Crie um repositório novo em <https://github.com/new>.
   - Nome: `orlando` (ou o que quiser — ele entra na URL final)
   - Marque **Public**
   - Crie
2. Na página do repositório, clique em **Add file → Upload files** e suba o
   `index.html` e o `README.md`. Confirme em **Commit changes**.
3. Vá em **Settings → Pages**.
   - Em *Source*, escolha **Deploy from a branch**
   - Branch: `main`, pasta `/ (root)` → **Save**
4. Espere ~1 minuto e recarregue. O GitHub mostra a URL:

   ```
   https://SEU-USUARIO.github.io/orlando/
   ```

Pronto — esse é o endereço que vai na TV e no grupo da família.

> **Dica pra TV:** digitar URL com controle remoto é sofrimento. Gere um link curto
> (bit.ly, encurtador da sua preferência) ou deixe a página já aberta na TV antes da reunião.

Se preferir terminal:

```bash
git init
git add .
git commit -m "Manual da Casa Orlando"
git branch -M main
git remote add origin https://github.com/SEU-USUARIO/orlando.git
git push -u origin main
```

E depois ative o Pages no passo 3 acima.

---

## Parte 2 — Ligar a sincronização dos slides

Sem isso a apresentação funciona normalmente, só não sincroniza entre aparelhos.
Com isso, quando você passa um card no celular, **todo mundo que estiver em modo
apresentação pula junto** — inclusive a TV.

O canal é um Firebase Realtime Database: gratuito, roda na porta 443 (não cai em
firewall de Wi-Fi de Airbnb) e não exige instalar nada em lugar nenhum.

### Criar o banco

1. Entre em <https://console.firebase.google.com> com sua conta Google.
2. **Adicionar projeto** → nome `orlando` → pode desativar o Google Analytics → criar.
3. No menu lateral: **Criar → Realtime Database** → **Criar banco de dados**.
   - Local: qualquer um (`us-central1` serve)
   - Escolha **Iniciar no modo de teste** → Ativar
4. Copie a URL que aparece no topo, algo como:

   ```
   https://orlando-xxxxx-default-rtdb.firebaseio.com
   ```

### Colar no arquivo

Abra o `index.html` e, logo no começo do `<script>`, troque a linha:

```js
var SYNC_URL = "";
```

por (usando **a sua** URL, e mantendo o final `/orlando/c7e06f64b6.json`):

```js
var SYNC_URL = "https://orlando-xxxxx-default-rtdb.firebaseio.com/orlando/c7e06f64b6.json";
```

Suba o arquivo alterado pro GitHub de novo. Feito.

> O `c7e06f64b6` é um caminho aleatório só pra ninguém esbarrar no seu canal por acaso.
> Pode trocar por outra coisa, desde que seja igual em todos os aparelhos (é o mesmo arquivo,
> então já é).

### Sobre as regras do Firebase

O "modo de teste" deixa o banco aberto pra leitura e escrita e **expira em 30 dias**.
Pra uma viagem isso resolve. Se quiser deixar aberto só o caminho da apresentação,
vá em **Realtime Database → Regras** e use:

```json
{
  "rules": {
    "orlando": {
      "c7e06f64b6": { ".read": true, ".write": true }
    }
  }
}
```

O único dado que trafega é `{slide, ts, by}` — o número do card. Nada pessoal.

---

## Parte 3 — Como usar na reunião

**Na TV**

1. Abra o endereço no navegador da TV
2. **Apresentar**
3. Deixe em **Seguindo** (já vem selecionado)

**No seu celular**

1. Abra o mesmo endereço
2. **Apresentar**
3. Toque em **Eu controlo**
4. Arraste o dedo pro lado (ou use os botões)

A TV — e o celular de qualquer parente que também esteja em modo apresentação —
acompanha em até ~1,5 segundo.

O terceiro botão, **Só aqui**, desliga a sincronização daquele aparelho: útil pra
alguém folhear no próprio ritmo sem bagunçar a tela dos outros.

### Sem sincronização

Se a internet cair ou você pular a Parte 2, a apresentação continua funcionando em
cada aparelho: setas do teclado, clique nas laterais da tela ou swipe.

---

## Estrutura

Arquivo único, sem build, sem dependências. As únicas coisas externas são as fontes
do Google Fonts (se não carregarem, o fallback do sistema assume) e, se você ligar a
Parte 2, as chamadas ao Firebase.

Os slides são **gerados a partir do próprio conteúdo da página** — cada regra, cada
card de app, cada bloco do glossário vira um slide automaticamente. Ou seja: pra
mudar a apresentação, é só editar o HTML da página. Não existe uma segunda cópia do
texto pra manter sincronizada.

### Onde mexer

| O que | Onde |
|---|---|
| Regras, textos, seções | direto no HTML, dentro de `<main>` |
| Itens da mochila | `<ul class="check-grid">` |
| Glossário | `<dl class="gloss">` |
| Cores (claro e escuro) | bloco `:root` no `<style>` |
| Velocidade da sincronização | `POLL_MS` no `<script>` |
