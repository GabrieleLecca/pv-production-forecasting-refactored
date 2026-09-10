# Previsione della produzione fotovoltaica — LSTM vs GRU (Stagionale)

Previsione della produzione fotovoltaica (PV) utilizzando il deep learning su serie temporali, confrontando le architetture LSTM e GRU su orizzonti stagionali (Primavera / Estate / Autunno / Inverno) e finestre di previsione (12 h, 24 h, 48 h).

## 📋 Cosa fa questo progetto

- Carica dati meteorologici e di produzione fotovoltaica da dataset Kaggle
- Ricampiona i dati con frequenza oraria e costruisce un dataset unificato di serie temporali
- Addestra modelli LSTM e GRU per la previsione multi-step
- Valuta le prestazioni stagionalmente (Primavera, Estate, Autunno, Inverno) per il 2022
- Calcola l'RMSE per ogni variante stagionale e stima l'impatto economico
- Utilizza un generatore di sequenze lazy (WindowSequence) per un addestramento efficiente in termini di memoria


### Preparazione dei dati

Scaricare i dataset Kaggle e inserirli nella cartella kaggle_data/:

kaggle_data/
└── archive.zip    ← Deve contenere dt1_*.csv e dt2_*.csv

Il notebook estrarrà automaticamente archive.zip nella cartella kaggle_data/ alla prima esecuzione.

Opzionale: se si dispone già di un file cache pre-elaborato, inserirlo in:

kaggle_data/dataset_ready.pkl

Il notebook salterà la fase di pre-elaborazione se questa cache è presente.

## ▶️ Come eseguire il progetto

1. Aprire l'ambiente che contiene tutte le dipendenze in VS Code o Jupyter
2. Selezionare il kernel Python
3. Eseguire tutte le celle dall'alto verso il basso (Shift+Enter oppure "Run All Cells")

### Configurazione

Tutti i parametri dell'esperimento sono centralizzati nel dizionario CONFIG nella parte iniziale del notebook. Impostazioni principali:

| Parametro | Predefinito | Descrizione |
|-----------|-------------|-------------|
| target_col | "Pg" | Colonna obiettivo (produzione PV) |
| resample_rule | "h" | Ricampionamento orario |
| time_steps | [12, 24, 48] | Lunghezze delle sequenze (ore) |
| model_types | ["lstm", "gru"] | Architetture da confrontare |
| units | 32 | Unità LSTM/GRU |
| dropout | 0.20 | Tasso di dropout |
| learning_rate | 0.001 | Learning rate dell'ottimizzatore Adam |
| batch_size | 32 | Dimensione del batch di addestramento |
| epochs | 100 | Numero massimo di epoche di addestramento |
| train_ratio | 0.70 | Suddivisione per il training |
| validation_ratio | 0.15 | Suddivisione per la validazione |
| energy_price_eur_kwh | 0.15 | Prezzo utilizzato per la stima economica |

## 📊 Output

Il notebook produce:

- Metriche RMSE per modello, orizzonte temporale e variante stagionale
- Grafici delle previsioni che confrontano la produzione prevista con quella effettiva
- Grafici di confronto stagionale (Primavera / Estate / Autunno / Inverno)
- Stima dell'impatto economico basata sui prezzi dell'energia
- Selezione del modello migliore sulla base dell'RMSE più basso

## 🔬 Metodologia scientifica

- Suddivisione temporale: 70% training / 15% validazione / 15% test (cronologica, senza mescolamento)
- Scaler: MinMaxScaler adattato esclusivamente ai dati di training per evitare data leakage
- Baseline: previsione naive t-24h (valore di 24 ore prima)
- Valutazione stagionale: Primavera (mar-mag), Estate (giu-ago), Autunno (set-nov), Inverno (dic-feb)
- Sequenze lazy: WindowSequence (sottoclasse di Keras Sequence) genera le finestre dinamicamente per ridurre al minimo l'utilizzo della RAM

## ⚠️ Vincoli noti
- RAM: il generatore di sequenze lazy mantiene basso l'utilizzo della memoria, ma il caricamento dell'intero dataset richiede circa 4-8 GB di RAM.

## 🔧 Possibili miglioramenti futuri

- **Limitare l'interpolazione temporale**  
  La funzione `interpolate(method="time")` riempie qualsiasi gap, anche molto grande.  
  Se mancano giorni o settimane, produce una linea retta non realistica.  
  Possibile miglioramento: impostare un limite massimo di interpolazione (es. 3–4 ore).

- **Passare da Pickle a Parquet per i DataFrame**  
  Il formato Parquet è più efficiente, compresso e compatibile con altri strumenti rispetto a Pickle.

- **Aumentare il parametro *patience***  
  Un valore più alto potrebbe evitare minimi locali durante l’addestramento del modello.

- **Testare dropout diversi**  
  Valutare dropout più alti o più bassi per migliorare la generalizzazione del modello.


## 🤖 Utilizzo di strumenti di Intelligenza Artificiale
- Durante lo sviluppo del presente progetto sono stati utilizzati strumenti di Intelligenza Artificiale (AI) come supporto alle attività di sviluppo e ottimizzazione del codice. In particolare, tali strumenti sono stati impiegati principalmente per attività di refactoring del codice originale e per l'individuazione di strategie volte a ridurre l'utilizzo della memoria RAM.

- Le ottimizzazioni relative alla memoria si sono rese necessarie a seguito dei limiti di memoria dell'ambiente Google Colab, che hanno portato al successivo trasferimento dell'esecuzione del progetto su un ambiente locale. Tra gli interventi effettuati rientra anche l'adozione di un generatore di sequenze lazy (WindowSequence), finalizzato a limitare l'allocazione simultanea in memoria delle finestre temporali utilizzate durante l'addestramento.

- Il codice generato o modificato con il supporto di strumenti AI è stato verificato, adattato e integrato manualmente nell'ambito del progetto. Le scelte metodologiche, progettuali e sperimentali, nonché la verifica della correttezza del codice e dei risultati ottenuti, rimangono sotto la responsabilità dell'autore.

- L'utilizzo dell'AI deve pertanto essere inteso come strumento ausiliario al processo di sviluppo, e non come sostituto delle attività di analisi, comprensione, implementazione e validazione richieste per il progetto.

## 📝 Licenza

Questo progetto è destinato a scopi di ricerca/educativi. I dataset Kaggle sono soggetti alle rispettive licenze.