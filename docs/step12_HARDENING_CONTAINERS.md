Anche se l’app gira in un container “minimal” e non-root, il rischio non è zero: se l’app viene compromessa (RCE), l’attaccante ottiene un punto d’appoggio nella tua rete interna e può provare movimento laterale verso altri servizi/host, e in alcuni casi tentare una “container escape” sfruttando configurazioni deboli o bug di kernel/runtime.​

Uscita dal container (escape)
L’isolamento dei container non è “una VM”: condividono il kernel dell’host, quindi vulnerabilità del kernel/handler di syscalls o del runtime possono permettere escalation/escape anche partendo da dentro il container. Nella pratica, molte escape diventano realistiche quando si combinano 2 fattori: una vulnerabilità + una configurazione rischiosa (privileged, mount pericolosi, ecc.). Un caso classico e devastante è avere montato /var/run/docker.sock: da lì un attaccante può pilotare Docker e far partire un container privilegiato con mount del filesystem host, ottenendo accesso al sistema.​

Movimento laterale (lateral movement)
Il fatto che sia “in rete interna proxata da Caddy” riduce l’esposizione diretta, ma se l’app dietro Caddy ha una falla e viene eseguito codice nel container, da lì l’attaccante può comunque parlare con ciò che è raggiungibile sulla rete Docker/LAN (DB, Redis, altri container, servizi interni). Docker, su bridge default, permette inter-container connectivity di base, quindi se metti tanti servizi sulla stessa rete senza segmentazione, stai creando una rete “piatta” che facilita il movimento laterale. La mitigazione tipica è micro-segmentare: reti separate e regole che permettono solo i flussi necessari, così anche se un servizio cade non diventa automaticamente “pivot” verso tutto il resto.​

Nel tuo setup: cosa conta davvero
“Python minimal non-root” aiuta perché riduce l’impatto di molte escalation banali dentro al container, ma non protegge da bug kernel/host e non impedisce scansione/abusi verso altri target raggiungibili via rete. La vera differenza la fanno: privilegi/capabilities del container (es. capability pericolose come quelle che abilitano azioni quasi “da host”) e mount sensibili (docker.sock, volumi host in scrittura, /proc//sys esposti male). Anche le policy di sistema (seccomp/AppArmor) contano, perché limitano le syscalls e quindi restringono la superficie per exploit e tecniche di escape.​

Hardening pratico (alto ROI)
-Taglia la rete: crea reti Docker distinte (frontend/proxy, backend, db) e attacca i container solo alle reti necessarie, come raccomandato nelle best practice per ridurre lateral movement.​
-Evita “chiavi del regno”: niente --privileged, niente --cap-add non indispensabili, e soprattutto niente mount di /var/run/docker.sock dentro container applicativi.​
-Riduci impatto di un’escape: valuta Docker rootless (o almeno userns-remap) per limitare i danni anche se un container viene bucato.​
-Abilita restrizioni runtime: mantieni seccomp attivo (profilo default o più stretto) e profili LSM (AppArmor/SELinux) dove possibile


Una “container escape” è quando un processo che gira dentro un container riesce a uscire dai confini di isolamento del container e ottenere accesso a risorse che non dovrebbe vedere, come filesystem/risorse dell’host o altri container sullo stesso host.​

Cosa significa in pratica
Se un attaccante compromette la tua app (es. RCE) e poi fa container escape, non è più “limitato” a quel container: può potenzialmente arrivare all’host Linux e quindi avere un impatto molto più grave (furto di dati, persistenza, controllo di altri servizi).​

Come può succedere
Le due cause tipiche sono: (1) misconfigurazioni che danno troppi privilegi al container (capabilities eccessive, container privilegiati, mount pericolosi), oppure (2) vulnerabilità nel runtime/kernel che permettono di bypassare i meccanismi di isolamento. Un esempio pratico di misconfigurazione molto rischiosa è dare al container accesso al Docker socket (/var/run/docker.sock), perché può portare a controllo del motore Docker e quindi dell’host.​

Perché è importante
I container condividono il kernel dell’host, quindi l’isolamento non è “assoluto” come una VM: un’escape riuscita è tra gli scenari peggiori perché può trasformare una singola app bucata in una compromissione dell’intero server.

container escape (uscire dal container verso l’host) e lateral movement (muoversi da un servizio compromesso verso altri servizi/host).​

Cosa vuol dire ogni punto (in concreto)
-Kernel aggiornato e patch veloci: i container condividono il kernel dell’host, quindi molte tecniche di escape sfruttano CVE del kernel/cgroups; patchare e riavviare su kernel fixed riduce proprio quella classe di attacchi.​
-SELinux/AppArmor + seccomp: sono “guardrail” runtime; OWASP consiglia di non disabilitare i profili di sicurezza di default e di usare seccomp/AppArmor/SELinux per restringere syscalls e azioni possibili nel container.​
-Ridurre privilegi: OWASP raccomanda di “set a user” (non root) e di prevenire escalation in-container (es. no-new-privileges, limitazione capabilities) perché i privilegi extra amplificano l’impatto di una compromissione.​
-Isolare risorse host (mount/namespace/cgroups): mount in RW di path sensibili, accesso eccessivo a /proc//sys, o esposizioni tipo Docker socket aumentano tantissimo le chance di takeover/escape.​
-Audit delle immagini: usare immagini trusted e scanner di vulnerabilità riduce la probabilità di portarti in casa CVE note (dipendenze Python, libc, openssl, ecc.).​
-Segmentazione di rete: OWASP cita anche il tema “disable inter-container communication” e, più in generale, separare reti/permessi limita il lateral movement se un container viene bucato.​
-Test e monitoraggio: l’idea è trovare vulnerabilità/config sbagliate prima degli altri e rilevare comportamenti anomali a runtime (exec sospetti, connessioni strane, ecc.).​

Tradotto nel tuo scenario (Python non-root + Caddy)
Queste misure non dicono “sei al 100% sicuro”, dicono “anche se ti bucano l’app, è più difficile uscire dal container e più difficile muoversi lateralmente”. Il grosso del rischio residuo, di solito, sta in: rete troppo piatta (tutti i container insieme), privilegi/capabilities/mount eccessivi, e host non aggiornato.


# trivy

sudo apt-get update
sudo apt-get install -y wget apt-transport-https gnupg lsb-release

wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key \
  | gpg --dearmor \
  | sudo tee /usr/share/keyrings/trivy.gpg > /dev/null

echo "deb [signed-by=/usr/share/keyrings/trivy.gpg] https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main" \
  | sudo tee /etc/apt/sources.list.d/trivy.list

sudo apt-get update
sudo apt-get install -y trivy

trivy image --scanners vuln --severity HIGH,CRITICAL nextcloud:latest


