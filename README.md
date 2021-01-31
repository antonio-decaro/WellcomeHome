
# WellcomeHome
WellcomeHome è un software che nasce con l'obbiettivo di ottenere un maggior controllo all'interno di un contesto privato, come ad esempio un appartamento, un ufficio o più generalmente un qualsiasi luogo non aperto al pubblico.
## Panoramica

Grazie alle tecnologie offerte dal Cloud, WellcomeHome è stato progettato con lo scopo di ridurre al minimo le risorse utilizzate on-premise dal dispositivo di videosorveglianza. La bassa complessità computazionale che preme sul dispositivo di hosting della camera permette infatti di impiegare un qualunque tipo di hardware: a partire da un semplice Raspberry fino ad arrivare a dispositivi più complessi come un Jetson Nano.

------------

L'utente interagisce con WellcomeHome attraverso l'utilizzo di un bot presente sul canale di comunicazione Telegram (**@wellcomehome_bot**). 

Ad un primo accesso, il bot chiederà all'utente di accedere al proprio account di GitHub in modo da poter registrare la sua identità nel dominio dell'applicazione.
 
Ogni utente avrà il suo ambiente, da personalizzare tramite l'inserimento di profili da far riconoscere a WellcomeHome. Tramite questo bot, l'utente sarà in grado di:
- aggiungere nuove identità all'interno dell'applicazione;
- rimuovere identità già presenti all'interno dell'applicazione;
- aggiornare identità già presenti all'interno dell'applicazione;
- testare il funzionamento dell'applicazione dal bot stesso;
- controllare i volti identificati dal dispositivo di videosorveglianza;


## Servizi Azure Utilizzati
WellcomeHome sfrutta diversi servizi servizi offerti da Azure.
La tabella seguente riassume e motiva tutti i servizi in Cloud utilizzati da WellcomeHome.
| # | Servizio | Ruolo |
|-|-|-|
|1|**Functions**|Sfruttano la capacità di computazione serverless di Azure per poter identificare quando necessario i volti presenti all'interno di un frame catturato dal dispositivio di videosorveglianza.| 
|2|**Cognitive Services - Face Recognition**|Wellcome Home sfrutta la Face Recognition dei Cognitive Services per identificare i volti presenti in un immagine. La Face Recognition viene utilizzata all'interno delle Functions.|
|3|**App Service**|Questo servizio viene impiegato (insieme al Bot Channel Registration) per poter hostare il bot in Cloud.|
|4|**Blob Storage**|Lo storage viene sfruttato dal dispositivo di videosorveglianza per poter effettuare l'upload di immagini, in modo da invocare l'esecuzione di una Function.|
|5|**Cosmos DB**|Il Cosmos DB immagazzina informazioni circa gli utenti che si registrano all'applicazione WellcomeHome ed informazioni circa le varie identità inserite dai diversi utenti. |
|6|**Key Vault**|Viene sfruttato sia dal Bot che dalle Functions per poter accedere ad informazioni sensibili come password, chiavi, etc.|
|7|**Bot Channel Registration**|In sinergia con App Service, il servizio di Bot Channel Registration viene impiegato per effettuare il binding del bot al canale di comunicazione Telegram.|

## Architettura
Gli attori principali dell'applicazione sono l'utente e la IoT Camera. L'interfaccia utente è definita dal bot, hostato attraverso l'utilizzo di un App Service e di un Bot Channel Registration.

La comunicazione tra l'utente ed il bot avviene grazie al canale di comunicazione telegram. Per utilizzare l'applicazione, l'utente deve necessariamente effettuare l'accesso al suo account di GitHub in modo da essere riconosciuto nel dominio dell'applicazione. L'autenticazione avviene grazie al provider OAuth di GitHub. 

La IoT camera si occupa di effettuare l'upload di frame all'interno di un Blob Storage tramite una stringa di connessione dotata di firma di accesso condiviso (SAS).

Quando viene effettuato l'upload di un immagine all'interno del Blob Storage viene eseguita la funzione FaceDetection che si avvale della Face Recognition di Azure per identificare il volto presente nell'immagine.

L'Azure Cosmos DB contiene informazioni circa gli utenti registrati all'applicazione e le corrispettive identità dei volti inseriti.

La funzione ManagePeople effettua operazioni di inserimento o aggiornamento di identità all'interno del database.

Infine sono presenti altre due funzioni StorageCleaner e CognitiveServiceCleaner che vengono eseguite rispettivamente ogni 15 minuti ed una volta a settimana. La prima elimina dal Blob Storage tutte le immagini caricate dalla camera che non sono state utilizzate nei 15 minuti precedenti, la seconda funzione invece serve per mantenere consistenti le informazioni che persistono nei Cognitive Services con quelle presenti nel database.

