# Pubblicare il calendario

Questi passaggi si fanno una sola volta da parte di chi gestisce l'app. Ai visitatori basta aprire il link, accedere con Google o ricevere un link di accesso via email, e impostare la loro data di nascita.

## Accesso e note

Crea un progetto Supabase e applica `supabase/schema.sql` dal suo editor SQL. Il database rende visibile a ciascun account solo il suo profilo e le sue note.

Pubblica `index.html` su un hosting HTTPS. Prima di pubblicare, imposta `supabaseUrl` e `supabasePublishableKey` nel blocco JSON `calendar-config`. Usa la chiave pubblicabile, mai la chiave di servizio. Iscrivi l'URL esatto del calendario come Site URL e come Redirect URL nella configurazione di Supabase Auth.

Abilita il provider Google in Supabase Auth. In Google Cloud crea un client OAuth per applicazioni web e aggiungi nei redirect autorizzati l'URL di callback che Supabase mostra per il provider Google. Inserisci ID e segreto del client solo nella configurazione di Supabase. Imposta il pubblico Google su External e pubblica la configurazione OAuth quando è pronta.

Abilita Email in Supabase Auth e mantieni il link `{{ .ConfirmationURL }}` nel template Magic Link. Configura un mittente SMTP personalizzato per consegnare i link anche a utenti esterni al team. Dopo la pubblicazione, prova i due accessi e la separazione delle note con due account distinti.

Chi usa la versione locale può scaricare le note in Impostazioni e importare lo stesso JSON dopo aver creato il suo calendario online.

## Invii programmati con Gmail

Puoi attivare questa funzione separatamente dal login. In Google Cloud abilita Gmail API e crea un client OAuth web dedicato. Aggiungi come redirect esatto `https://<progetto>.supabase.co/functions/v1/calendar-gmail/callback`. Per consentire invii da account pubblici prepara il consenso per lo scope `https://www.googleapis.com/auth/gmail.send` e completa la verifica richiesta da Google.

Nella funzione Supabase imposta i segreti `CALENDAR_GOOGLE_ID`, `CALENDAR_GOOGLE_SECRET`, `CALENDAR_APP_URL`, `CALENDAR_TOKEN_KEY` e `CALENDAR_CRON_SECRET`. L'URL è quello pubblico del calendario. Genera la chiave dei token da 32 byte casuali codificati in Base64. Genera un segreto casuale distinto per la coda. Conserva chiavi e segreti nel progetto Supabase, fuori da HTML e repository. La funzione usa anche le variabili Supabase riservate fornite dal progetto.

Distribuisci la funzione `calendar-gmail` con la configurazione contenuta in `supabase/config.toml`. La funzione verifica la sessione Supabase per le richieste dell'app, lo stato OAuth monouso per la callback Google e il segreto del cron per gli invii.

Abilita Supabase Cron e pg_net. Inserisci in Vault `calendar_worker_url`, con valore `https://<progetto>.supabase.co/functions/v1/calendar-gmail/deliver`, e `calendar_cron_secret`, con lo stesso valore di `CALENDAR_CRON_SECRET`. Applica `supabase/cron.sql` una volta. Solo dopo aver verificato il collegamento e la coda, imposta `gmailEnabled` a `true` nell'HTML e ripubblica.

Nell'app l'utente sceglie **Collega Gmail** solo se vuole inviare posta. Il mittente è lo stesso indirizzo confermato per il suo account. La coda controlla gli invii ogni cinque minuti, usa l'ora italiana e continua a funzionare a pagina chiusa. Gli esiti incerti richiedono una verifica prima di programmare di nuovo lo stesso messaggio.

[Google login in Supabase](https://supabase.com/docs/guides/auth/social-login/auth-google)

[Configurazione SMTP](https://supabase.com/docs/guides/auth/auth-smtp)

[Invii Gmail](https://developers.google.com/workspace/gmail/api/auth/web-server)
