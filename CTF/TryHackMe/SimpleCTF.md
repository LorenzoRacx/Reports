# Report tecnico -- Attività di penetration testing su macchina CTF

## 1. Ricognizione iniziale

Per iniziare l'attività ho effettuato una scansione della macchina
target tramite **Nmap**, con l'obiettivo di individuare le porte aperte
e i servizi in esecuzione.

Dalla scansione sono emerse tre porte aperte:

  **Porta**   **Servizio**
  ----------- --------------
  21          FTP
  80          HTTP
  2222        SSH

La presenza di questi servizi ha permesso di individuare i primi
possibili punti di ingresso nella macchina.

Successivamente ho effettuato una scansione con **Nuclei**, che ha
evidenziato alcune configurazioni e vulnerabilità potenzialmente
sfruttabili:

-   **FTP:** possibilità di accesso anonimo (*anonymous login*).
-   **OpenSSH 7.2p2:** vulnerabilità di username enumeration, che
    potrebbe consentire di verificare l'esistenza di determinati utenti
    sul sistema.
-   **Credenziali FTP deboli:** è emerso un possibile problema relativo
    alla gestione delle password.

Durante la ricognizione ho inoltre identificato il server web come
**Apache/2.4.18 (Ubuntu)**.

## 2. Analisi del servizio FTP

Considerando la possibilità di accedere al servizio FTP senza
autenticazione, ho effettuato una connessione anonima e analizzato i
file disponibili sul server.

Tra i file presenti ho trovato un documento di testo contenente il
seguente messaggio:

> Dammit man... you're the worst dev i've seen. You set the same pass
> for the system user, and the password is so weak... i cracked it in
> seconds. Gosh... what a mess!

Il messaggio rappresentava un indizio importante: suggeriva che un
utente di sistema utilizzasse la stessa password di un altro account e
che tale password fosse particolarmente debole.

Ho quindi tenuto in considerazione questa informazione durante le
successive fasi di enumerazione.

## 3. Enumerazione del server web

Ho proseguito analizzando il servizio HTTP sulla porta 80. Per
individuare eventuali directory e pagine non immediatamente visibili, ho
utilizzato **Gobuster**, selezionando una wordlist di dimensioni ridotte
(*small*).

La scansione ha individuato una directory denominata `/simple`.

Accedendo alla pagina ho scoperto che il sito utilizzava **CMS Made
Simple versione 2.2.8**.

Dalle successive verifiche ho individuato una vulnerabilità nota di tipo
**SQL Injection**, identificata dal codice **CVE-2019-9053**.

Questa vulnerabilità consentiva di recuperare informazioni dal database
del CMS, tra cui dati relativi agli utenti e le rispettive password
salvate sotto forma di hash.

## 4. Sfruttamento della SQL Injection

Per sfruttare la vulnerabilità ho cercato su **Exploit-DB** un exploit
Python specifico per la CVE individuata.

Dopo averlo eseguito contro il server, sono riuscito a recuperare
diverse informazioni relative all'account amministrativo:

``` text
[+] Salt for password found: 1dac0d92e9fa6bb2

[+] Username found: mitch

[+] Email found: admin@admin.com

[+] Password found: 0c01f4468bd75d7a84c7eb73846e8d96
```

Tra i dati ottenuti era presente anche l'hash della password dell'utente
`mitch`.

## 5. Recupero della password

Una volta ottenuto l'hash, ho utilizzato il servizio online
**Hashes.com** per tentare di identificarne la password originale.

L'operazione ha avuto esito positivo: la password recuperata è risultata
essere:

``` text
secret
```

Questo risultato ha confermato l'indizio trovato precedentemente sul
server FTP, relativo all'utilizzo di password deboli e riutilizzate.

## 6. Accesso tramite SSH

Disponendo di username e password, ho tentato l'accesso alla macchina
tramite SSH, utilizzando la porta 2222 individuata durante la scansione
iniziale.

I dati utilizzati sono stati:

-   **Indirizzo:** IP della macchina target
-   **Porta:** 2222
-   **Username:** mitch
-   **Password:** secret

L'autenticazione è andata a buon fine e sono riuscito ad accedere al
sistema con l'account `mitch`.

Una volta ottenuto l'accesso, ho iniziato a esplorare il filesystem
utilizzando il comando `ls`.

Nella directory dell'utente ho individuato il file `user.txt`,
contenente la prima flag. Ho utilizzato il comando `cat` per
visualizzarne il contenuto.

## 7. Escalation dei privilegi

Dopo aver recuperato la prima flag, ho proseguito cercando un modo per
ottenere privilegi amministrativi sulla macchina.

Durante l'attività ho utilizzato **Vim**, sfruttando la possibilità di
eseguire comandi direttamente dalla sua interfaccia tramite il comando:

``` bash
:!/bin/bash
```

Questo ha permesso di avviare una shell Bash. Nel contesto utilizzato,
l'operazione ha consentito di ottenere una shell con privilegi di root.

Una volta ottenuti i privilegi amministrativi, ho navigato nella
directory `/root`, dove ho individuato il file `root.txt`.

Anche in questo caso ho utilizzato `cat` per leggere il contenuto del
file e recuperare la seconda flag.

## 8. Conclusioni

L'attività si è conclusa con il recupero di entrambe le flag, `user.txt`
e `root.txt`, ottenendo quindi l'accesso completo alla macchina.

Durante la risoluzione sono state utilizzate diverse tecniche e
strumenti tipici delle attività di penetration testing:

-   **Nmap:** individuazione delle porte e dei servizi esposti.
-   **Nuclei:** identificazione di vulnerabilità e configurazioni
    potenzialmente insicure.
-   **Gobuster:** enumerazione di directory e risorse web.
-   **Exploit-DB:** ricerca di un exploit specifico per la vulnerabilità
    individuata.
-   **SQL Injection:** recupero di informazioni dal database del CMS.
-   **Hashes.com:** identificazione della password a partire dall'hash.
-   **SSH:** accesso remoto alla macchina.
-   **Vim:** utilizzo della funzionalità di esecuzione di comandi per
    ottenere una shell con privilegi elevati.

La macchina presentava diverse criticità di sicurezza, tra cui l'accesso
anonimo al servizio FTP, password deboli e riutilizzate, un CMS
vulnerabile e una possibilità di escalation dei privilegi.

L'esperienza ha mostrato come informazioni apparentemente secondarie,
come un semplice file di testo presente su un server FTP, possano
diventare indizi fondamentali per ricostruire il percorso di
compromissione di un sistema.
