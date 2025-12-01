# Dockerized-network-security

Benvenuto nel mio progetto personale dedicato alla costruzione di un ambiente self-hosted, modulare e scalabile, basato su:

* Ubuntu Server
* Docker & Docker Compose
* Portainer
* AdGuard Home (DNS filtering)
* WireGuard (VPN moderna e sicura)

Lo scopo della repository è documentare passo dopo passo la configurazione dell’intero sistema, in modo semplice, ripetibile e adatto sia ad ambienti VirtualBox sia ad hardware dedicato come Raspberry Pi 5.
Ho cercato di includere i problemi riscontrati e le relative soluzioni.

📚 Indice dei contenuti

Ciascuna guida è contenuta nella cartella /docs.

1️⃣ Preparazione dell’ambiente (Ubuntu Server)

📄 step1-UBUNTU-SERVER.md

Installazione di Ubuntu Server, configurazioni iniziali, SSH, rete, locale, troubleshooting.

2️⃣ Installazione di Docker

📄 step2-DOCKER.md

Installazione ufficiale di Docker, prerequisiti, repository GPG, test e configurazione.

3️⃣ Installazione e configurazione di Portainer

📄 step3-PORTAINER.md

Gestione visuale dei container, deploy di stack, gestione volumi e rete Docker.

4️⃣ Installazione di AdGuard Home

📄 step4-ADGUARD.md

Setup DNS filtering, mapping porte, gestione volumi, configurazione interfaccia web.

5️⃣ Installazione di WireGuard VPN

📄 step5-WIREGUARD.md

Creazione server VPN, rete interna, generazione peer, forwarding router, sicurezza.

🧱 Obiettivi del progetto

Centralizzare servizi di rete in un ambiente dockerizzato

Migliorare sicurezza e privacy tramite DNS filtering + VPN

Preparare un’infrastruttura portabile verso Raspberry Pi

Documentare tutto per poter replicare facilmente il sistema
