Perfetto Stefano 👌, ecco il **manuale completo** formattato in Markdown, pronto da incollare nel tuo file `blueprint.md`. Ho integrato tutti i passaggi: creazione repo, bot Telegram, secrets, workflow di ping su `/core/wake/`, keepalive, logging via artifact e README.

---

```markdown
# 📘 Manuale di istruzioni – GitHub Ping Keepalive per otello-web.onrender.com/core/wake/

Questo manuale descrive passo passo come configurare una repository GitHub per mantenere attivo il sito [otello-web.onrender.com/core/wake/](https://otello-web.onrender.com/core/wake/) tramite GitHub Actions, con ping programmati, notifiche Telegram e log salvati come artifact.

---

## 1. Creazione della repository GitHub
1. Vai su [github.com](https://github.com) ed effettua il login.
2. Clicca su **New repository**.
3. Nome consigliato:
   ```
   otello-web-pinger
   ```
4. Descrizione:
   ```
   Repository per mantenere attivo otello-web.onrender.com/core/wake/ con ping programmati, notifiche Telegram e log via artifact.
   ```
5. Seleziona **Public** o **Private**.
6. Spunta **Add a README file**.
7. Clicca **Create repository**.

---

## 2. Creazione del bot Telegram
1. In Telegram cerca **BotFather**.
2. Scrivi `/start` e poi `/newbot`.
3. Dai un nome e un username al bot.
4. Copia il **token API** che ti fornisce (es. `123456:ABC-DEF...`).  
   👉 Questo è il tuo `TELEGRAM_TOKEN`.

---

## 3. Recupero del `chat_id`
1. Apri la chat con il tuo bot e scrivigli un messaggio (es. “ciao”).
2. Apri nel browser:
   ```
   https://api.telegram.org/bot<TELEGRAM_TOKEN>/getUpdates
   ```
3. Nell’output JSON cerca:
   ```json
   "chat": {
     "id": 987654321,
     "first_name": "Stefano",
     "type": "private"
   }
   ```
   👉 Copia il numero in `id` (es. `987654321`). Questo è il tuo `TELEGRAM_CHAT_ID`.

---

## 4. Aggiunta dei secrets nella repository
1. Vai nella tua repository su GitHub.
2. Clicca su **Settings → Secrets and variables → Actions**.
3. Clicca su **New repository secret** e aggiungi:
   - `TELEGRAM_TOKEN` → il token del bot.
   - `TELEGRAM_CHAT_ID` → il numero chat.id.

---

## 5. Creazione della cartella workflow
1. Nella root della repo, crea la cartella `.github/workflows/`.
2. Dentro, crea due file: `ping.yml` e `keepalive.yml`.

---

## 6. Workflow di ping con notifiche Telegram e log
Crea `.github/workflows/ping.yml`:

```yaml
name: Ping otello-web.onrender.com/core/wake (artifacts)

on:
  schedule:
    # ogni 15 minuti dalle 07:30 alle 22:15, da lunedì (1) a sabato (6)
    - cron: '30,45 7 * * 1-6'         # 07:30 e 07:45
    - cron: '0,15,30,45 8-21 * * 1-6' # 08:00–21:45 ogni 15 min
    - cron: '0,15 22 * * 1-6'         # 22:00 e 22:15
  workflow_dispatch:                   # avvio manuale per test

jobs:
  ping:
    runs-on: ubuntu-latest
    steps:
      - name: Esegui ping e registra log
        id: ping_otello
        run: |
          TIMESTAMP=$(date '+%Y-%m-%d %H:%M:%S')
          STATUS=$(curl -s -o /dev/null -w "%{http_code}" https://otello-web.onrender.com/core/wake/ || echo "ERR")
          echo "$TIMESTAMP | https://otello-web.onrender.com/core/wake/ | HTTP $STATUS" >> log.txt

      - name: Notifica Telegram se il ping fallisce
        if: failure()
        run: |
          SITE="https://otello-web.onrender.com/core/wake/"
          ERRORS=$(curl -v https://otello-web.onrender.com/core/wake/ 2>&1 || echo "Errore sconosciuto")
          curl -s -X POST "https://api.telegram.org/bot${{ secrets.TELEGRAM_TOKEN }}/sendMessage" \
            -d chat_id="${{ secrets.TELEGRAM_CHAT_ID }}" \
            -d parse_mode="Markdown" \
            -d text="⚠️ Ping fallito su *${SITE}*.\nDettagli:\n\`\`\`\n${ERRORS}\n\`\`\`"

      - name: Salva il log come artifact
        uses: actions/upload-artifact@v4
        with:
          name: ping-log-otello
          path: log.txt
```

---

## 7. Workflow keepalive
Crea `.github/workflows/keepalive.yml`:

```yaml
name: Keepalive Workflow

on:
  schedule:
    - cron: '0 0 * * 0'  # ogni domenica a mezzanotte
  workflow_dispatch:

jobs:
  keepalive:
    runs-on: ubuntu-latest
    steps:
      - name: Keep workflow alive
        uses: liskin/gh-workflow-keepalive@main
        with:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          max_days_since_commit: 45
          commit_message: "keepalive: refresh to prevent cron disable"
```

---

## 8. Verifica e utilizzo
1. Fai commit e push dei file.
2. Vai su **Actions** e verifica che i workflow siano visibili.
3. Avvia manualmente `ping.yml` con “Run workflow” per testare.
4. Se il ping fallisce, riceverai un messaggio Telegram dal tuo bot.
5. Ogni run produce un artifact `ping-log-otello` scaricabile dalla pagina **Actions → Run → Artifacts**.

---

## 9. Esempio di log
Il file `log.txt` salvato come artifact contiene righe come:
```
2025-11-06 17:26:18 | https://otello-web.onrender.com/core/wake/ | HTTP 200
```
oppure:
```
2025-11-06 17:26:18 | https://otello-web.onrender.com/core/wake/ | ERR
```

---

## 📌 Note finali
- I workflow sono configurati per escludere la domenica.
- Gli artifacts non vengono committati nella repo: si scaricano dalla sezione **Actions**.
- Puoi aggiungere altri endpoint duplicando lo step `curl` in `ping.yml`.
- Il workflow è compatibile con le nuove versioni delle actions (`upload-artifact@v4`).
```

---

👉 Ora hai un **manuale completo in Markdown** pronto da incollare nel tuo `blueprint.md`.  
Vuoi che ti prepari anche una sezione finale “Troubleshooting” con gli errori più comuni (es. workflow non parte, secrets mancanti, artifact non visibile)?
