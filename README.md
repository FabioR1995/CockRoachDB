# CockRoachDB
Informazioni riguardanti CockRoachDB

# ARCHITETTURA
- Una distribuzione di CockRoachDB consiste in uno o piu processi server del database.
- I nodi di un cluster CockRoachDB sono simmetrici, cioè non esiste un nodo "primario" o "speciale".
- Il protocollo attraverso il quale avviene la comunicazione fra il client e server è il **wire protocol di PostgreSQL**.
  - Questo protocollo descrive come avvengono le richieste/risposte SQL sono trasmette fra il PostgreSQL client/server.
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
