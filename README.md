# Dockerized-network-security

**Benvenuto nel mio progetto personale dedicato alla costruzione di un ambiente self-hosted, modulare e scalabile, basato su:**

* Ubuntu Server
* Docker 
* Portainer
* AdGuard Home (DNS filtering)
* WireGuard (VPN)
* Caddy (reverse proxy)
* Ip tables/UFW (firewall nativo Linux)
* Cloudflare DDNS
* Crowdsec & FAIL2BAN
* Netdata

Lo scopo della repository è documentare passo dopo passo la configurazione dell’intero sistema, in modo semplice, ripetibile e adatto sia ad ambienti VirtualBox sia ad hardware dedicato come Raspberry Pi o mini PC.
Ho cercato di includere i problemi riscontrati e le relative soluzioni.

📚**Indice dei contenuti**

📄 [Step 1 – Ubuntu Server](docs/step1-UBUNTU-SERVER.md)  
📄 [Step 2 – Docker](docs/step2-DOCKER.md)  
📄 [Step 3 – Portainer](docs/step3-PORTAINER.md)  
📄 [Step 4 – AdGuard Home](docs/step4-ADGUARD.md)  
📄 [Step 5 – WireGuard](docs/step5-WIREGUARD.md)<br>
📄 [Step 6 – Raspberry Pi 5](docs/step6-RaspberryPi5.md)<br>
📄 [Step 7 – Caddy](docs/step7_CADDY.md)<br>
📄 [Step 8 – UFW](docs/step8_FIREWALL.md)<br>
📄 [Step 9 – CROWDSEC](docs/step9_CROWDSEC.md)<br>
📄 [Step 10 – CLOUDFLARE](docs/step10_CLOUDFLARE.md)<br>
📄 [Step 11 – FAIL2BAN](docs/step11_FAIL2BAN.md)<br>
📄 [Step 12 – HARDENING](docs/step12_HARDENING_CONTAINERS.md)<br>


**🧱 Obiettivi del progetto**

* Centralizzare servizi di rete in un ambiente dockerizzato.
* Migliorare sicurezza e privacy tramite DNS filtering + VPN.
* Hostare un’ home server a casa.
* Documentare tutto per poter replicare facilmente il sistema.