![Architettura](/architettura.png)
*ps. Sia le Functions che il Bot comunicano con il servizio di Azure Key Vault per accedere alle informazioni sensibili.*

## Installazione e guida all'utilizzo
All'interno di questa repository è presente un file chiamato raspberry.py che contiene uno script realizzato in python che deve essere eseguito all'interno della macchina host della camera.
I test sono stati eseguiti con un Raspberry Pi 4, al quale è stata collegata una webcam ordinaria.

------------

### Installazione
Di seguito vengono riportati i passaggi da effettuare per eseguire lo script su un Raspberry Pi 4.

#### 0. Effettuare il primo accesso al bot
Collegarsi al bot da Telegram (**@wellcomehome_bot**) ed effettuare l'accesso con il proprio account GitHub seguendo le istruzioni del bot.
Memorizzare il proprio **ID** inviando al bot un messaggio contenente il testo "id" o cliccando l'apposito pulsante consigliato dal bot.

#### 1. Aggiornare il raspberry
Accedere al Raspberry e digitare i seguenti comandi:

    $ sudo apt-get update
    $ sudo apt-get upgrade
   
#### 2. Installare Python
Se si dispone già di una versione di python 3.8 o superiore è possibile saltare questo passaggio, altrimenti digitare i seguenti comandi:

    $ sudo apt-get install python3
    $ sudo apt-get install pip3

#### 3. Inizializzare l'applicazione
Scaricare la cartella **WellcomeHome_Raspberry** presente all'interno del repository.

Dall'interno della cartella installare le dipendenze dell'applicazione eseguire il comando:

    $ pip3 install -r requirements.txt

A questo punto è necessario copiare il file **/data/haarcascade_frontalface_default.xml** all'interno del percorso di installazione del pacchetto di OpenCV **[percorso installazione python]/site-packages/cv2/data** installato precedentemente. Nel mio caso il pacchetto è stato installato al percorso */usr/local/lib/python3.8/site-packages/cv2* 

#### 4. Avviare l'applicazione
Collegare una webcam al raspberry ed eseguire il comando:

    $ python3 script.py <ID dello step 0> <percorso installazione python>/site-packages/cv2/data/haarcascade_frontalface_default.xml

Ora WellcomeHome è pronto!

------------

### Guida all'utilizzo

Ora che l'applicazione è in funzione è sufficiente comprendere quali sono le funzionalità principali offerte dal bot.

#### Rilevamento volti
Nel momento in cui la camera di sorveglianza rileva un volto, il bot ci invierà uno dei seguenti messaggi:

> [Nome] [Cognome] è tornato a casa.

nel caso in cui l'applicazione sia riuscita correttamente ad identificare il volto, oppure

> Qualcuno è tornato a casa ma non so chi sia.
> [immagine con volto evidenziato]

nel caso contrario.

#### Inserimento di una nuova identità
L'utente può insegnare nuove identità al bot con l'apposito comando "Inserisci Persona".
Il bot chiederà all'utente di inviargli un immagine (contenente solo il volto della persona che si vuole far riconoscere al bot).

A questo punto l'utente può decidere o di creare un'identità da zero digitando o cliccando su "Nuovo", oppure di aggiornare un profilo già presente nel database  cliccando su "Esistente".

Nel primo caso il bot chiederà di inserire il nome ed il cognome della persona da aggiungere al database.
Nel secondo caso invece il bot mostrerà l'elenco delle persone già presenti nel database, l'utente dovrà solo cliccare sul pulsante contenente il nome ed il cognome della persona di cui si vuole aggiornare l'identità. 

#### Eliminare un'identità
L'utente ha anche la possibilità di far dimenticare al bot un'identità già conosciuta cliccando sul pulsante "Cancella Persona". Il bot mostrerà l'elenco delle persone presenti nel database.

#### Testare il funzionamento dell'applicazione
Tramite il comando "Test" l'utente può testare il funzionamento dell'applicazione inviando al bot un immagine contenente uno o più volti da identificare, a prescindere dal funzionamento della camera di sorveglianza.

#### Comandi di utility
Con il comando "Logout" l'utente può scollegare il suo account GitHub dall'applicazione.
Tramite il comando "Cancel" l'utente può annullare qualunque operazione in corso.
Infine il comando "Aiuto" è possibile visualizzare un messaggio che aiuta l'utente con la sua esperienza con il bot.

## Link Utili
Ai seguenti link è possibile consultare il codice del bot e delle Azure Functions.

 - https://github.com/antonio-decaro/WellcomeHome-Bot
 - https://github.com/antonio-decaro/WellcomeHome-Functions

In caso di errori è possibile contattare l'autore al seguente indirizzo email: antonio.decaro99@outlook.it.
