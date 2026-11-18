# Kodi Now Play

Documento iniziale — 18 novembre 2026. Stato: requisiti e piano di verifica; nessun add-on o emulatore implementato o collaudato.

## Obiettivo

Realizzare un add-on di servizio Kodi installabile e versionato che integri il lettore con un'autoradio Bluetooth, evitando modifiche manuali ai file di Kodi. Prevedere installazione da ZIP, impostazioni e log propri. Verificare conservazione negli aggiornamenti compatibili e reinstallazione su un sistema nuovo: la compatibilità futura non è garantita automaticamente.

## Scenario reale e informazioni disponibili

- Sorgente: Raspberry Pi reale con Kodi e musica locale; audio inviato via Bluetooth alla Tesla Model Y.
- Esiste uno script precedente, disponibile all'utente, con alcuni bug: da acquisire e analizzare, senza assumerne il contenuto.
- La soluzione attuale mostra titolo, artista e avanzamento; le copertine non funzionano e lo spostamento della posizione non è stato ottenuto.
- L'utente riferisce copertine visibili dal telefono sulla Tesla. Lo spostamento dello slider con YouTube è ricordato come probabile; con VLC resta incerto. Non sono prove ripetute durante questa sessione.
- Sistema operativo e versioni di Kodi e BlueZ sul Raspberry ancora da identificare. L'esistenza del repository LibreELEC.tv non prova che il dispositivo usi LibreELEC.

## Vincolo principale: sviluppo a casa

Non simulare Kodi: usare il Raspberry reale. Preparare invece un ricevitore di prova che svolga il ruolo dell'autoradio. L'utente non deve sviluppare in macchina. Riservare alla Tesla brevi prove finali di interoperabilità.

## Prima fase: laboratorio autoradio Bluetooth

Proposta da verificare: PC con Linux, eventualmente in macchina virtuale, adattatore Bluetooth USB dedicato assegnato direttamente alla VM. Serve supporto Bluetooth Classic, non soltanto BLE. Il ricevitore dovrà fornire:

1. Ricezione audio A2DP dal Raspberry e riproduzione locale.
2. Ruolo AVRCP Controller, lettura di metadati, durata, posizione e stato.
3. Invio di play, pausa, precedente, successivo e avanzamento/riavvolgimento supportati.
4. Ricezione e visualizzazione delle copertine tramite AVRCP Cover Art/BIP over OBEX, se supportato dalle versioni scelte.
5. Interfaccia di prova e acquisizione di log Bluetooth e comandi, per distinguere trasmissione, ricezione e applicazione dei comandi.

Non presumere che un normale ricevitore audio o un'interfaccia grafica Bluetooth implementi tutte queste funzioni. Verificare separatamente disponibilità e limiti dei componenti. Non è stato individuato o collaudato un emulatore Tesla completo.

Per provare il protocollo via radio serviranno due estremità reali: Bluetooth del Raspberry e adattatore del ricevitore. Durante la verifica iniziale del ricevitore si potrà usare un telefono come sorgente di riferimento, senza confondere i suoi risultati con quelli di Kodi.

## Seconda fase: architettura add-on

- Add-on Kodi di tipo servizio, identificativo proposto `service.kodi.nowplay`.
- Lettura eventi, metadati, copertina e posizione da Kodi; inoltro dei comandi ricevuti al lettore.
- Integrazione con lo stack Bluetooth del sistema da progettare dopo aver identificato piattaforma e versioni.
- Verificare se servano componenti di sistema esterni, dipendenze native, permessi o modifiche a BlueZ/OBEX: un add-on Python da solo potrebbe non essere sufficiente, in particolare per servire le copertine.
- Riconnessione, gestione cambio brano, pausa, aggiornamento posizione e log diagnostici.
- Lo script precedente costituisce materiale di analisi, non una modifica da applicare senza revisione.

## Slider: distinzione necessaria

Visualizzare la posizione, avanzare tenendo premuto un comando e saltare a un punto assoluto trascinando lo slider sono funzioni diverse. Registrare i comandi effettivamente emessi dal controller; non presumere che AVRCP o l'autoradio offrano un seek assoluto arbitrario. La compatibilità dello slider Tesla rimane da verificare.

## Criteri di verifica

| Area | Verifica richiesta |
| --- | --- |
| Audio | Ricezione stabile dal Raspberry sul banco |
| Metadati | Titolo, artista, durata e stato corretti al cambio brano |
| Posizione | Aggiornamento coerente durante riproduzione, pausa e avanzamento |
| Comandi | Ogni comando ricevuto è registrato e produce l'azione attesa in Kodi |
| Copertine | Immagine trasferita realmente via Bluetooth, visualizzata e aggiornata al cambio brano |
| Robustezza | Riconnessione dopo disconnessione e riavvio, senza dati obsoleti |
| Distribuzione | ZIP installabile, impostazioni persistenti negli aggiornamenti compatibili e reinstallazione documentata |
| Tesla | Prove finali sull'auto; il banco generico non garantisce comportamento identico |

## Prossimi passi

1. Acquisire script esistente e identificare OS, Kodi e stack Bluetooth del Raspberry.
2. Identificare PC, sistema host, disponibilità di VM e adattatori Bluetooth.
3. Selezionare e collaudare il ricevitore: prima audio e AVRCP, poi copertine e comandi di avanzamento.
4. Stabilire le dipendenze installabili sulla piattaforma reale e i limiti dell'add-on.
5. Implementare e verificare l'add-on sul banco; raccogliere un breve collaudo finale Tesla.

## Riferimenti tecnici iniziali

- [Kodi: Service add-ons](https://kodi.wiki/view/Service_add-ons)
- [Kodi: Add-on structure](https://kodi.wiki/view/Add-on_structure)
- [BlueZ: Media API](https://bluez.readthedocs.io/en/latest/media-api/)
- [Bluetooth SIG: AVRCP 1.6.3](https://files.bluetooth.com/wp-content/uploads/dlm_uploads/2024/10/AVRCP_v1.6.3.pdf)

I riferimenti descrivono le interfacce e il protocollo; non certificano che le versioni installate implementino tutte le funzioni richieste.
