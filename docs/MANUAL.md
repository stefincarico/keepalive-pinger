# 📡 Keep Render Awake - Manuale d'uso

Script automatico per mantenere attivo un servizio Render e ricevere notifiche in caso di problemi.

---

## 📋 Indice

1. [Cosa fa lo script](#cosa-fa-lo-script)
2. [Prerequisiti](#prerequisiti)
3. [Installazione](#installazione)
4. [Configurazione](#configurazione)
5. [Come consultare i log](#come-consultare-i-log)
6. [Manutenzione](#manutenzione)
7. [Risoluzione problemi](#risoluzione-problemi)

---

## 🎯 Cosa fa lo script

Questo workflow GitHub Actions risolve un problema comune dei servizi gratuiti su Render: vanno in standby dopo un periodo di inattività, impiegando 30-60 secondi per riattivarsi.

**Funzionalità:**
- ✅ Esegue un ping HTTP ogni 15 minuti al servizio Render
- ✅ Funziona solo nei giorni lavorativi (lunedì-sabato) dalle 07:30 alle 22:15 (ora italiana)
- ✅ Invia notifiche Telegram immediate in caso di errori
- ✅ Mantiene uno storico permanente dei ping su GitHub Gist
- ✅ Crea backup giornalieri degli ultimi 7 giorni

---

## 🔧 Prerequisiti

### Account necessari:
1. **GitHub** (dove risiede questo repository)
2. **Telegram** (per ricevere le notifiche)
3. **Render** (il servizio da mantenere attivo)

### Tool richiesti:
- Nessuno! Tutto funziona su GitHub Actions

---

## 📥 Installazione

### Step 1: Crea un bot Telegram

1. Apri Telegram e cerca **@BotFather**
2. Invia il comando `/newbot`
3. Segui le istruzioni per scegliere nome e username
4. Copia il **token** che ti viene fornito (es: `123456789:ABCdefGHIjklMNOpqrsTUVwxyz`)
5. **Non condividere mai questo token!**

### Step 2: Ottieni il tuo Chat ID

**Metodo 1 (consigliato):**
1. Scrivi un messaggio qualsiasi al bot che hai appena creato
2. Apri questa URL nel browser (sostituisci `<TUO_TOKEN>`):
   ```
   https://api.telegram.org/bot<TUO_TOKEN>/getUpdates
   ```
3. Cerca nel JSON la riga `"chat":{"id":123456789` → quello è il tuo chat_id

**Metodo 2:**
1. Cerca **@userinfobot** su Telegram
2. Invia `/start`
3. Ti risponderà con il tuo ID

### Step 3: Crea un GitHub Gist per il log

1. Vai su https://gist.github.com
2. Crea un nuovo Gist:
   - **Filename:** `ping-log.txt`
   - **Contenuto:** scrivi una riga qualsiasi, ad esempio:
     ```
     # Log ping Render - Inizializzato il 2025-11-08
     ```
   - **Visibilità:** seleziona **"Create secret gist"**
3. Salva e copia l'**ID del Gist** dall'URL:
   ```
   https://gist.github.com/tuousername/a1b2c3d4e5f6g7h8
                                      ^^^^^^^^^^^^^^^^^^^^
                                      Questo è il GIST_ID
   ```

### Step 4: Configura i Secrets su GitHub

1. Vai nella repository su GitHub
2. Clicca su **Settings** → **Secrets and variables** → **Actions**
3. Clicca su **New repository secret** e crea questi 3 secrets:

| Nome Secret | Valore | Dove ottenerlo |
|-------------|--------|----------------|
| `GIST_ID` | L'ID del Gist (es: `a1b2c3d4e5f6g7h8`) | Step 3 |
| `TELEGRAM_TOKEN` | Il token del bot (es: `123456789:ABC...`) | Step 1 |
| `TELEGRAM_CHAT_ID` | Il tuo chat ID (es: `987654321`) | Step 2 |

**⚠️ Importante:** Il secret `GITHUB_TOKEN` è già disponibile automaticamente, non serve crearlo.

### Step 5: Crea il workflow

1. Nella repository, crea la cartella `.github/workflows/` (se non esiste già)
2. Crea il file `.github/workflows/ping.yml` con il contenuto dello script
3. Commit e push

---

## ⚙️ Configurazione

### Modificare gli orari di esecuzione

Il workflow usa la sintassi cron. Modifica le righe nel file `ping.yml`:

```yaml
schedule:
  - cron: '30,45 6 * * 1-6'      # 07:30, 07:45
  - cron: '0,15,30,45 7-20 * * 1-6'  # 08:00-21:45 ogni 15 min
  - cron: '0,15 21 * * 1-6'      # 22:00, 22:15
```

**Esempi di personalizzazione:**

#### Ping ogni 30 minuti invece di 15:
```yaml
schedule:
  - cron: '30 6-21 * * 1-6'  # Alle :30 di ogni ora
```

#### Attivo anche la domenica:
```yaml
schedule:
  - cron: '30,45 6 * * 0-6'  # Cambia 1-6 in 0-6
  - cron: '0,15,30,45 7-20 * * 0-6'
  - cron: '0,15 21 * * 0-6'
```

#### Solo giorni lavorativi (lun-ven):
```yaml
schedule:
  - cron: '30,45 6 * * 1-5'  # Cambia 1-6 in 1-5
  - cron: '0,15,30,45 7-20 * * 1-5'
  - cron: '0,15 21 * * 1-5'
```

**Riferimento cron:**
- `*` = ogni valore
- `0-6` = giorni della settimana (0=domenica, 1=lunedì, ..., 6=sabato)
- `7-20` = ore (in UTC, quindi aggiungi 1-2 ore per l'ora italiana)
- `0,15,30,45` = minuti specifici

### Modificare l'URL del servizio

Modifica la riga nello step "Ping servizio Render":

```bash
SITE="https://otello-web.onrender.com/core/wake/"
```

Sostituisci con il tuo endpoint.

### Disabilitare le notifiche per ping riusciti

Lo script invia notifiche **solo in caso di errore**. Se vuoi ricevere notifiche anche per ping riusciti, rimuovi questa condizione:

```yaml
- name: Invia notifica errore su Telegram
  if: steps.ping.outputs.status != '200'  # ← Rimuovi questa riga
```

---

## 📊 Come consultare i log

### Metodo 1: GitHub Gist (storico permanente)

1. Vai su https://gist.github.com
2. Nella sidebar sinistra, cerca il tuo Gist `ping-log.txt`
3. Clicca per visualizzare lo storico completo

**Formato log:**
```
2025-11-08 14:30:00 | HTTP 200
2025-11-08 14:45:00 | HTTP 200
2025-11-08 15:00:00 | HTTP 500
2025-11-08 15:15:00 | HTTP 200
```

**Nota:** Il log mantiene solo le ultime 500 righe per evitare file troppo grandi.

### Metodo 2: GitHub Actions (ultimi 90 giorni)

1. Vai nella repository su GitHub
2. Clicca sul tab **Actions**
3. Clicca su **Keep Render Awake** nella sidebar sinistra
4. Clicca su una delle esecuzioni recenti
5. Espandi lo step **"Ping servizio Render"**
6. Vedrai l'output: `✅ Ping eseguito: 200`

### Metodo 3: Artifacts (ultimi 7 giorni)

1. Vai su **Actions** → **Keep Render Awake**
2. Clicca su un'esecuzione recente
3. In fondo alla pagina, nella sezione **Artifacts**, troverai `ping-log-backup`
4. Scarica il file ZIP contenente `log.txt`

### Metodo 4: Notifiche Telegram

Ricevi automaticamente un messaggio quando il ping fallisce:

```
⚠️ Ping fallito

Servizio: `otello-web.onrender.com`
Codice HTTP: `500`
Orario: 2025-11-08 15:00:00
```

---

## 🔄 Manutenzione

### Testare manualmente lo script

1. Vai su **Actions** → **Keep Render Awake**
2. Clicca sul pulsante **Run workflow** (in alto a destra)
3. Seleziona il branch (solitamente `main`)
4. Clicca **Run workflow**
5. Attendi 1-2 minuti e controlla i risultati

### Verificare che i workflow siano attivi

GitHub disabilita automaticamente i workflow schedulati dopo 60 giorni di inattività della repository.

**Soluzione:** Lo script `keepalive.yml` (se presente) fa un commit automatico ogni domenica per prevenire questo problema.

**Verifica manuale:**
1. Vai su **Actions**
2. Nella sidebar, controlla che **Keep Render Awake** non abbia il badge "disabled"

### Aggiornare i secrets

Se devi cambiare token Telegram, chat ID o Gist ID:

1. Vai su **Settings** → **Secrets and variables** → **Actions**
2. Clicca sul secret da modificare
3. Clicca **Update secret**
4. Inserisci il nuovo valore

---

## 🐛 Risoluzione problemi

### Il workflow non parte automaticamente

**Causa:** GitHub Actions potrebbe essere disabilitato per la repository.

**Soluzione:**
1. Vai su **Settings** → **Actions** → **General**
2. Verifica che "Allow all actions and reusable workflows" sia selezionato
3. Scorri in basso e verifica che "Allow GitHub Actions to create and approve pull requests" sia abilitato (se necessario)

### Non ricevo notifiche Telegram

**Possibili cause e soluzioni:**

1. **Token o Chat ID errati**
   - Verifica i secrets in **Settings** → **Secrets and variables**
   - Prova a rigenerare il token da @BotFather

2. **Non hai avviato il bot**
   - Apri Telegram e scrivi `/start` al tuo bot

3. **Il ping ha successo (HTTP 200)**
   - Le notifiche vengono inviate **solo in caso di errore**
   - Per testare, puoi temporaneamente modificare l'URL del servizio con uno errato

### Errore "Invalid workflow file"

**Causa:** Sintassi YAML errata.

**Soluzione:**
1. Copia nuovamente lo script da questo manuale
2. Assicurati di non avere tabulazioni (usa solo spazi)
3. Verifica che non ci siano caratteri strani copiati per errore
4. Usa un validatore YAML online: https://www.yamllint.com/

### Il log non si aggiorna sul Gist

**Possibili cause:**

1. **GIST_ID errato**
   - Verifica che l'ID nel secret corrisponda all'URL del Gist

2. **Permessi insufficienti**
   - Vai sul Gist e verifica che sia tuo (non puoi modificare Gist di altri)

3. **Gist cancellato**
   - Crea un nuovo Gist e aggiorna il secret

### Il servizio Render va ancora in standby

**Causa:** Il ping non arriva abbastanza frequentemente.

**Soluzione:**
1. Verifica che il workflow stia effettivamente partendo (controlla **Actions**)
2. Aumenta la frequenza dei ping (es: ogni 10 minuti invece di 15)
3. Verifica che l'endpoint `/core/wake/` sia corretto

### Timeout del workflow

**Errore:** `The job running on runner ... has exceeded the maximum execution time of 3 minutes`

**Causa:** Il servizio impiega troppo tempo a rispondere.

**Soluzione:** Aumenta il timeout nel file `ping.yml`:
```yaml
timeout-minutes: 5  # Aumenta da 3 a 5
```

### Errore "jq: command not found"

**Causa:** L'immagine Ubuntu usata da GitHub Actions non ha `jq` installato (raro, ma possibile).

**Soluzione:** Aggiungi questo step prima di "Scarica log precedente":
```yaml
- name: Installa dipendenze
  run: sudo apt-get update && sudo apt-get install -y jq
```

---

## 📈 Statistiche d'uso

### Consumo GitHub Actions

- **Esecuzioni giornaliere:** ~60 (4 per ora × 15 ore)
- **Durata media:** ~30 secondi per esecuzione
- **Tempo totale mensile:** ~30 minuti/mese
- **Limiti GitHub Free:** 2.000 minuti/mese ✅ (consuma ~1.5%)

### Storage utilizzato

- **Gist log:** ~10-50 KB (max 500 righe)
- **Artifacts:** ~1 KB × 7 giorni = ~7 KB
- **Totale:** Trascurabile

---

## 🔐 Note sulla sicurezza

### ✅ Sicuro

- I secrets (`TELEGRAM_TOKEN`, `TELEGRAM_CHAT_ID`, `GIST_ID`) non sono mai visibili nei log
- Il Gist è privato (secret gist)
- GitHub Actions esegue in un ambiente isolato

### ⚠️ Considerazioni

- **Repository pubblica:** Chiunque può vedere che stai pingando `otello-web.onrender.com`
- **Chat ID nei log:** Il `TELEGRAM_CHAT_ID` potrebbe apparire nei log pubblici (non è critico, ma visibile)
- **Rate limiting:** Non superare 1 ping ogni 5 minuti per evitare di essere bloccato da Render

### 🔒 Raccomandazioni

1. Non condividere mai i token Telegram
2. Non commitare secrets direttamente nel codice
3. Usa sempre GitHub Secrets per dati sensibili
4. Se la repository contiene codice sensibile, rendila privata

---

## 📞 Supporto

### Documentazione ufficiale

- **GitHub Actions:** https://docs.github.com/actions
- **Telegram Bot API:** https://core.telegram.org/bots/api
- **GitHub Gists:** https://docs.github.com/gists
- **Sintassi cron:** https://crontab.guru/

### Check-list debug

Prima di chiedere aiuto, verifica:

- [ ] Tutti i secrets sono configurati correttamente
- [ ] Il workflow è abilitato in Settings → Actions
- [ ] Hai eseguito almeno un test manuale
- [ ] Hai controllato i log in Actions → Keep Render Awake
- [ ] Il bot Telegram è stato avviato con `/start`
- [ ] L'URL del servizio è corretto e raggiungibile

---

## 📝 Changelog

### v1.0.0 (2025-11-08)
- Prima versione stabile
- Ping ogni 15 minuti
- Notifiche Telegram
- Log su GitHub Gist
- Backup su Artifacts

# 📝 Sintesi: Implementazione sistema di log con GitHub Gist

## 🎯 Panoramica

Il sistema di log utilizza **GitHub Gist** come storage permanente per mantenere uno storico dei ping (ultime 500 righe). Per funzionare, richiede un **Personal Access Token** con permessi specifici sui Gist.

---

## 🔑 Step 1: Creare un Personal Access Token (PAT)

### Perché serve
Il `GITHUB_TOKEN` automatico di GitHub Actions **non può modificare i Gist** per limitazioni di sicurezza. Serve un token personale con permessi espliciti.

### Procedura

1. **Vai nelle impostazioni sviluppatore**
   - Clicca sulla tua **foto profilo** (in alto a destra su GitHub)
   - **Settings** → **Developer settings** (in fondo alla sidebar)
   - **Personal access tokens** → **Tokens (classic)**

2. **Genera un nuovo token**
   - Clicca **Generate new token** → **Generate new token (classic)**

3. **Configura il token**
   - **Note:** `Gist Writer for Keep-Alive` (o qualsiasi nome descrittivo)
   - **Expiration:** Scegli la durata (es: `90 days`, `1 year`, `No expiration`)
   - **Scopes:** Seleziona **SOLO** `gist` (Create gists)
     - ✅ `gist`
     - ❌ Non selezionare altro per sicurezza

4. **Genera e salva**
   - Clicca **Generate token** in fondo
   - **Copia immediatamente** il token (inizia con `ghp_...`)
   - ⚠️ **Salvalo in un posto sicuro** - non potrai rivederlo!

---

## 📦 Step 2: Creare il GitHub Gist

### Perché serve
Il Gist è il "database" dove viene salvato lo storico dei ping in formato testo.

### Procedura

1. **Vai su GitHub Gist**
   - Apri https://gist.github.com

2. **Crea un nuovo Gist**
   - **Gist description:** `log per il keep-alive ping otello` (opzionale)
   - **Filename:** `ping-log.txt`
   - **Contenuto:** Scrivi una riga iniziale, ad esempio:
     ```
     # Log ping Render - Inizializzato 2025-11-08
     ```
   - ⚠️ Il contenuto non può essere vuoto, GitHub non lo permette

3. **Crea come Secret Gist**
   - Clicca **Create secret gist** (NON "Create public gist")
   - Questo rende il Gist privato

4. **Copia l'ID del Gist**
   - Guarda l'URL del browser:
     ```
     https://gist.github.com/tuousername/a1b2c3d4e5f6g7h8i9j0
                                        ^^^^^^^^^^^^^^^^^^^^
                                        Questo è il GIST_ID
     ```
   - Copia la parte finale (senza `/`)

---

## 🔐 Step 3: Configurare i Secrets su GitHub

### Secrets necessari

Devi configurare **4 secrets** totali nella repository:

| Secret | Valore | Dove ottenerlo |
|--------|--------|----------------|
| `GIST_ID` | ID del Gist | Step 2, punto 4 |
| `GIST_TOKEN` | Personal Access Token | Step 1, punto 4 |
| `TELEGRAM_TOKEN` | Token bot Telegram | Da @BotFather |
| `TELEGRAM_CHAT_ID` | ID della chat | Da @userinfobot o API |

### Procedura

1. **Vai nella repository**
   - Clicca su **Settings** (in alto)

2. **Vai su Secrets**
   - Sidebar sinistra: **Secrets and variables** → **Actions**

3. **Aggiungi ogni secret**
   - Clicca **New repository secret**
   - **Name:** (es: `GIST_TOKEN`)
   - **Secret:** (incolla il valore)
   - Clicca **Add secret**
   - Ripeti per tutti e 4 i secrets

---

## 📄 Step 4: Modificare il workflow

### Modifiche necessarie

Il workflow deve usare `GIST_TOKEN` invece di `GITHUB_TOKEN` in **DUE punti**:

#### ⚠️ Punto 1: Scaricamento del log
```yaml
- name: Scarica log precedente da Gist
  run: |
    curl -s -H "Authorization: token ${{ secrets.GIST_TOKEN }}" \
      "https://api.github.com/gists/$GIST_ID" | ...
```

#### ⚠️ Punto 2: Aggiornamento del log
```yaml
- name: Aggiorna log su Gist
  run: |
    curl -s -X PATCH \
      -H "Authorization: token ${{ secrets.GIST_TOKEN }}" \
      "https://api.github.com/gists/$GIST_ID" ...
```

### Errore comune
❌ Usare `GIST_TOKEN` solo in un punto → il workflow fallisce con `Resource not accessible by integration`

✅ Usare `GIST_TOKEN` in **entrambi i punti** (download E upload)

---

## ✅ Step 5: Test e verifica

### Test manuale

1. **Esegui il workflow**
   - **Actions** → **Keep Render Awake** → **Run workflow**

2. **Controlla i log**
   - Clicca sull'esecuzione
   - Espandi lo step **"Aggiorna log su Gist"**
   - Cerca: `✅ Log caricato su Gist con successo!`

3. **Verifica sul Gist**
   - Vai su https://gist.github.com
   - Apri il tuo `ping-log.txt`
   - Dovresti vedere:
     ```
     # Log ping Render - Inizializzato
     2025-11-08 15:30:00 | HTTP 200
     2025-11-08 15:45:00 | HTTP 200
     ```

### Cosa controllare in caso di errori

| Errore | Causa | Soluzione |
|--------|-------|-----------|
| `Resource not accessible by integration` | `GITHUB_TOKEN` usato invece di `GIST_TOKEN` | Verifica di aver sostituito in ENTRAMBI i punti |
| `Not Found` | `GIST_ID` errato | Ricontrolla l'ID copiato dal browser |
| `Bad credentials` | `GIST_TOKEN` errato o scaduto | Rigenera il token e aggiorna il secret |
| Log vuoto sul Gist | Token manca dello scope `gist` | Rigenera il token assicurandoti di selezionare `gist` |

---

## 📊 Come funziona il sistema

### Flusso ad ogni esecuzione

```
1. ⬇️  Scarica log precedente dal Gist (usando GIST_TOKEN)
2. ✂️  Mantiene solo le ultime 500 righe
3. 🌐 Esegue ping al servizio Render
4. ✍️  Aggiunge nuova riga al log locale
5. ⬆️  Carica log aggiornato sul Gist (usando GIST_TOKEN)
6. 💾 Salva backup come artifact (7 giorni)
```

### Gestione dello storico

- **Limite:** 500 righe (~3-4 giorni di storico)
- **Rotazione:** Automatica, mantiene solo le più recenti
- **Persistenza:** Permanente sul Gist (finché non lo cancelli)
- **Backup:** Artifact di GitHub (ultimi 7 giorni)

---

## 📋 Checklist finale

Prima di considerare l'implementazione completa, verifica:

- [ ] Personal Access Token creato con scope `gist`
- [ ] Token salvato in un posto sicuro
- [ ] Gist creato come "Secret" (non pubblico)
- [ ] GIST_ID copiato correttamente
- [ ] Secret `GIST_ID` configurato
- [ ] Secret `GIST_TOKEN` configurato
- [ ] Secret `TELEGRAM_TOKEN` configurato
- [ ] Secret `TELEGRAM_CHAT_ID` configurato
- [ ] Workflow modificato in ENTRAMBI i punti
- [ ] Test manuale eseguito con successo
- [ ] Log visibile sul Gist dopo l'esecuzione

---

## 🔒 Note sulla sicurezza

### Token PAT
- ✅ Ha accesso **solo** ai Gist (scope limitato)
- ✅ È salvato come secret (crittografato)
- ✅ Non appare mai nei log
- ⚠️ Ha una scadenza (se configurata)
- ⚠️ Può essere revocato in qualsiasi momento

### Gist
- ✅ È "secret" quindi non pubblico
- ✅ Ma chiunque abbia il link può vederlo
- ✅ Non contiene dati sensibili (solo timestamp e codici HTTP)

### Raccomandazioni
1. Usa sempre token con scope minimi necessari
2. Imposta una scadenza ragionevole (es: 1 anno)
3. Rigenera periodicamente il token
4. Revoca immediatamente se compromesso

---

## 🔄 Manutenzione

### Rigenerare il token (quando scade)

1. Developer settings → Personal access tokens
2. Trova il vecchio token → **Delete**
3. Genera un nuovo token (stessa procedura)
4. Aggiorna il secret `GIST_TOKEN` nella repository

### Consultare il log

**Metodo rapido:**
- Vai su https://gist.github.com
- Clicca su `ping-log.txt`
- Visualizza lo storico

**Metodo avanzato:**
- URL diretta: `https://gist.github.com/tuousername/[GIST_ID]/raw`
- Salvala nei preferiti per accesso rapido

---

## 📚 Riepilogo tecnico

| Componente | Tecnologia | Scopo |
|------------|------------|-------|
| **Storage principale** | GitHub Gist | Storico permanente (500 righe) |
| **Autenticazione** | Personal Access Token | Permessi scrittura Gist |
| **Backup** | GitHub Artifacts | Ridondanza (7 giorni) |
| **Rotazione** | `tail -n 500` | Limita dimensione file |
| **Formato** | Plain text | `YYYY-MM-DD HH:MM:SS \| HTTP XXX` |

---

Questa sintesi copre tutti i passaggi critici per implementare il sistema di log funzionante! 🎉


---

## 📄 Licenza

Questo script è fornito "as is" senza garanzie. Usalo a tuo rischio.

---

**Ultima modifica:** 8 novembre 2025
