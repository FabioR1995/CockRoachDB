# CockRoachDB
Informazioni riguardanti CockRoachDB

# ARCHITETTURA
- Una distribuzione di CockRoachDB consiste in uno o piu processi server del database.
- I nodi di un cluster CockRoachDB sono simmetrici, cioè non esiste un nodo "primario" o "speciale".
- Il protocollo attraverso il quale avviene la comunicazione fra il client e server è il **wire protocol di PostgreSQL**.
  - Questo protocollo descrive come avvengono le richieste/risposte SQL che sono trasmesse fra il PostgreSQL client/server.
    - Ogni driver di PostgreSQL può essere utilizzato per comunicare con CockRoachDB.
- In distribuzioni più complesse, uno o piu processi di bilanciamento di carico, si occupano di distribuire le connessioni ai vari nodi.
- Il nodo leasholder (gestore), sarà responsabile del controllo delle letture e scritture su un dato range di chiavi-valori. 
- **rappresentazione di una architettura**:
  - Un client si connette ad un load balancer.
  - Il load balancer si occupa di indirizzare la richiesta ad un nodo disponibile (2).
  - Questo nodo diventa il **nodo gateway** per questa connessione.
  - La richiesta richiede i dati nel **range 4**.
  - Il nodo gateway comunica comunica con il nodo leaseholder per questo intervallo (3).
  - Il nodo leaseholder ritornerà i dati al nodo gateway (3).
  - Il nodo gateway ritornerà i dati al client.
  - <img src="https://github.com/FabioR1995/CockRoachDB/blob/main/Immagini/cluster_architecture.png" width="400" height="350">
- Sotto il cofano, i dati della tabelle di CockRoachDB sono organizzate come coppie K-V all' interno del sistema di memorizzazione.
- **CockRoachDB SQL Layer**
  - <img src="https://github.com/FabioR1995/CockRoachDB/blob/main/Immagini/stack_cockroach.png" width="450" height="350">
  - Lo strato SQL è responsabile di gestire le richieste SQL e riceve le richieste attraverso il **Postgres wire protocol**. Per l' esattezza fa il parsing (verifica la accuratezza sintaticca e i permessi necessari per eseguire un tipo di azione), ottimizza (viene creato un piano di esecuzione che poi viene ottimizzato) e mappa le richieste SQL in richieste K-V per il sottosistema.
- **Tabelle rappresentate in una K-V store**
  - La chiave della K-V corrisponde alla primary key della tabella.
  - il valore della K-V corrisponde alla rappresentazione binaria di tutti i valori delle colonne di quella riga.
  - <img src="https://github.com/FabioR1995/CockRoachDB/blob/main/Immagini/k-v_store.png" width="450" height="200">
  - E' possibile anche indirizzare cockroach a gruppi di colonne in separate voci K-V (**column families**).
    - <img src="https://github.com/FabioR1995/CockRoachDB/blob/main/Immagini/column_families.png" width="450" height="350"> 
  -
- La definizione dello schema per le tabelle (e gli indici ad essi associati), vengono memorizzati in uno speciale keyspace chiamato table descriptor.
  - Per questioni di performance la table descriptor viene memorizzata in ogni nodo.
  - La table descriptor viene usata per fare parsing e ottimizzare i SQL e costruire correttamente le operazioni K-V per una tabella.
  - CockRoachDB supporta le modifiche online dei cambiamenti degli schema utilizzando ALTER TABLE, CREATE INDEX e altri comandi.
    - Lo schema viene cambiato in fasi discrete, in modo tale che venga introdotto mentre è ancora in uso lo schema precedente, operando quindi come cambiamenti in background.
  - I cambiamenti di uno schema potrebbero involvere in cambiamenti sui dati delle tabelle (aggiunta o rimozione delle colonne) e/o con la creazione delle strutture di nuovi indici. Quando tutte le tabelle di tutte le istanze sono memorizzate in concomitanza ai requisiti del nuovo schema, allora tutti i nodi switcheranno al nuovo schema e permetteranno la lettura/scrittura delle tabelle usando il nuovo schema.
- **Transiction Layer CockRoachDB**
  - E' lo strato responsabile per mantenere la atomicità delle transazioni assicurando che tutte le transazioni siano committate o abortite.
  - da riempire con gli appunti scritti nel pc aziendale.
- **Principi MVCC**
  -   al tempo t1, viene letto dalla sessione s1 la riga r2 e acceduta la versione v1 della riga. (1)
  -   Al tempo t2, un' altra sessione database (s2) aggiorna la riga creando la seconda versione di quella riga. (2)
  -   viene creata la seconda versione di quella riga. (3)
  -   Al tempo t3, la sessione s1 legge nuovamente la riga, ma visto che la sessione s2 non la ha ancora committata viene letta la versione 1. (4)
  -   Dopo il commit dalla sessione 2 (5), la sessione 1 potrà ora leggere la nuova versione v2 della riga (6).
  -   <img src="https://github.com/FabioR1995/CockRoachDB/blob/main/Immagini/mvcc.png" width="450" height="350">
  -   write intents
    -  Durante una fase iniziale del processo di transazione, quando non sappiamo se la transazione avverrà con successo, il nodo portatore scriverà questo tentativo di scrittura sul valore che è stato tentato di modificare.
    -  Il write intent (intento di scrittura) è uno speciale costrutto MVCC, il quale viene marchiato come provvisorio.
    
    
