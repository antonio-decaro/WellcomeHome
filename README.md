
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
![Architettura](/architettura.png)
