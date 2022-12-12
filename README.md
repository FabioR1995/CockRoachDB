# CockRoachDB
Informazioni riguardanti CockRoachDB

# ARCHITETTURA
- Una distribuzione di CockRoachDB consiste in uno o piu processi server del database
- I nodi di un cluster CockRoach sono simmetrici, cioè non esiste un nodo "primario" o "speciale"
- Il protocollo attraverso il quale avviene la comunicazione fra il client e server è il **wire protocol di PostgreSQL** 
  - Questo protocollo descrive come avvengono le richieste/risposte SQL sono trasmette fra il PostgreSQL client/server.
    - Ogni driver di PostgreSQL può essere utilizzato per comunicare con CockRoachDB.
