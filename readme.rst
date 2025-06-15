Centro Notifiche Avanzato - Versione Standalone
📖 Panoramica

Questo progetto è una versione standalone del notifier AppDaemon, completamente riscritta come blueprint nativo di Home Assistant. Offre tutte le funzionalità del sistema originale senza dipendere da AppDaemon.
✨ Caratteristiche Principali

    Multi-canale: Telegram, Push Mobile, TTS, Alexa
    Gestione priorità: 4 livelli di priorità (bassa, normale, alta, critica)
    Modalità silenziosa: Ore silenziose e modalità Non Disturbare
    Ripetizioni intelligenti: Ripete le notifiche se non riconosciute
    Logging avanzato: Tracciamento completo delle notifiche
    Dashboard integrata: Interfaccia completa per gestione e monitoraggio
    Template Jinja2: Supporto completo per messaggi dinamici
    Statistiche: Contatori e analisi delle notifiche inviate

🚀 Installazione
Passo 1: Installare il Blueprint

    Copia il contenuto del file Blueprint Centro Notifiche
    Vai in Home Assistant → Impostazioni → Automazioni e Scene → Blueprint
    Clicca su "Importa Blueprint" → "Importa da file"
    Incolla il codice YAML del blueprint

Passo 2: Configurare il Package Helper

    Crea la cartella packages nella directory di configurazione se non esiste
    Aggiungi questa riga al tuo configuration.yaml:

    yaml

    homeassistant:
      packages: !include_dir_named packages

    Salva il file Package Helper come packages/notifier_center.yaml
    Riavvia Home Assistant

Passo 3: Configurare la Dashboard (Opzionale)

    Vai in Panoramica → ⋮ Menu → Modifica dashboard
    Aggiungi una nuova scheda
    Seleziona "Mostra editor di codice"
    Incolla il contenuto del file Dashboard Lovelace

⚙️ Configurazione
Servizi di Notifica Richiesti

Prima di utilizzare il blueprint, assicurati di aver configurato i servizi di notifica:
Telegram

yaml

# configuration.yaml
telegram_bot:
  - platform: polling
    api_key: "TUO_BOT_TOKEN"
    allowed_chat_ids:
      - TUO_CHAT_ID

notify:
  - platform: telegram
    name: telegram
    chat_id: TUO_CHAT_ID

Mobile App

Installa l'app ufficiale Home Assistant sul tuo dispositivo mobile. Il servizio notify sarà automaticamente disponibile come notify.mobile_app_NOME_DISPOSITIVO.
TTS

yaml

# configuration.yaml
tts:
  - platform: google_translate
    language: 'it'

Alexa (con Alexa Media Player)

Installa il componente personalizzato Alexa Media Player tramite HACS.
Creazione di una Notifica

    Vai in Impostazioni → Automazioni e Scene
    Clicca su "Aggiungi Automazione"
    Seleziona "Sfoglia Blueprint"
    Scegli "Centro Notifiche Avanzato"
    Configura i parametri secondo le tue esigenze

Esempio di Configurazione

yaml

# Notifica per porta aperta
alias: "Notifica Porta Ingresso"
use_blueprint:
  path: centro_notifiche_avanzato.yaml
  input:
    trigger_entity: binary_sensor.porta_ingresso
    trigger_state: "on"
    notification_title: "🚪 Porta Aperta"
    notification_message: "La porta d'ingresso è stata aperta alle {{ now().strftime('%H:%M') }}"
    enable_telegram: true
    telegram_service: "notify.telegram"
    enable_mobile: true
    mobile_service: "notify.mobile_app_iphone"
    notification_priority: "normal"
    enable_dnd: true
    dnd_entity: "input_boolean.notifier_dnd"
    respect_quiet_hours: true
    quiet_hours_start: "22:00:00"
    quiet_hours_end: "07:00:00"

🎯 Funzionalità Avanzate
Template Jinja2 nei Messaggi

Puoi utilizzare template per messaggi dinamici:

yaml

notification_message: |
  {% if trigger.to_state.state == 'on' %}
    🔓 La {{ trigger.entity_id.split('.')[1] | replace('_', ' ') | title }} è stata aperta
  {% else %}
    🔒 La {{ trigger.entity_id.split('.')[1] | replace('_', ' ') | title }} è stata chiusa
  {% endif %}
  
  Orario: {{ now().strftime('%H:%M:%S') }}
  Data: {{ now().strftime('%d/%m/%Y') }}

