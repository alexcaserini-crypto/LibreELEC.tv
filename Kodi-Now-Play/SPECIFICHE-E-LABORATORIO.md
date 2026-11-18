# Specifiche Bluetooth e laboratorio Kodi Now Play

Data: 18 novembre 2026. Questo documento accompagna il [README](README.md). Contiene riferimenti verificati e un progetto di laboratorio: non certifica funzionalità già implementate o prove già eseguite.

## 1. Decisioni e contesto

Kodi resta sul Raspberry reale. Il banco deve svolgere il ruolo dell'autoradio. Il software finale sarà un add-on di servizio installabile, con eventuali dipendenze di sistema da identificare. Non si vuole mantenere una modifica manuale ai file di Kodi.

Il banco può essere un PC Linux o un secondo Raspberry con Linux. La VM non è un requisito. Se si usa una VM, l'adattatore USB deve essere assegnato alla macchina ospite. Se si usa un secondo Raspberry, verificare modello, supporto Bluetooth Classic e sistema operativo prima di scegliere pacchetti e configurazione. Non è necessario acquistare un PC per questo scopo. Non confondere i due Raspberry: uno produce la musica, l'altro la riceve e controlla il lettore.

L'utente può collegare un telefono Android come sorgente di riferimento al banco. Non sta chiedendo al telefono di ricevere la musica come un'autoradio. Successivamente si sostituisce Android con il Raspberry Kodi mantenendo invariato il ricevitore.

## 2. Fonti ufficiali e indice di ricerca

| Fonte | Collegamento | Utilità |
| --- | --- | --- |
| Bluetooth SIG, AVRCP 1.6.3, 2024-10-01 | https://files.bluetooth.com/wp-content/uploads/dlm_uploads/2024/10/AVRCP_v1.6.3.pdf | Riferimento normativo per controllo e copertine |
| Bluetooth SIG, catalogo specifiche | https://www.bluetooth.com/specifications/specs/ | Ricerca versioni ed eventuali aggiornamenti |
| Bluetooth SIG, A2DP | https://www.bluetooth.com/specifications/specs/advanced-audio-distribution-profile/ | Profilo per distribuzione audio |
| BlueZ, Media API | https://bluez.readthedocs.io/en/latest/media-api/ | Collegamento tra applicazioni Linux e stack Bluetooth |
| BlueZ, sorgenti MediaPlayer | https://github.com/bluez/bluez/blob/master/doc/org.bluez.MediaPlayer.rst | Confronto con documentazione della versione installata |
| MPRIS, Player | https://specifications.freedesktop.org/mpris/latest/Player_Interface.html | Interfaccia locale del lettore, distinta dal protocollo radio |
| Kodi, servizi | https://kodi.wiki/view/Service_add-ons | Ciclo di vita dell'add-on |
| Kodi, struttura | https://kodi.wiki/view/Add-on_structure | Organizzazione del pacchetto |
| Android, Bluetooth automotive | https://source.android.com/docs/automotive/ivi_connectivity | Ruoli telefono e infotainment |
| Android, verifica e debug | https://source.android.com/docs/core/connect/bluetooth/verifying_debugging | Acquisizione log HCI e bug report |

I collegamenti sono i riferimenti da consultare integralmente; questo documento non riproduce le specifiche. Una pagina online corrente non descrive necessariamente il software installato sul dispositivo.

## 3. Protocollo e ruoli

A2DP riguarda il trasporto audio; AVRCP riguarda controllo e informazioni del lettore. Nel nostro scenario, il lettore è A2DP Source e AVRCP Target; il banco è A2DP Sink e AVRCP Controller. I ruoli non vanno dedotti da chi ha avviato l'associazione.

Per AVRCP 1.6.3 consultare capitolo 2 per i ruoli, capitoli 3 e 4 per funzioni e procedure, sezione 5.14 per Cover Art, capitoli 13 e 14 per BIP/OBEX e interoperabilità. La copertina usa un handle nei metadati e il trasferimento BIP su OBEX. Le operazioni includono GetImageProperties, GetImage e GetLinkedThumbnail. Il controller è Imaging Initiator, il lettore Imaging Responder. La miniatura comune è JPEG 200 × 200; per i dettagli di formato consultare 5.14.2.2.1. La validità degli handle dipende dalla connessione e dagli elementi associati: vedere 5.14.1 e 5.14.2.2.2.

Questa descrizione non implica che ogni dispositivo che dichiara AVRCP implementi Cover Art. Verificare le capacità e il comportamento effettivi.

## 4. Confine tra standard, BlueZ e Kodi

BlueZ documenta RegisterPlayer su Media1 per registrare un lettore locale con interfaccia MPRIS. Sul controller, MediaPlayer1 espone metodi come Play, Pause, Next, Previous, FastForward e Rewind e proprietà di traccia, stato e posizione. La posizione documentata è in millisecondi ed è di sola lettura su questa interfaccia. Non confondere l'oggetto remoto MediaPlayer1 con l'interfaccia implementata dal lettore locale.

MPRIS definisce anche Seek, SetPosition e CanSeek, con tempi in microsecondi. Queste API locali non dimostrano che una funzione equivalente venga trasportata da AVRCP o utilizzata dalla Tesla. Prima di scrivere il ponte, definire conversioni di unità, associazione della traccia e risposta quando un'operazione non è supportata.

Un percorso file o un URL presente nei metadati locali non equivale a una copertina trasferita via Bluetooth. Nel progetto verificare separatamente il lato che serve l'immagine e quello che la richiede. Il supporto alla ricezione non dimostra supporto alla pubblicazione. La scelta della versione BlueZ e di OBEX resta aperta: non è stato verificato il sistema dell'utente.

