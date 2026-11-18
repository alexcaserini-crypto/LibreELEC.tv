# Piano di sviluppo — Kodi Now Play

Aggiornato il 18 novembre 2026. Leggere insieme a [README](README.md) e [specifiche e laboratorio](SPECIFICHE-E-LABORATORIO.md). Questo documento integra le precedenti opzioni generiche con le decisioni della conversazione. Stato: progettazione documentata, sviluppo non iniziato.

## Obiettivo e confini

Realizzare un add-on di servizio Kodi installabile, aggiornabile e reinstallabile, evitando modifiche manuali al programma. Deve integrare il Raspberry musicale con l'autoradio Bluetooth della Tesla Model Y: informazioni del brano, stato e posizione, copertine e comandi supportati. Seek assoluto e comportamento Tesla restano da verificare, non sono funzionalità già garantite.

Lo sviluppo avviene a casa sul Raspberry Kodi reale. Non simuliamo Kodi. Costruiamo un banco che svolga il ruolo dell'autoradio. La Tesla serve per brevi collaudi finali o diagnosi di differenze specifiche.

## Hardware previsto

| Elemento | Impiego | Stato |
| --- | --- | --- |
| Raspberry attuale con Kodi | Sorgente audio e lettore da integrare | Disponibile; modello, OS e versioni da rilevare |
| Secondo Raspberry Pi 3 B+ | Ricevitore e controller del banco | L'utente ritiene di averlo; confermare modello e disponibilità |
| Bluetooth integrato del 3 B+ | Prima scelta per collegamento con telefono o Kodi | Da collaudare sul sistema scelto |
| PC dell'utente | Sviluppo remoto, consultazione log e interfaccia browser | Accesso di rete da predisporre |
| Android | Sorgente di riferimento per verificare il banco prima di Kodi | Disponibile |
| microSD e alimentatore del banco | Sistema Linux dedicato | Disponibilità da verificare; non sovrascrivere schede in uso senza accordo |
| Uscita audio locale | Ascolto di verifica tramite uscita supportata, per esempio jack o HDMI | Scegliere in base all'attrezzatura disponibile |

Non sono previsti acquisti iniziali di adattatori Bluetooth, PC o display. Se il controller integrato mostra un limite concreto, valutarne la causa prima di proporre hardware aggiuntivo. Bluetooth 4.2 e AVRCP 1.6 sono versioni di livelli differenti: il numero del controller non basta a stabilire il supporto software alle copertine.

## Sistema e accesso

Proposta: Linux sul secondo Raspberry, preferibilmente una variante leggera di Raspberry Pi OS compatibile con il 3 B+. Versione, architettura e pacchetti vanno selezionati e registrati all'avvio dello sviluppo, verificando le esigenze di BlueZ, OBEX e del servizio audio. Non è stata ancora preparata un'immagine o installato alcun pacchetto.

Il banco sarà amministrabile via SSH. L'interfaccia di prova sarà una pagina web raggiungibile dal PC sulla rete locale; non richiede display collegato al Raspberry. La pagina è un componente da sviluppare, non una funzionalità già disponibile. Il browser visualizza dati e invia azioni al servizio del banco; il traffico Bluetooth resta fra i dispositivi reali. L'audio può essere riprodotto direttamente dal banco: non si richiede lo streaming dell'audio al browser.

Usare preferibilmente Ethernet per amministrazione e log, quando disponibile. L'accesso remoto consentirà di sviluppare e raccogliere evidenze senza spostare il PC in macchina.

## Ruoli del sistema

| Dispositivo | Audio | Controllo | Copertine |
| --- | --- | --- | --- |
| Android di riferimento | A2DP Source | AVRCP Target | Serve l'immagine quando supportato |
| Raspberry Kodi | A2DP Source | AVRCP Target da integrare con Kodi | Pubblicazione da implementare o integrare |
| Raspberry del banco | A2DP Sink | AVRCP Controller | Richiede, riceve e mostra l'immagine |

Android sostituisce temporaneamente Kodi come sorgente: non fa da autoradio. La prova Android verifica il nostro banco, non riproduce automaticamente le richieste Tesla.

## Componenti proposti

1. Servizi di sistema per Bluetooth, audio e trasferimento immagini, scelti dopo la verifica delle versioni e delle interfacce disponibili.
2. Servizio del banco che legge eventi e metadati, espone comandi e conserva log con orari coerenti.
3. Interfaccia browser per connessione, stato, brano, copertina e comandi.
4. Add-on Kodi di servizio, identificativo proposto `service.kodi.nowplay`, con impostazioni e diagnostica proprie.
5. Eventuale componente di sistema sul Raspberry Kodi, solo se necessario per bus, dipendenze native o copertine. La necessità va accertata: un add-on Python da solo non è ancora dimostrato sufficiente.

