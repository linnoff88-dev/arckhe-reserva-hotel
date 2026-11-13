# reserva-hotel

**ðŸ‡§ðŸ‡· [PortuguÃªs](README.md) Â· ðŸ‡ºðŸ‡¸ [English](README.en.md) Â· ðŸ‡ªðŸ‡¸ [EspaÃ±ol](README.es.md)**

[![Reserva Hotel](guia/assets/banner.jpg)](https://inematds.github.io/reserva-hotel/guia/)

## ðŸ“– Guia de uso

Guia completo (landing + passo a passo): **https://inematds.github.io/reserva-hotel/guia/**

ðŸŽ¬ VÃ­deo: https://www.youtube.com/watch?v=HqaWLOahAag

Sistema de reservas para **uma pousada ou hotel pequeno**, em Python 3 com biblioteca padrÃ£o e SQLite: inventÃ¡rio por noite (tipos de quarto e quartos numerados), tarifas por temporada com pacote de baixa e desconto de reserva direta, cotaÃ§Ã£o/reserva/cancelamento pela conversa (chat web e WhatsApp pela Evolution API), lembrete com prÃ©-check-in, check-in/check-out/no-show pela equipe, lista de espera, mensagem pÃ³s-estadia com consentimento, calendÃ¡rios iCal para bloquear datas na Booking/Airbnb e importar o calendÃ¡rio delas por arquivo, origem de cada reserva, equipe pelo Telegram, cadastros, backup, LGPD e exportaÃ§Ã£o mensal para o Painel de RecuperaÃ§Ã£o do **Raio-X de Margem** (pacote `hotel`).

**Estado (19/10/2026): sistema implementado, v0.9.0.** `python3 -m pytest -q tests/` â†’ **98 passed**; contrato recongelado (`a2729b84a50387a9`) depois de corrigir o teste de reimportaÃ§Ã£o iCal que contradizia a regra de UID da especificaÃ§Ã£o; verificaÃ§Ã£o independente **INDEPENDENTE OK** (pousada nunca vista + `docker build` e healthcheck reais). Servidor local, SQLite, autenticaÃ§Ã£o, inventÃ¡rio, tarifas, reservas, conversa, lembretes, prÃ©-check-in, lista de espera, pÃ³s-estadia, cadastros, LGPD, Raio-X HTTP/CLI, backup/restauraÃ§Ã£o, iCal por arquivo, Evolution, Telegram, chat web, pÃ¡gina da equipe e Docker implementados. Depende do dono: credenciais reais da Evolution/WhatsApp e do Telegram, deploy na VPS, dados reais da pousada e operaÃ§Ã£o humana de `/` e `/equipe`. HistÃ³rico das falhas em `FALHAS.md`.

## O que o sistema faz (v1)

- **InventÃ¡rio por noite:** check-in inclusivo, check-out exclusivo; unidades por tipo (ou nÂº de quartos cadastrados); estadia mÃ­nima por perÃ­odo; fechamentos; bloqueio de quarto; **sem overbooking** (pedidos concorrentes para a Ãºltima unidade: um 201, os outros 409).
- **Tarifas:** tarifa base por tipo, temporadas (alta/baixa/feriado) com preÃ§o por noite, pessoa extra, pacote de baixa (ex.: 3 noites pagam 2), desconto de canal prÃ³prio (direta/balcÃ£o), sinal como polÃ­tica escrita; centavos com arredondamento meio para cima.
- **Conversa** (chat web e WhatsApp): consultar disponibilidade e preÃ§o, reservar, ver e cancelar a prÃ³pria reserva conforme a polÃ­tica, FAQ, atendente humano pelo Telegram, "PARAR".
- **Ciclo da estadia:** lembrete com prÃ©-check-in, check-in/check-out/no-show pela equipe, pÃ³s-estadia convidando a reservar direto (sÃ³ com consentimento), lista de espera.
- **OTAs:** exportaÃ§Ã£o iCal por tipo e por quarto; importaÃ§Ã£o do `.ics` baixado da OTA (vira reserva de origem `booking`/`airbnb`); nenhuma busca de URL externa.
- **Equipe:** pÃ¡gina `/equipe` com token; Telegram com `/chegadas`, `/ocupacao`, `/fila`, `/responder`, `/encerrar` e avisos de reserva nova/cancelada.
- **Raio-X:** `GET /api/raiox?mes=` e CLI `raiox --acompanhamento` com os ids do pacote `hotel`; backup/restauraÃ§Ã£o; Docker.

O que fica de fora do v1 estÃ¡ na seÃ§Ã£o 14 da especificaÃ§Ã£o.

## Como foi construÃ­do (mÃ©todo, para refazer ou adaptar)

O sistema foi construÃ­do assim: contrato e testes congelados, execuÃ§Ã£o longa por um agente pela assinatura e verificaÃ§Ã£o independente. Para sÃ³ rodar, basta Python 3.10+ (veja "Executar" e "Deploy na VPS"); os passos abaixo servem para refazer ou adaptar o sistema com outro contrato.

### PrÃ©-requisitos

- Linux/macOS com `git`, `python3` â‰¥ 3.10 e `pytest` (`pip install pytest`; Ã© ferramenta de desenvolvimento, a aplicaÃ§Ã£o nÃ£o depende dela).
- Um agente pela **assinatura**: Codex CLI (`codex`) ou Claude Code (`claude`). Nenhuma API Ã© chamada.
- Para a verificaÃ§Ã£o independente final: `docker`.
- MÃ©todo e ferramentas de execuÃ§Ã£o longa: clone de `~/projetos/execucao-longa` (`tools/loop-longrun.sh`, `docs/PLANO-EXECUCAO-LONGA.md`). Se nÃ£o tiver, a rota `/goal` interativa (abaixo) nÃ£o precisa dele.

### 1. Responder as decisÃµes e congelar o contrato

```sh
cd reserva-hotel
# leia docs/DECISOES-ABERTAS.md; "ok em tudo" = nada a mudar
python3 -m pytest -q --collect-only | tail -n 1     # 98 tests collected
bash longrun/2026-10-05-reservas-v1/congelar.sh     # grava hash-congelado.txt e faz 1 commit
```

Se mudar algo marcado âš  nas decisÃµes, edite `docs/ESPECIFICACAO.md` e os testes **antes** de congelar. Depois de congelado, contrato e testes sÃ£o protegidos: o agente nÃ£o os altera; se um teste contradizer a spec, ele registra em `failures.md` e para (portÃ£o humano).

### 2a. Loop headless (Codex `gpt-6-astra`)

```sh
~/projetos/execucao-longa/tools/loop-longrun.sh longrun/2026-10-05-reservas-v1
```

ConfiguraÃ§Ã£o em `longrun/2026-10-05-reservas-v1/loop.env` (teste rÃ¡pido/final, arquivos protegidos, 20 ciclos Ã— 30 min, 8 GB, parada apÃ³s 3 ciclos sem avanÃ§o, `CODEX_ARGS` liberando socket local no sandbox). O loop faz `flock`, `timeout`, `MemoryMax`, reverte ciclo que mexer em arquivo protegido, commita checkpoint por ciclo e para sozinho quando o teste final passa.

### 2b. `/goal` no Codex TUI

```sh
codex -c sandbox_workspace_write.network_access=true
```

Cole o conteÃºdo de `longrun/2026-10-05-reservas-v1/prompt-goal-codex.md`. Acompanhe com `/goals`; pause/retome com `/goal pause|resume`.

### 2c. `/goal` no Claude Code

Abra uma sessÃ£o nova na pasta do projeto e siga `longrun/2026-10-05-reservas-v1/prompt-goal-claude.md` (a condiÃ§Ã£o do `/goal` + o mesmo bloco do Codex como primeira mensagem).

### 3. Acompanhar

```sh
tail -n 20 longrun/2026-10-05-reservas-v1/loop.log        # headless
cat longrun/2026-10-05-reservas-v1/state.md               # Funciona / Falta / Como retomar
tail -n 5 longrun/2026-10-05-reservas-v1/progress.md
python3 -m pytest -q tests/ | tail -n 3
```

Pronto = `python3 -m pytest -q tests/` â†’ **98 passed** e `bash longrun/2026-10-05-reservas-v1/verificar-limites.sh` â†’ **LIMITES OK** (hash do contrato intacto, nÂº de testes igual, sÃ³ arquivos do escopo, sem dependÃªncia externa, sem URL externa, sem chave, sem `0.0.0.0` no cÃ³digo).

### 4. VerificaÃ§Ã£o independente (nÃ­vel 4, humano)

```sh
python3 longrun/2026-10-05-reservas-v1/verificar-independente.py   # INDEPENDENTE OK
```

Procura valores do fixture copiados no cÃ³digo, sobe uma pousada nunca vista (outros tipos, preÃ§os, temporada, polÃ­ticas, relÃ³gio) e confere regras que sÃ³ passam se a lÃ³gica for geral; depois faz `docker build` e testa o healthcheck do container. Abra `/` e `/equipe` no navegador.

## Executar

```sh
mkdir -p dados && cp exemplos/pousada.json dados/pousada.json   # troque TROQUE-ESTE-TOKEN e TROQUE-ESTE-SEGREDO
./reserva serve --porta 8080 --dados dados
```

O chat do hÃ³spede fica em `/`, a pÃ¡gina da equipe em `/equipe` (header `X-Token`). `RESERVA_AGORA` fixa o relÃ³gio sÃ³ para testes. Compose, proxy HTTPS, webhooks da Evolution e do Telegram, calendÃ¡rios iCal nas OTAs e backup diÃ¡rio estÃ£o em "Deploy na VPS", abaixo.

## Arquivos

- `docs/ESPECIFICACAO.md` â€” contrato (20 seÃ§Ãµes, tabela de rotas, regras com fronteiras, fora do v1).
- `docs/DECISOES-ABERTAS.md` â€” decisÃµes com proposta padrÃ£o; perguntas para o dono.
- `docs/MAPA-RAIO-X.md` â€” cada vazamento do pacote `hotel` â†’ peÃ§a do v1 ou motivo de ficar fora.
- `tests/` â€” 98 testes caixa-preta (`conftest.py` sobe o CLI, relÃ³gio fixo, servidor falso de Evolution/Telegram). Fixture em `tests/fixtures/pousada.json`; cÃ³pia em `exemplos/`.
- `longrun/2026-10-05-reservas-v1/` â€” goal (nÃ­vel 3+), plan, state, loop.env, prompts, `congelar.sh`, `verificar-limites.sh`, `verificar-independente.py`.
- `FALHAS.md` â€” uma linha por falha (data, o que quebrou, menor correÃ§Ã£o, prompt | infra).

## Deploy na VPS

Dockerfile, compose e exemplo de ambiente disponÃ­veis; o `docker build` e o healthcheck jÃ¡ passaram na verificaÃ§Ã£o independente. O deploy na VPS, com as suas credenciais, Ã© do operador.

1. Copie `.env.exemplo` para `.env`, configure os segredos e a pousada no volume de dados. Suba com `docker compose up -d --build`. O serviÃ§o deve publicar somente a porta local `127.0.0.1:8080`, com volume persistente em `/dados`.
2. Configure um proxy reverso com HTTPS no domÃ­nio da pousada, encaminhando para essa porta local. Confira `/api/saude` pelo proxy.
3. Para WhatsApp, preencha `EVOLUTION_URL`, `EVOLUTION_API_KEY`, `EVOLUTION_INSTANCIA` e `WEBHOOK_SEGREDO`. Configure na Evolution o evento `MESSAGES_UPSERT`, com destino no domÃ­nio HTTPS seguido de `/webhook/evolution/<segredo>`.
4. Para a equipe, preencha `TELEGRAM_API_URL`, `TELEGRAM_TOKEN`, `TELEGRAM_CHAT_ID` e `TELEGRAM_SEGREDO`. Registre pelo mÃ©todo `setWebhook` o endereÃ§o HTTPS terminado em `/webhook/telegram`, com `secret_token` igual a `TELEGRAM_SEGREDO`. IntegraÃ§Ãµes vazias ficam desativadas.
5. Cadastre na Booking/Airbnb o endereÃ§o do domÃ­nio seguido de `/ical/<segredo>/tipo/<tipo>.ics` ou `/ical/<segredo>/quarto/<quarto>.ics`. Para importar, baixe o arquivo `.ics` da OTA e envie pela pÃ¡gina da equipe. O sistema nÃ£o busca calendÃ¡rios por URL.
6. Agende backup diÃ¡rio no cron com `./reserva backup --dados DIR --saida ARQ` (dentro do container: `docker compose exec -T reserva ./reserva backup --dados /dados --saida /dados/backup-diario.db`). Guarde cÃ³pias fora da VPS e valide uma restauraÃ§Ã£o com o servidor parado usando `./reserva restaurar --dados DIR --entrada ARQ` antes de operar com dados reais.

---

Projeto adaptado da comunidade INEMA TDS - Marca Arckhe