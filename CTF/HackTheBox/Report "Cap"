# Report tecnico -- Attività di penetration testing su macchina CTF

## 1. Ricognizione iniziale

Per iniziare l'attività ho eseguito una scansione della macchina target
utilizzando **Nmap**, con le opzioni `-sV` e `-O`.

-   `-sV`: permette di identificare i servizi e, quando possibile, le
    relative versioni.
-   `-O`: tenta di riconoscere il sistema operativo della macchina
    target.

La scansione ha individuato **tre porte aperte**. Tra i servizi rilevati
ho deciso di concentrarmi inizialmente sulla porta **80**, associata al
protocollo HTTP, per analizzare il sito web esposto.

## 2. Analisi del servizio web

Accedendo all'indirizzo IP della macchina tramite browser, ho
individuato un sito web dedicato alla visualizzazione di file di cattura
del traffico di rete in formato **PCAP**.

Durante l'esplorazione del sito ho notato che l'URL utilizzava un
percorso piuttosto semplice:

``` text
/data/1
```

La struttura dell'indirizzo suggeriva che il numero finale potesse
identificare una specifica acquisizione di traffico.

Per verificare questa ipotesi ho modificato il valore numerico,
sostituendo `1` con `0`. In questo modo sono riuscito ad accedere alle
scansioni associate a un altro utente, che ho successivamente scaricato
per analizzarle.

Questo comportamento rappresentava un possibile problema di **controllo
degli accessi**: il sito permetteva infatti di visualizzare dati
appartenenti a un altro utente modificando direttamente un parametro
dell'URL.

## 3. Analisi del traffico con Wireshark

Dopo aver scaricato i file PCAP, li ho aperti con **Wireshark**, uno
strumento utilizzato per analizzare il traffico di rete e ricostruire le
comunicazioni tra host.

Durante l'ispezione ho individuato uno **stream TCP** contenente
informazioni di autenticazione trasmesse in chiaro.

In particolare, all'interno della comunicazione erano visibili un nome
utente e una password.

Il ritrovamento di queste credenziali è stato fondamentale per
proseguire l'attività, poiché mi ha fornito i dati necessari per tentare
l'accesso diretto al sistema.

## 4. Accesso alla macchina tramite SSH

Considerando che durante la ricognizione era stata individuata anche la
porta **22**, ho deciso di utilizzare il servizio SSH per verificare le
credenziali recuperate dall'analisi del traffico.

Ho quindi avviato una connessione SSH verso la macchina target,
inserendo username e password individuati nello stream TCP.

L'autenticazione è andata a buon fine e sono riuscito ad accedere al
sistema con l'account dell'utente di base.

Una volta all'interno della macchina, ho esplorato il filesystem e
individuato il file `user.txt`, contenente la prima flag.

## 5. Enumerazione delle capability e escalation dei privilegi

Dopo aver recuperato la prima flag, ho proseguito l'analisi del sistema
con l'obiettivo di individuare possibili configurazioni che
consentissero di ottenere privilegi più elevati.

Per verificare le capability assegnate ai programmi presenti sulla
macchina ho utilizzato il comando:

``` bash
getcap -r / 2>/dev/null
```

L'enumerazione ha evidenziato che l'interprete **Python 3.8** disponeva
di una capability che gli consentiva di modificare il proprio UID,
permettendo quindi di sfruttare questa configurazione per ottenere
privilegi di root.

Ho utilizzato il seguente comando:

``` bash
python3.8 -c 'import os; os.setuid(0); os.system("/bin/sh")'
```

Il comando ha eseguito le seguenti operazioni:

1.  Ha avviato l'interprete Python 3.8.
2.  Ha importato il modulo `os`, che permette di interagire con alcune
    funzionalità del sistema operativo.
3.  Ha richiamato `os.setuid(0)` per impostare l'UID del processo a `0`,
    corrispondente all'utente root.
4.  Ha avviato una shell tramite `/bin/sh`.

Grazie alla capability assegnata a Python 3.8, sono riuscito a ottenere
una shell con privilegi amministrativi.

## 6. Recupero della flag di root

Una volta ottenuto l'accesso come root, ho navigato nella directory:

``` bash
cd /root
```

All'interno ho individuato il file `root.txt`, contenente la seconda
flag.

Ho quindi utilizzato il comando `cat` per visualizzarne il contenuto e
completare la macchina.

## 7. Conclusioni

L'attività si è conclusa con il recupero di entrambe le flag: `user.txt`
e `root.txt`.

Durante la risoluzione ho utilizzato diverse tecniche e strumenti tipici
delle attività di penetration testing:

-   **Nmap:** ricognizione iniziale, identificazione dei servizi e
    tentativo di riconoscimento del sistema operativo.
-   **Browser:** analisi del servizio web e individuazione di percorsi
    accessibili.
-   **Manipolazione dell'URL:** accesso a dati appartenenti a un altro
    utente tramite la modifica dell'identificativo della risorsa.
-   **Wireshark:** analisi dei file PCAP e ricostruzione di uno stream
    TCP.
-   **SSH:** accesso remoto alla macchina utilizzando le credenziali
    recuperate.
-   **getcap:** enumerazione delle capability assegnate ai programmi.
-   **Python 3.8:** sfruttamento di una capability e ottenimento di una
    shell con privilegi di root.

La macchina presentava principalmente due criticità: un controllo degli
accessi insufficiente sul sito web, che permetteva di consultare le
acquisizioni di traffico di un altro utente, e una configurazione non
sicura delle Linux capabilities, che consentiva a Python 3.8 di ottenere
privilegi amministrativi.

Un aspetto particolarmente importante emerso durante questa attività è
il rischio legato alla trasmissione di credenziali in chiaro. Anche
quando le credenziali non sono immediatamente esposte da un servizio,
possono essere recuperate analizzando le comunicazioni di rete salvate
in file PCAP.

L'esercizio ha quindi permesso di mettere in pratica un percorso
completo: dalla ricognizione iniziale, all'analisi del traffico, fino
all'accesso remoto e all'escalation dei privilegi.
