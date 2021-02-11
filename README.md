# Kerberos

<p align="center">
  <img src="https://web.mit.edu/kerberos/images/dog-ring.jpg" />
</p>

Kerberos è un protocollo di autenticazione dei servizi di rete creato da MIT che si serve della crittografia a chiave segreta per autenticare gli utenti per i servizi di rete - eliminando così la necessità di inviare le password attraverso la rete.

Autenticare mediante Kerberos impedisce agli utenti non autorizzati di intercettare le password inviate attraverso la rete, prevenendo quindi attacchi del tipo **eavesdropping** e **replay attack** e assicurando l'integrità dei dati.

I passi necessari per l'autenticazione in un ambiente che implementa Kerberos sono i seguenti:
1. Il client richiede un TGT (Ticket Granted Ticket / Authentication Ticket) al KDC (Key Distribuition Center)
2. Il KDC verifica le credenziali e invia come risposta un TGT cifrato ed una Session Key.
3. Il TGT è cifrato tramite la Secret Key del TGS (Ticket Granting Service).
4. Il client memorizza il TGT e quando scade il Local Session Manager ne richiede uno nuovo (processo trasparente all'utente).

Se il client sta richiedendo l'accesso ad un servizio o una risorsa nella rete, il processo è il seguente (il processo di autenticazione dell'utente deve essere già avvenuto e completato):
1. Il client invia il TGT corrente al TGS con il SPN (Service Principal Name) della risorsa a cui vuole accedere.
2. Il KDC verifica il TGT e che l'utente abbia accesso al servizio.
3. Il TGS invia al client una Session Key valida per il serivizio.
4. Il client inoltra la Session Key al servizio per provare i propri privilegi e infine il servizio concede l'accesso.

Sono possibili numerosi attacchi a Kerberos:
* **Pass-the-ticket**: Viene forgiata una Session Key e viene presentata alla risorsa come credenziale.
* **Golden Ticket:** Un ticket che garantisce l'accesso ad un utente con ruolo di Domain Admin.
* **Silver Ticket:** Un ticket forgiato che permette l'accesso ad un servizio.
* **Credential Stuffing/Brute force:** Processo di invio continuo di password per cercare di indovinare quella corretta.
* **Encryption downgrade with Skeleton Key Malware:** Un malware che può bypassare Kerberos (l'attacco deve avvenire già con l'accesso all'admin).
* **DCShadow attack:** Un attaccante guadagna l'accesso all'interno di una rete dove installa un DC (Domain Controller) malevolo da utilizzare in attacchi futuri.

> Filippo Zane | 11.02.2021