Dati Extra per Canali Specifici
Telegram - Tastiera Inline

yaml

telegram_data: |
  parse_mode: 'HTML'
  inline_keyboard:
    - "Disattiva Allarme:/disattiva_allarme"
    - "Stato Casa:/stato_casa"

Mobile - Azioni e Icone

yaml

mobile_data: |
  actions:
    - action: "ALARM_DISARM"
      title: "Disattiva"
    - action: "VIEW_CAMERA"
      title: "Telecamera"
  image: "/local/icons/security.png"
  icon_url: "/local/icons/notification.png"
  color: "#ff5722"

Condizioni Avanzate

Puoi aggiungere condizioni personalizzate:

yaml

additional_condition: |
  {{ is_state('alarm_control_panel.home', 'armed_away') and 
     states('sensor.people_home') | int == 0 }}

Gestione Priorità

    Bassa (low): Notifiche informative, rispetta tutte le modalità silenziose
    Normale (normal): Notifiche standard, rispetta ore silenziose e DND
    Alta (high): Notifiche importanti, ignora ore silenziose se soglia ≤ 2
    Critica (critical): Notifiche di emergenza, ignora sempre tutte le modalità

📊 Monitoraggio e Statistiche
Dashboard Integrata

La dashboard fornisce:

    Stato in tempo reale del sistema
    Controlli per modalità DND e manutenzione
    Statistiche dettagliate per canale
    Storico delle ultime notifiche
    Script di test e utilità
    Grafici di andamento

Log del Sistema

Tutte le notifiche vengono registrate nel log di Home Assistant con il logger blueprint.notifier. Puoi visualizzarle in:

    Strumenti di Sviluppo → Log
    File di log: home-assistant.log

Contatori e Statistiche

Il sistema mantiene contatori automatici per:

    Notifiche per canale (Telegram, Mobile, TTS, Alexa)
    Notifiche totali giornaliere
    Storico delle ultime notifiche per canale

🛠️ Troubleshooting
Problemi Comuni
Le notifiche non vengono inviate

    Verifica che i servizi di notifica siano configurati correttamente
    Controlla se la modalità DND o manutenzione sono attive
    Verifica le condizioni di orario (ore silenziose)
    Controlla i log per errori specifici

Template non funzionano

    Testa i template in Strumenti di Sviluppo → Template
    Verifica la sintassi Jinja2
    Assicurati che le entità referenziate esistano

TTS/Alexa non funziona

    Verifica che i media player siano online e disponibili
    Controlla i volumi configurati
    Testa manualmente i servizi TTS/Alexa

Debug Avanzato

Abilita il debug per il logger specifico:

yaml

# configuration.yaml
logger:
  default: info
  logs:
    blueprint.notifier: debug

🔧 Personalizzazioni
Aggiungere Nuovi Canali

Per aggiungere supporto per altri servizi di notifica (es. Discord, Slack), modifica il blueprint aggiungendo:

    Nuovi input per abilitare/configurare il servizio
    Sezione nell'action per l'invio
    Contatori e logging appropriati

Integrazioni Avanzate

Il sistema può essere integrato con:

    Sistema di allarmi
    Automazioni di presenza
    Sensori IoT
    Telecamere di sicurezza
    Sistema domotico completo

📝 Migrazione da AppDaemon

Se stai migrando dal notifier AppDaemon originale:

    Backup: Salva le configurazioni AppDaemon esistenti
    Installazione: Installa questo blueprint come descritto sopra
    Ricreazione: Ricrea le automazioni utilizzando il nuovo blueprint
    Test: Verifica che tutte le notifiche funzionino correttamente
    Rimozione: Rimuovi le configurazioni AppDaemon obsolete

Differenze Principali

    Indipendenza: Non richiede AppDaemon
    Integrazione nativa: Utilizza solo funzionalità core di Home Assistant
    Dashboard: Include dashboard di gestione integrata
    Performance: Minore overhead di sistema
    Manutenzione: Più semplice da aggiornare e mantenere

🤝 Supporto

Per supporto, problemi o suggerimenti:

    Verifica la documentazione sopra
    Controlla i log di Home Assistant
    Testa con configurazioni semplificate
    Verifica la sintassi YAML

📄 Licenza

Questo progetto è rilasciato sotto licenza MIT. Liberamente utilizzabile e modificabile.

Versione: 1.0
Compatibilità: Home Assistant 2024.1+
Autore: Conversione da AppDaemon a Blueprint nativo
Data: 2025
