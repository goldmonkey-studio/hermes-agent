# Guia: transformar o seu MacBook M2 num servidor pessoal

Objetivo: ter **arquivos, notas/agenda, senhas e mídia** disponíveis em **todos**
os seus aparelhos (iPhone, iPad, Android, carro e bar), com o Mac como hub central.

Tudo roda em containers Docker. Você só copia, preenche senhas e liga.

---

## Visão geral

| Serviço | Para quê | Endereço (após subir) |
|---|---|---|
| **Nextcloud** | Arquivos, Notas, Calendário, Contatos | `http://<mac-tailscale>:8080` |
| **Vaultwarden** | Senhas (app **Bitwarden**) | `http://<mac-tailscale>:8222` |
| **Jellyfin** | Música, vídeo e fotos em streaming | `http://<mac-tailscale>:8096` |

`<mac-tailscale>` = o nome ou IP do seu Mac no Tailscale (ex.: `macbook-m2` ou `100.x.x.x`).

---

## Passo 1 — Instalar os pré-requisitos no Mac

1. **Docker Desktop** (Apple Silicon): https://www.docker.com/products/docker-desktop/
   - Abra o Docker Desktop ao menos uma vez e deixe-o rodando.
2. **Tailscale**: https://tailscale.com/download → faça login.
   - Instale o Tailscale **também** no iPhone, iPad e Android, com a **mesma conta**.
   - No Mac, descubra o nome/IP rodando no Terminal:
     ```bash
     tailscale status
     ```

## Passo 2 — Preparar a pasta

1. Copie a pasta `home-server-setup` para algum lugar do Mac (ex.: sua Home).
2. No Terminal, entre nela e crie o arquivo de senhas:
   ```bash
   cd ~/home-server-setup
   cp .env.example .env
   ```
3. Abra o `.env` e preencha **todas** as senhas e os dados do Tailscale.
   Gere senhas fortes com:
   ```bash
   openssl rand -base64 24
   ```

## Passo 3 — Ajustar os caminhos de mídia (Jellyfin)

No `docker-compose.yml`, troque `SEU_USUARIO` pelo seu usuário do Mac nas linhas
de mídia (e ajuste as pastas se quiser):

```yaml
- /Users/SEU_USUARIO/Music:/media/musica:ro
- /Users/SEU_USUARIO/Movies:/media/videos:ro
- /Users/SEU_USUARIO/Pictures:/media/fotos:ro
```

> No Docker Desktop, garanta que essas pastas estão liberadas em
> **Settings → Resources → File sharing**.

## Passo 4 — Ligar tudo

```bash
docker compose up -d
```

Acompanhe a subida (o Nextcloud demora ~1 min na primeira vez):

```bash
docker compose logs -f
```

Pronto. Acesse cada serviço pelo navegador usando os endereços da tabela acima.

---

## Passo 5 — Configurar cada serviço

### Nextcloud (arquivos + notas + agenda + contatos)
1. Entre em `http://<mac-tailscale>:8080` com o usuário/senha admin do `.env`.
2. No menu de apps, ative **Notes**, **Calendar** e **Contacts**.
3. Nos aparelhos:
   - **iPhone/iPad**: app **Nextcloud** (arquivos) + adicione a conta em
     **Ajustes → Calendário/Contatos → Adicionar conta → Outra → CalDAV/CardDAV**,
     servidor = `<mac-tailscale>:8080`.
   - **Android**: app **Nextcloud** + app **DAVx⁵** para sincronizar agenda e contatos.

### Vaultwarden (senhas)
1. Entre em `http://<mac-tailscale>:8222` e **crie sua conta** (Create Account).
2. Instale o app/extensão **Bitwarden** no iPhone, Android e navegadores.
   Em **Settings → Self-hosted**, coloque `http://<mac-tailscale>:8222`.
3. Depois de criar sua conta, edite o `.env` e mude `SIGNUPS_ALLOWED` para
   `"false"` (no compose), e rode `docker compose up -d` de novo para fechar
   novos cadastros.
   > Observação: para a extensão de navegador funcionar fora do `localhost`, o
   > Vaultwarden pede **HTTPS**. O jeito mais fácil é o Tailscale dar um endereço
   > HTTPS grátis: `tailscale serve --bg 8222` (veja "HTTPS fácil" abaixo).

### Jellyfin (mídia → celular, carro e bar)
1. Entre em `http://<mac-tailscale>:8096` e siga o assistente.
2. Adicione bibliotecas apontando para `/media/musica`, `/media/videos`,
   `/media/fotos`.
3. Nos aparelhos: app **Jellyfin** (iPhone/Android).
   - **Carro**: a reprodução de áudio aparece no **CarPlay/Android Auto**.
   - **Bar**: app Jellyfin na **TV** (Apple TV, Android TV, Fire TV) ou
     espelhando via AirPlay/Chromecast.

---

## HTTPS fácil (recomendado para uso fora de casa)

O Tailscale fornece HTTPS gratuito por serviço. Exemplo para o Vaultwarden:

```bash
tailscale serve --bg --https=443 http://localhost:8222
```

Aí o endereço vira `https://<mac-tailscale>.<sua-tailnet>.ts.net`. Repita em
portas diferentes para os outros serviços, se quiser HTTPS neles também.

---

## Manutenção

```bash
docker compose pull && docker compose up -d   # atualizar imagens
docker compose down                           # desligar
docker compose logs -f nextcloud              # ver logs de um serviço
```

**Backup:** tudo importante fica na pasta `./data`. Faça backup dela (Time
Machine já cobre, se a pasta estiver dentro do Mac). Para o Nextcloud, há o app
de backup integrado também.

---

## Próximo passo: o agente Hermes em todo lugar

Com a base no ar, dá para somar o **Hermes Agent** (deste repositório) para
comandar tudo por **texto ou voz** do celular/carro via Telegram. Me chame quando
quiser que eu monte essa parte — ela liga o gateway do Hermes aos seus aparelhos.
