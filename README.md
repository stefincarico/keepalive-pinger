# GitHub Ping Keepalive – otello-web.onrender.com

Questo repository contiene due workflow GitHub Actions per mantenere attivo il sito [otello-web.onrender.com](https://otello-web.onrender.com):

- **Ping Workflow**  
  Esegue un ping al sito ogni 15 minuti dalle 07:30 alle 22:15, dal lunedì al sabato.  
  In caso di fallimento, invia una notifica su Telegram con i dettagli dell’errore.

- **Keepalive Workflow**  
  Eseguito settimanalmente per mantenere attivi i workflow pianificati, evitando la disattivazione automatica di GitHub.

## 🔧 Configurazione

1. **Crea un bot Telegram** con [BotFather](https://t.me/BotFather) e ottieni il token.
2. **Scrivi un messaggio al bot** e recupera il tuo `chat_id` tramite l’API `getUpdates`.
3. **Aggiungi i secrets** nella repository:
   - `TELEGRAM_TOKEN` → il token del bot.
   - `TELEGRAM_CHAT_ID` → il numero chat.id.
4. Personalizza l’URL da pingare modificando `ping.yml` (in questo caso è già impostato su `https://otello-web.onrender.com`).

## 📌 Note

- I workflow sono configurati per escludere la domenica.
- Puoi aggiungere altri URL duplicando lo step `curl` in `ping.yml`.
- Il workflow keepalive previene la disattivazione automatica dei cronjob.
