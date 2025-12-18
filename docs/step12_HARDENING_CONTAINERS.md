# Ulteriori misure di hardening dei Containers

Anche se l’app gira in un container “minimal” e non-root, il rischio non è azzerato: se l’app viene compromessa (RCE), l’attaccante ottiene un punto d’appoggio nella rete interna e può provare movimento laterale verso altri servizi/host, e in alcuni casi tentare una “container escape” sfruttando configurazioni deboli o bug di kernel/runtime.​

## Container escape
Una “container escape” è quando un processo che gira dentro un container riesce a uscire dai confini di isolamento del container e ottenere accesso a risorse che non dovrebbe vedere, come filesystem/risorse dell’host o altri container sullo stesso host.​
Se un attaccante compromette la tua app (es. RCE) e poi fa container escape, non è più “limitato” a quel container: può potenzialmente arrivare all’host Linux e quindi avere un impatto molto più grave (furto di dati, persistenza, controllo di altri servizi).​

Come può succedere
Le due cause tipiche sono: (1) misconfigurazioni che danno troppi privilegi al container (capabilities eccessive, container privilegiati, mount pericolosi), oppure (2) vulnerabilità nel runtime/kernel che permettono di bypassare i meccanismi di isolamento. Un esempio pratico di misconfigurazione molto rischiosa è dare al container accesso al Docker socket (/var/run/docker.sock), perché può portare a controllo del motore Docker e quindi dell’host.​

Perché è importante
I container condividono il kernel dell’host, quindi l’isolamento non è “assoluto” come una VM: un’escape riuscita è tra gli scenari peggiori perché può trasformare una singola app bucata in una compromissione dell’intero server.

container escape (uscire dal container verso l’host) e lateral movement (muoversi da un servizio compromesso verso altri servizi/host).​

## Lateral movement
Il fatto che sia “in rete interna proxata da Caddy” riduce l’esposizione diretta, ma se l’app dietro Caddy ha una falla e viene eseguito codice nel container, da lì l’attaccante può comunque parlare con ciò che è raggiungibile sulla rete Docker/LAN (DB, Redis, altri container, servizi interni). Docker, su bridge default, permette inter-container connectivity di base, quindi se metti tanti servizi sulla stessa rete senza segmentazione, stai creando una rete “piatta” che facilita il movimento laterale. La mitigazione tipica è micro-segmentare: reti separate e regole che permettono solo i flussi necessari, così anche se un servizio cade non diventa automaticamente “pivot” verso tutto il resto.​

