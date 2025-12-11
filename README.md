# Gemini CLI Modules — agents.toml

Questo repository usa un file agents.toml per definire "agents" (comandi/moduli) che Gemini può eseguire e usare all'interno dei prompt, in modo simile alle "Skills" di altre piattaforme.  
Il file agents.toml è un elenco di comandi configurabili che vanno posizionati nella cartella `.gemini` (a livello di progetto o nella home dell'utente) e che il CLI di Gemini carica per esporre funzionalità esterne alla AI.

## Dove posizionare agents.toml

- Per uso globale (tutte le sessioni/ progetti dell'utente): `~/.gemini/agents.toml`  
- Per uso specifico al progetto: `.gemini/agents.toml` nella radice del repository

Scegli la posizione più adatta: per comandi che dipendono dal progetto è preferibile la versione in `./.gemini`; per comandi riutilizzabili globalmente usa `~/.gemini`.

## A cosa serve

- Definire comandi esterni che Gemini può eseguire e il cui output può essere incorporato nei prompt.
- Consentire la creazione di "moduli" riutilizzabili: estrazione dati, trasformazioni, integrazioni con API, ecc.
- Standardizzare nome, descrizione e modalità di esecuzione dei comandi per semplificare l'uso da CLI o da interfacce automatizzate.

## Formato (sintassi TOML) — struttura consigliata

Ogni agent è tipicamente una tabella ripetuta (es. `[[agent]]`) con campi comuni. Ecco i campi più utili (esempi di campo, alcuni opzionali):

- name (string) — nome identificativo dell'agent
- id (string, opzionale) — identificatore unico
- description (string) — breve descrizione di cosa fa
- command (string) — comando o percorso allo script/eseguibile
- args (array di stringhe, opzionale) — argomenti predefiniti
- env (table, opzionale) — variabili d'ambiente richieste (valori o riferimenti a env vars)
- working_dir (string, opzionale) — directory da cui eseguire il comando
- shell (bool, opzionale) — se eseguire il comando via shell
- platform (array, opzionale) — piattaforme supportate, es. ["linux","darwin","windows"]
- timeout (int, opzionale) — timeout in secondi
- interactive (bool, opzionale) — se il comando richiede interazione
- show_in_help (bool, opzionale) — se elencare l'agent nella guida/lista pubblica
- triggers (array di stringhe, opzionale) — parole chiave che possono attivare l'agent
- permissions (array, opzionale) — permessi richiesti (es. accesso rete, file system)

Questa è una bozza d'esempio:

```toml
[[agent]]
name = "gpt-data-fetcher"
description = "Estrae dati da un'API e li formatta per inserirli nei prompt"
command = "scripts/fetch_data.sh"
args = ["--limit", "10"]
shell = true
working_dir = "tools"
timeout = 30
platform = ["linux", "darwin"]
show_in_help = true

[[agent]]
name = "local-summarizer"
description = "Riassume testo lungo tramite uno script locale"
command = "python tools/summarize.py"
args = []
shell = true
interactive = false
```

Nota: usa percorsi relativi (rispetto a `working_dir` o alla cartella `.gemini`) o percorsi assoluti; assicurati che gli script siano eseguibili.

## Sicurezza e variabili sensibili

- Non inserire chiavi API o segreti direttamente in agents.toml. Usa variabili d'ambiente e riferimenti (es. `env = { API_KEY = "env:MY_API_KEY" }`) oppure meccanismi di secret management.
- Controlla i permessi dei file (`chmod +x`) per gli script eseguibili.
- Limita i permessi degli agent che eseguono comandi potenzialmente pericolosi.

## Esempi di uso

- Lista agent disponibili:
  - `gemini agents list` (comando esemplificativo — dipende dall'implementazione CLI)
- Eseguire un agent:
  - `gemini run gpt-data-fetcher --input "Domanda..."`  
  (la CLI esegue il comando definito, raccoglie l'output e lo passa al modello o lo mostra all'utente)

(Comandi CLI esatti possono variare in base alla versione del CLI; i nomi sopra sono indicativi.)

## Consigli pratici

- Testa ogni comando dalla riga di comando prima di inserirlo in agents.toml.
- Mantieni le descrizioni concise ma informative per migliorare l'usabilità.
- Se l'agent produce output strutturato (JSON), documenta il formato in modo che il codice che invoca l'agent sappia come interpretarlo.
- Tieni agents.toml sotto controllo versione se è parte del progetto; evita di commitare segreti.

## Troubleshooting

- Errore esecuzione: verifica che il comando esista e sia eseguibile.
- Timeout: aumenta `timeout` se il comando impiega più tempo.
- Problemi di percorso: verifica `working_dir` e i percorsi relativi.
- Malformazione TOML: usa un linter TOML o un parser per verificare la sintassi.

---

Se vuoi, incolla qui il contenuto reale di `agents.toml` e aggiorno il README per descrivere precisamente i campi e ogni agent presente nel file.
