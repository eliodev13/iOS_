# SnapComics — Fumetti Accessibili per Non Vedenti

SnapComics è un'app iOS sviluppata in Swift che permette alle persone non vedenti e ipovedenti di fruire dei fumetti in modo autonomo, combinando riconoscimento ottico dei caratteri (OCR), riconoscimento dei personaggi tramite intelligenza artificiale e sintesi vocale.

L'app è nata da un'esigenza reale: il progetto è stato sviluppato raccogliendo il feedback diretto di un'associazione per non vedenti a Napoli, che ha contribuito a definire le priorità di accessibilità e il flusso d'uso.

---

## Il problema

I fumetti sono un medium visivo per eccellenza: i testi sono incorporati nelle vignette, i personaggi si distinguono graficamente, e la narrazione dipende dalla combinazione di immagini e parole. Per una persona non vedente, un fumetto tradizionale è inaccessibile — non basta una semplice lettura del testo, serve anche capire *chi* sta parlando e in quale contesto visivo.

SnapComics risolve questo problema rendendo i fumetti "ascoltabili".

---

## Come funziona

### 1. Acquisizione dell'immagine

L'utente può inserire le pagine del fumetto in due modi:

- **Fotocamera** — scattando una foto direttamente alla pagina del fumetto fisico
- **Libreria foto** — importando immagini già presenti sul dispositivo, con supporto alla selezione multipla

Le pagine vengono organizzate in **album**, che rappresentano i singoli fumetti o volumi.

### 2. Riconoscimento dei personaggi (CoreML + Vision)

Quando viene acquisita un'immagine, l'app attiva una pipeline di rilevamento oggetti basata su un modello CoreML addestrato su personaggi dei fumetti Disney (Topolino, Paperino, Minnie, Pippo e altri).

Il modello — chiamato `Disney2` — analizza la vignetta e identifica quale personaggio è presente, dopodiché **annuncia vocalmente il nome del personaggio** prima di leggere il testo. In questo modo l'utente capisce immediatamente chi sta parlando.

Il modello è stato addestrato con **Create ML** su dataset personalizzati, con iterazioni successive (`MyObjectDetector`, `Detector_1`, `MLFinale`) per migliorare la precisione.

### 3. OCR — Lettura del testo (Vision Framework)

Dopo il riconoscimento del personaggio, l'app esegue il riconoscimento ottico dei caratteri usando il framework **Vision** di Apple:

- `VNRecognizeTextRequest` con livello di accuratezza massimo
- Correzione linguistica attivata
- Lingua configurata su **italiano (it-IT)**
- Le osservazioni testuali vengono ordinate per posizione verticale per rispettare l'ordine di lettura naturale delle vignette

Il testo riconosciuto viene quindi passato alla sintesi vocale.

### 4. Sintesi vocale (AVFoundation)

Il testo estratto viene letto ad alta voce tramite `AVSpeechSynthesizer`:

- Voce italiana nativa del sistema
- Velocità di lettura ridotta (0.25×–0.50×) per favorire la comprensione
- Controlli manuali: **Play / Pausa / Ripeti / Immagine precedente / Immagine successiva**
- La navigazione tra le pagine avvia automaticamente la lettura della nuova pagina

---

## Struttura dell'app

```
Home (ContentView)
├── Fotocamera → rilevamento personaggio + OCR + lettura vocale
└── Libreria (LibraryView)
    ├── "In lettura" — fumetto corrente
    └── "Pronti da leggere" — album salvati
        └── AlbumView — griglia pagine + controlli audio
            └── Lettura sequenziale con Play/Pausa/Ripeti
```

---

## Tecnologie utilizzate

| Componente | Tecnologia |
|---|---|
| Interfaccia utente | SwiftUI |
| Riconoscimento personaggi | CoreML (`Disney2.mlmodel`) |
| OCR | Apple Vision Framework (`VNRecognizeTextRequest`) |
| Sintesi vocale | AVFoundation (`AVSpeechSynthesizer`) |
| Selezione foto | PhotosUI (`PHPickerViewController`) |
| Fotocamera | UIKit (`UIImagePickerController`) |
| Persistenza dati | `UserDefaults` + storage locale su disco |
| Training modelli | Create ML |

---

## Accessibilità

L'interfaccia è progettata attorno all'audio come canale primario di interazione:

- Tutti i contenuti rilevanti vengono letti automaticamente senza azioni aggiuntive da parte dell'utente
- I pulsanti di navigazione sono grandi (40×40 pt), circolari e ben distanziati per facilitare l'uso tattile
- L'immagine correntemente selezionata è evidenziata visivamente (opacità piena vs. 0.6 per le altre) — utile anche per gli ipovedenti
- Nessun testo è necessario per navigare: l'intera esperienza è guidata dall'audio

---

## Origine del progetto

Il progetto è stato sviluppato con l'obiettivo di rendere i fumetti accessibili a chi non può vederli. Per definire le priorità e validare le scelte di design, il team ha coinvolto direttamente **un'associazione per non vedenti a Napoli**, raccogliendone il feedback sul flusso d'uso, sulla velocità di lettura e sulla chiarezza delle informazioni fornite vocalmente.

Questo approccio ha portato, tra le altre cose, a:

- Scegliere di annunciare prima il personaggio e poi il testo (e non solo il testo)
- Ridurre la velocità di lettura predefinita rispetto ai valori standard
- Mantenere i controlli di navigazione semplici e sempre visibili


## Linguaggio e localizzazione

L'app è attualmente configurata per la lingua **italiana** (`it-IT`) sia per l'OCR che per la sintesi vocale.
