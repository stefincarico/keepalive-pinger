# GitHub Ping Keepalive – otello-web.onrender.com/core/wake/

Questo repository contiene workflow GitHub Actions per monitorare e mantenere attivo il sito [otello-web.onrender.com/core/wake/](https://otello-web.onrender.com/core/wake/).

## 🚀 Funzionalità

- **Ping programmato**  
  Il workflow esegue un ping alla pagina `/core/wake/` ogni 15 minuti dalle 07:30 alle 22:15, dal lunedì al sabato.  
  La domenica è esclusa.

- **Notifica Telegram**  
  In caso di fallimento del ping, viene inviato un messaggio Telegram con i dettagli dell’errore.  
  Il bot Telegram è configurato tramite i secrets `TELEGRAM_TOKEN` e `TELEGRAM_CHAT_ID`.

- **Log via artifact**  
  Ogni esecuzione genera un file `log.txt` con:
  - Timestamp
  - URL della pagina pingata
  - Codice di risposta HTTP (o `ERR` in caso di errore)  

  Il file `log.txt` viene salvato come artifact scaricabile dalla pagina **Actions → Run → Artifacts**.

- **Keepalive workflow**  
  Un secondo workflow settimanale mantiene attivi i cronjob, evitando che GitHub li disattivi per inattività.

---

## 🔧 Configurazione

1. **Crea un bot Telegram** con [BotFather](https://t.me/BotFather) e ottieni il token.  
2. **Scrivi un messaggio al bot** e recupera il tuo `chat_id` tramite l’API `getUpdates`.  
3. **Aggiungi i secrets** nella repository:
   - `TELEGRAM_TOKEN` → il token del bot.
   - `TELEGRAM_CHAT_ID` → il numero chat.id.  
4. **Verifica i workflow**:
   - `ping.yml` → ping programmato su `/core/wake/` + logging + notifica Telegram.
   - `keepalive.yml` → mantiene attivi i cronjob.

---

## 📌 Note

- Gli artifacts sono scaricabili dalla sezione **Actions** di GitHub, non vengono committati nel repository.  
- Puoi aggiungere altri endpoint duplicando lo step `curl` in `ping.yml`.  
- Il workflow è compatibile con le nuove versioni delle actions (`upload-artifact@v4`).  