## Contromisure
“Python minimal non-root” aiuta perché riduce l’impatto di molte escalation banali dentro al container, ma non protegge da bug kernel/host e non impedisce scansione/abusi verso altri target raggiungibili via rete. La vera differenza la fanno: privilegi/capabilities del container (es. capability pericolose come quelle che abilitano azioni quasi “da host”) e mount sensibili (docker.sock, volumi host in scrittura, /proc//sys esposti male). Anche le policy di sistema (seccomp/AppArmor) contano, perché limitano le syscalls e quindi restringono la superficie per exploit e tecniche di escape.​

## Container security attraverso integrazioni nel docker-compose.yml

## Cap_drop: ALL 
In Linux, il potere dell'utente root non è un blocco unico, ma è diviso in circa 40 unità chiamate **Capabilities**.

Tradizionalmente, in Linux, il mondo era diviso in due:
1. **Root (UID 0)**: Può fare tutto.
2. **Utenti normali**: Non possono fare quasi nulla che riguardi il sistema (modificare la rete, spegnere la macchina, ecc.).

Il problema è che spesso un'applicazione ha bisogno di un solo piccolo potere del root. Per esempio, un server web (come Nginx) ha bisogno di legarsi alla porta 80. In Linux, solo il root può usare le porte sotto la 1024. Senza le capabilities, saresti costretto a far girare tutto Nginx come root, il che è un enorme rischio di sicurezza.

**Le Capabilities spezzettano il potere assoluto del root in circa 40 "micro-permessi" indipendenti.**

Le Capabilities più comuni:

CAP_NET_BIND_SERVICE: Permette di occupare porte basse (es. 80, 443) anche se non sei root.
CAP_CHOWN: Permette di cambiare il proprietario dei file.
CAP_NET_ADMIN: Permette di configurare interfacce di rete e firewall.
CAP_SYS_TIME: Permette di cambiare l'orologio di sistema.
CAP_KILL: Permette di killare processi che appartengono ad altri utenti.

Le capabilities vengono gestite attraverso dei set (insiemi) che il kernel controlla ogni volta che un processo prova a fare un'operazione privilegiata.

I tre set principali che trovi in un processo sono:

- **Permitted**: È il "limite massimo". Sono tutte le capacità che il processo può effettivamente usare o attivare.
- **Inheritable**: Sono le capacità che il processo può tramandare ai suoi processi figli (es. quando lanci un altro programma).
- **Effective**: È il set dei poteri attualmente attivi. Quando il kernel deve decidere se farti fare qualcosa, guarda qui.

**Cap_drop: ALL**: Rimuove letteralmente ogni privilegio speciale. Il processo diventa un "ospite" estremamente limitato.
Molti exploit sfruttano capacità specifiche (come CAP_RAW_NET) per fare scansioni di rete o attacchi ARP spoofing all'interno del cluster. Se le togliamo tutte, l'attaccante si ritrova con le mani legate.

**Di solito si usa cap_drop: ALL e poi si riaggiunge solo lo stretto necessario con cap_add.**

## No-new-privileges: true 
Questo è un parametro del kernel Linux (PR_SET_NO_NEW_PRIVS). È la difesa contro il classico attacco di **Privilege Escalation.**

Il trucco del SUID: In Linux, esistono file (come passwd o sudo) che hanno un bit chiamato SUID. Se esegui quel file, lo esegui con i permessi del proprietario (root), non i tuoi.

Cosa fa questa opzione: Impedisce al processo (e a tutti i suoi figli) di guadagnare nuovi privilegi tramite bit SUID o SGID.

Scenario: Se un attaccante riesce a caricare uno script malevolo che tenta di sfruttare un binario di sistema per diventare root, il kernel bloccherà l'operazione sul nascere. È come dire al sistema: "Qualunque cosa accada, questo processo non può mai diventare più potente di quanto lo sia ora".

## Tmpfs: /tmp (Isolamento della scrittura)
Montare /tmp in RAM (tramite tmpfs) ha tre vantaggi strategici:

**Anti-Persistenza**: Se un attaccante scarica un malware o un file di configurazione malevolo in /tmp, non appena il container crasha o viene riavviato, quel file svanisce nel nulla.

**Performance**: Scrivere in RAM è molto più veloce che scrivere sul file system del container (che spesso usa un driver "overlay" lento).

**Read-Only Root FS**: Spesso questa opzione si usa insieme a readOnlyRootFilesystem: true. Se rendi tutto il container non scrivibile (per sicurezza), l'app smetterebbe di funzionare perché non può scrivere i suoi file temporanei. Usando tmpfs su /tmp, dai all'app un piccolo spazio sicuro dove "sfogarsi" senza compromettere l'integrità del resto del file system.

## Hardening 

1. **segmentare la rete**: crea reti Docker distinte e attacca i container solo alle reti necessarie, come raccomandato nelle best practice per ridurre **lateral movement**.​
2. Evitare  **--privileged** e **--cap-add(permessi Linux granulari) non indispensabili**, e soprattutto niente mount di /var/run/docker.sock dentro container applicativi.​
3. Ridurre impatto di una **container escape**: valutare Docker rootless per limitare i danni anche se un container viene bucato.​
4. **Abilitare restrizioni runtime**: mantieni seccomp attivo (profilo default o più stretto) e profili LSM (AppArmor/SELinux) dove possibile
5. **Kernel aggiornato e patch veloci**: i container condividono il kernel dell’host, quindi molte tecniche di escape sfruttano CVE del kernel/cgroups; patchare e riavviare su kernel fixed riduce proprio quella classe di attacchi.​
6. **SELinux/AppArmor + seccomp** OWASP consiglia di non disabilitare i profili di sicurezza di default e di usare seccomp/AppArmor/SELinux per restringere syscalls e azioni possibili nel container.
**Application armor** è un sistema di Mandatory Access Control (MAC), ovvero controllo degli accessi obbligatorio: non basta che un file    sia leggibile dal tuo utente, deve anche essere esplicitamente permesso dal profilo AppArmor del programma che stai usando
Quando usi Docker su sistemi come Ubuntu o Debian, Docker applica automaticamente un profilo di base chiamato docker-default a tutti i container, a meno che tu non lo disabiliti.​

Questo profilo predefinito fa già un buon lavoro:

Impedisce di scrivere in cartelle critiche del sistema (/sys, /proc).
Blocca operazioni pericolose (come montare nuovi dischi).
Riduce il rischio che un exploit nel container riesca a "scappare" sull'host

7.Ridurre privilegi: OWASP raccomanda di “set a user” (non root) e di prevenire escalation in-container (es. no-new-privileges, limitazione capabilities) perché i privilegi extra amplificano l’impatto di una compromissione.
8. Isolare risorse host (mount/namespace/cgroups):
- **Mount di Path Sensibili (in RW)**
- **Montare una cartella del server (host) dentro il container con permessi di scrittura** (RW - Read/Write). Ad esempio, montare la cartella /etc dell'host dentro il container.

Se un attaccante ottiene il controllo dell'app nel container, può usare questo "ponte" per modificare direttamente i file di sistema dell'host. Potrebbe aggiungere un utente, modificare le chiavi SSH autorizzate, disabilitare il firewall, ecc. È la via più diretta per un takeover completo del server.

- **Accesso a /proc o /sys**
/proc e /sys sono filesystem speciali in Linux che non contengono file veri e propri, ma l'interfaccia di controllo del kernel. Permettono di vedere e a volte modificare i processi in esecuzione e i parametri del sistema operativo.

Di default, Docker limita ciò che un container può vedere e fare con /proc e /sys. Ma se si allentano queste restrizioni (spesso usando --privileged), un attaccante potrebbe raccogliere informazioni su tutti i processi dell'host (non solo quelli del container) o tentare di manipolare il kernel per facilitare un escape (fuga dal container).

- **Esposizione del Docker Socket**
Montare il file /var/run/docker.sock dell'host all'interno di un container.

Questo file è l'API del demone Docker. Chiunque possa comunicare con questo socket può dare ordini a Docker, come "avvia un nuovo container". Un attaccante che controlla un container con accesso al socket può semplicemente lanciare un altro container, questa volta privilegiato (--privileged) e con la cartella radice dell'host (/) montata al suo interno. A quel punto ha il controllo totale della macchina. È l'escape più classico e devastante.<br>
​
9.  **Audit delle immagini**: usare immagini trusted e scanner di vulnerabilità ,come Trivy nel mio caso,riduce la probabilità di portarti in casa CVE note (dipendenze Python, libc, openssl, ecc.).<br>​
10. **Segmentazione di rete**: OWASP cita anche il tema “disable inter-container communication” e, più in generale, separare reti/permessi limita il lateral movement se un container viene bucato.<br>
​
11. Test e monitoraggio: l’idea è trovare vulnerabilità/config sbagliate prima degli altri e rilevare comportamenti anomali a runtime (exec sospetti, connessioni strane, ecc.).<br>​
​
# Trivy

**Trivy è uno scanner di sicurezza per:**
- immagini Docker
- filesystem
- repository di codice
- dipendenze
- configurazioni (IaC)

**Trivy individua:**

- CVE nelle librerie (Python, pip, apt, ecc.)
- pacchetti di sistema vulnerabili
- secret hardcoded (password, token)
- errori di configurazione Dockerfile / compose
- immagini base vecchie o insicure


```dockerfile
sudo apt-get update
sudo apt-get install -y wget apt-transport-https gnupg lsb-release

wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key \
  | gpg --dearmor \
  | sudo tee /usr/share/keyrings/trivy.gpg > /dev/null

echo "deb [signed-by=/usr/share/keyrings/trivy.gpg] https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main" \
  | sudo tee /etc/apt/sources.list.d/trivy.list

sudo apt-get update
sudo apt-get install -y trivy

trivy image --scanners vuln --severity HIGH,CRITICAL nextcloud:latest           #per scannerizzare una immagine specifica
```

