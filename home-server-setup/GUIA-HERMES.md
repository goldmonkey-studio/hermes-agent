# Guia: Hermes em todo lugar (celular, carro e bar)

Com o servidor do Mac já no ar (Nextcloud + Vaultwarden + Jellyfin), o **Hermes
Agent** vira o "cérebro" que você comanda por **texto ou voz** de qualquer lugar
— principalmente pelo **Telegram**, que funciona no iPhone, Android, e até no
**carro** (mensagem de voz via CarPlay/Android Auto) e no bar.

```
  Você (voz/texto no Telegram)  ─►  Bot do Telegram  ─►  Hermes no Mac
                                                            │
                         pode mexer em arquivos, mídia, rodar tarefas, cron…
```

---

## Passo 1 — Instalar o Hermes no Mac

No Terminal do Mac (Apple Silicon M2):

```bash
curl -fsSL https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.sh | bash
source ~/.zshrc      # recarrega o terminal
```

Confirme:

```bash
hermes --version
```

## Passo 2 — Escolher o modelo de IA

O Hermes funciona com vários provedores (Nous Portal, OpenRouter, OpenAI, etc.).
Configure um:

```bash
hermes model
```

Siga o assistente e cole a chave de API do provedor que você escolher.

## Passo 3 — Criar o bot do Telegram

1. No Telegram, fale com **@BotFather** → `/newbot` → escolha nome e usuário.
2. Ele te dá um **token** (algo como `123456:ABC-DEF...`). Guarde.
3. Descubra o seu **user ID**: fale com **@userinfobot** → ele responde seu ID
   numérico.

## Passo 4 — Configurar o gateway

```bash
hermes gateway setup
```

Quando pedir o Telegram, informe:
- **Bot token** → o token do @BotFather
- **Allowed users** → o seu user ID (só você pode comandar o bot)
- **Home channel** → opcional, o chat padrão para relatórios/cron

> Em vez do assistente, você também pode editar o `.env` do Hermes:
> ```
> TELEGRAM_BOT_TOKEN=123456:ABC-DEF...
> TELEGRAM_ALLOWED_USERS=SEU_USER_ID
> TELEGRAM_HOME_CHANNEL=SEU_USER_ID
> GATEWAY_ALLOW_ALL_USERS=false        # mantém só você autorizado
> ```

## Passo 5 — Ligar e testar

Em primeiro plano (para testar):

```bash
hermes gateway start
```

Agora mande uma mensagem para o seu bot no Telegram. Ele deve responder. 🎉
Teste uma mensagem de **voz** também — o Hermes transcreve e responde.

## Passo 6 — Deixar rodando 24/7 (serviço launchd no Mac)

Para o bot continuar no ar mesmo depois de fechar o terminal:

```bash
hermes gateway install     # instala como serviço do macOS (launchd)
```

Comandos úteis depois:

```bash
hermes gateway status
hermes gateway restart
hermes gateway stop
```

> Atenção: o serviço só responde enquanto o **Mac estiver ligado e acordado**.
> Para ficar 100% disponível mesmo com o Mac desligado, dá para rodar o Hermes
> num VPS barato — me avise que eu te passo esse caminho.

---

## Como isso liga no seu "tudo em todo lugar"

- **Arquivos (Nextcloud):** peça ao Hermes para ler/escrever na sua pasta de
  trabalho — os mesmos arquivos que aparecem no celular e no iPad.
- **Mídia (Jellyfin):** automações tipo "monte uma playlist da pasta X" ou avisos.
- **Senhas (Vaultwarden):** mantenha o cofre separado; o Hermes não precisa dele
  para funcionar.
- **Carro:** abra o Telegram pelo CarPlay/Android Auto e mande **áudio** — é a
  forma segura de comandar dirigindo.
- **Bar:** o mesmo bot do Telegram funciona em qualquer aparelho logado na sua
  conta; relatórios podem cair no canal que você definir.

---

## Automações (opcional, mas poderoso)

O Hermes tem **cron** em linguagem natural. Exemplos que você pode pedir:

- "Todo dia às 8h, me mande um resumo da minha agenda."
- "Toda sexta, faça backup da pasta de projetos e me avise."

É só pedir no chat — ele agenda e entrega no Telegram.

---

### Resumo dos comandos

| Ação | Comando |
|---|---|
| Instalar | `curl ... install.sh \| bash` |
| Escolher modelo | `hermes model` |
| Configurar gateway | `hermes gateway setup` |
| Ligar (teste) | `hermes gateway start` |
| Rodar 24/7 | `hermes gateway install` |
| Status / reiniciar | `hermes gateway status` / `restart` |
