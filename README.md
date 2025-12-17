# Dockerized-network-security

**Benvenuto nel mio progetto personale dedicato alla costruzione di un ambiente self-hosted, modulare e scalabile, basato su:**

* Ubuntu Server
* Docker & Docker Compose
* Portainer
* AdGuard Home (DNS filtering)
* WireGuard (VPN)
* Caddy (reverse proxy)
* Ip tables/UFW (firewall nativo Linux)

Lo scopo della repository è documentare passo dopo passo la configurazione dell’intero sistema, in modo semplice, ripetibile e adatto sia ad ambienti VirtualBox sia ad hardware dedicato come Raspberry Pi 5.
Ho cercato di includere i problemi riscontrati e le relative soluzioni.

📚**Indice dei contenuti**

Ciascuna guida è contenuta nella cartella /docs.  <br>
📄 [Step 1 – Ubuntu Server](docs/step1-UBUNTU-SERVER.md)  
📄 [Step 2 – Docker](docs/step2-DOCKER.md)  
📄 [Step 3 – Portainer](docs/step3-PORTAINER.md)  
📄 [Step 4 – AdGuard Home](docs/step4-ADGUARD.md)  
📄 [Step 5 – WireGuard](docs/step5-WIREGUARD.md)<br>
📄 [Step 6 – Raspberry Pi 5](docs/step6-RaspberryPi5.md)<br>
📄 [Step 7 – Caddy](docs/step7_CADDY.md)<br>
📄 [Step 8 – UFW](docs/step8_FIREWALL.md)<br>

**🧱 Obiettivi del progetto**

* Centralizzare servizi di rete in un ambiente dockerizzato
* Migliorare sicurezza e privacy tramite DNS filtering + VPN
* Preparare un’infrastruttura portabile verso Raspberry Pi
* Documentare tutto per poter replicare facilmente il sistema

🛠️ **Tecnologie utilizzate**

* Ubuntu Server 22.04+
* Docker Engine & Docker Compose
* Portainer CE
* AdGuard Home
* WireGuard
* VirtualBox / Raspberry Pi 5