Kodi documenta add-on di servizio avviati con Kodi o all'accesso del profilo e arrestati all'uscita. Per il progetto si propone `service.kodi.nowplay`, distribuito come ZIP. Registrazione Bluetooth, accesso al bus di sistema, librerie native ed eventuali componenti esterni devono essere verificati sulla piattaforma reale prima di promettere un pacchetto autosufficiente.

## 5. Piano operativo del banco

Questa è una proposta di implementazione, non una sequenza normativa completa di pacchetti.

1. Rilevare hardware, OS, kernel, Kodi, BlueZ, servizio audio e OBEX sui due dispositivi. Conservare l'inventario insieme ai risultati dei test.
2. Configurare il banco come ricevitore audio e controller, verificando che il sistema non lo presenti soltanto come sorgente.
3. Collegare Android, riprodurre un brano noto e verificare l'uscita audio locale.
4. Mostrare in una piccola interfaccia i dati effettivamente ricevuti, distinguendoli da valori calcolati localmente.
5. Provare comandi semplici, avanzamento e ritorno alla riproduzione normale; registrare richiesta, risposta ed effetto osservato.
6. Aggiungere la ricezione delle copertine. Salvare evidenza della richiesta Bluetooth e dell'immagine ricevuta, senza considerare sufficiente una copertina recuperata da Internet.
7. Sostituire Android con Kodi. Usare gli stessi contenuti di prova e confrontare i risultati.
8. Implementare l'add-on dopo aver identificato i componenti mancanti; collaudarlo sul banco prima della Tesla.

Il ricevitore dovrà essere attivo: limitarsi ad ascoltare le trasmissioni spontanee del telefono non verifica le risposte alle richieste che una vera autoradio potrebbe inviare.

## 6. Posizione, avanzamento e slider

Tenere separati tre risultati: mostrare la posizione, avanzare/riavvolgere e saltare a una posizione assoluta. Non aggiungere uno slider apparentemente funzionante che aggiorni soltanto la grafica del banco. Per ogni azione osservare la posizione reale in Kodi e verificare che anche il suono cambi come atteso.

Prima di concludere che Tesla supporta il salto assoluto via Bluetooth, ripetere la prova sulla sorgente Bluetooth del telefono. L'utente ricorda una probabile funzione con YouTube; VLC resta incerto. Il ricordo non costituisce un collaudo. Documentare anche il caso in cui la barra sia soltanto informativa.

## 7. Matrice di verifica proposta

| Funzione | Prova | Evidenza da conservare | Stato iniziale |
| --- | --- | --- | --- |
| Associazione | Collegamento da zero e riconnessione | Versioni, capacità, esito | Da eseguire |
| Audio | Android, poi Kodi | Codec negoziato, ascolto, interruzioni | Da eseguire sul banco |
| Metadati | Cambio fra due brani diversi | Valori ricevuti e orari | Da eseguire |
| Stato | Play, pausa, stop | Comando, risposta, stato reale | Da eseguire |
| Posizione | Riproduzione e pausa | Valori ricevuti, unità e scostamento | Da eseguire |
| Avanzamento | Pressione e rilascio, se esposti | Sequenza ed effetto reale | Da eseguire |
| Seek assoluto | Verifica disponibilità prima della prova | Percorso completo fino a Kodi o limite accertato | Non confermato |
| Copertina | Due immagini chiaramente diverse | Handle, richieste, immagine ricevuta | Da eseguire |
| Immagine assente | Brano senza copertina | Nessuna immagine precedente rimasta | Da eseguire |
| Recupero | Spegnimento Bluetooth e riavvio Kodi | Riconnessione senza blocchi | Da eseguire |
| Distribuzione | Installazione e aggiornamento add-on | Versioni e impostazioni conservate | Da eseguire |
| Tesla | Breve prova finale | Modello, software auto, sorgente e risultati | Da eseguire |

## 8. Registrazione e diagnosi

Android documenta la registrazione Bluetooth HCI nelle opzioni sviluppatore e l'estrazione dei log tramite bug report; disponibilità e contenuto effettivi devono essere verificati sul telefono. Non dare per garantito un percorso universale del file. Registrare la connessione dall'inizio e annotare gli orari delle azioni.

Per il banco Linux si propone di raccogliere la cattura HCI insieme ai log del servizio, dell'interfaccia di controllo e di Kodi. Ogni evento deve identificare origine, destinazione, traccia e risultato. Non archiviare un bug report completo in questo repository pubblico: produrre estratti pertinenti e rimuovere credenziali, identificativi e contenuti estranei prima di pubblicarli.

I log non sono un prerequisito per conoscere il protocollo standard. Servono a diagnosticare le implementazioni. Il telefono verso il banco verifica il nostro ricevitore; non rivela le richieste specifiche della Tesla. Un breve log telefono–Tesla sarà utile solo quando necessario per risolvere una differenza reale.

## 9. Informazioni mancanti e prossima decisione

Acquisire lo script precedente; identificare il sistema Kodi; scegliere il secondo Raspberry o PC già disponibile; verificare il Bluetooth Classic; selezionare le versioni dei componenti e provarne prima i ruoli di ricezione e controllo. Il modello del Raspberry del banco va chiesto prima di indicare un'immagine o un adattatore da acquistare.

Al momento non sono stati installati ambienti, implementati emulatori o eseguiti test radio. Sono stati creati i documenti di progetto. Il successo del banco generico e quello sulla Tesla dovranno restare due risultati distinti nel registro delle prove.