Non scegliere librerie o inventare API prima di verificarle sulla piattaforma reale. Tenere distinta la ricezione delle immagini sul banco dalla loro pubblicazione su Kodi.

## Interfaccia del banco: requisiti

- Identità e stato del dispositivo collegato; stato dei servizi e capacità rilevate.
- Titolo, artista, album, durata, posizione e stato di riproduzione, con gestione dei campi assenti.
- Copertina ricevuta via Bluetooth, con indicazione di assenza o errore. Non recuperarla online per mascherare un trasferimento mancante.
- Play, pausa, precedente e successivo; avanzamento e riavvolgimento se supportati.
- Barra della posizione inizialmente informativa. Abilitare il trascinamento solo dopo aver verificato un percorso di comando realmente funzionante fino al lettore.
- Registro degli eventi che distingua richiesta inviata, risposta ricevuta e cambiamento effettivamente osservato.
- Funzione per esportare evidenze pertinenti di un test; limitare accesso e contenuti alla rete e allo scopo del laboratorio.

Una barra che si muove soltanto nel browser non dimostra che Kodi abbia eseguito un seek.

## Fasi e criteri per proseguire

| Fase | Attività | Criterio di completamento |
| --- | --- | --- |
| 0 — Inventario | Confermare hardware, OS, versioni e script precedente | Inventario riproducibile e materiali acquisiti |
| 1 — Ricevitore | Collegare Android, ricevere e ascoltare audio | Audio reale verificato e riconnessione funzionante |
| 2 — Metadati e controlli | Visualizzare dati e inviare comandi semplici | Valori coerenti e azioni confermate sul telefono |
| 3 — Copertine | Verificare capacità e trasferimento BIP/OBEX | Due immagini diverse ricevute e aggiornate correttamente |
| 4 — Kodi | Collegare il Raspberry reale e analizzare lo script | Differenze rispetto ad Android identificate |
| 5 — Add-on | Implementare ponte, configurazione, log e dipendenze | Pacchetto installabile e funzionalità collaudate sul banco |
| 6 — Robustezza | Disconnessione, riavvio, cambio brano rapido e dati mancanti | Nessun blocco o dato precedente persistente impropriamente |
| 7 — Tesla | Breve verifica sulla sorgente Bluetooth dell'auto | Risultati per funzione registrati, inclusi limiti dello slider |

Non rinviare la verifica del supporto Cover Art a fine sviluppo: è un rischio tecnico da chiarire presto. Se un componente non lo supporta, documentare il limite e decidere come integrarlo prima di promettere il risultato.

## Materiale di prova

Preparare due brani facilmente distinguibili con metadati e copertine diverse, un brano senza copertina e contenuti di durata nota. Annotare risultati sul telefono, sul banco e su Kodi usando gli stessi casi quando possibile. Registrare valori di posizione e relative unità, senza confondere millisecondi delle API Bluetooth con microsecondi di MPRIS.

La cattura Android HCI non è un prerequisito per implementare lo standard. Può essere usata per il confronto con Tesla se necessario. L'app nRF non sostituisce la cattura HCI dei profili Bluetooth Classic. Non si prevede un ESP32 esterno come sniffer della connessione cifrata telefono–Tesla.

## Documentazione ed evidenze

Per ogni sessione registrare versioni, configurazione, passi eseguiti, comportamento atteso, comportamento osservato e file di log pertinenti. La sola risposta positiva a un comando non basta: controllare anche l'effetto nel lettore.

Il repository LibreELEC.tv è pubblico. Conservare qui configurazioni di esempio ed estratti ripuliti; non pubblicare bug report completi, segreti o identificativi non necessari. Collegare i documenti alla versione del codice verificata quando inizierà lo sviluppo.

## Aggiornamenti e distribuzione

L'add-on deve poter essere installato da ZIP su un sistema nuovo. Per gli aggiornamenti compatibili verificare conservazione di impostazioni e funzionamento; non assumere compatibilità universale con nuove versioni Kodi o del sistema operativo. Documentare separatamente eventuali prerequisiti esterni e il loro ripristino, perché il solo ZIP potrebbe non installarli.

## Punto di ripartenza

Quando l'utente avvia lo sviluppo: confermare che il secondo dispositivo sia un 3 B+, verificare microSD e alimentatore disponibili, identificare il sistema del Raspberry Kodi e acquisire lo script precedente. Poi scegliere il sistema del banco e iniziare dalla fase 1. L'autorizzazione attuale riguarda la documentazione del progetto; non è stata eseguita alcuna installazione o formattazione.
