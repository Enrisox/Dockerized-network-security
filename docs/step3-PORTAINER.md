# Installare Portainer per gestire i container da interfaccia grafica

Portainer permette di creare, avviare, fermare, configurare e monitorare i container Docker direttamente attraverso una dashboard web. Piuttosto che digitare comandi nella shell, possiamo interagire con i nostri container attraverso un'interfaccia grafica user-friendly.

**1)Per iniziare ho creato il volume**

```bash
sudo docker volume create portainer_data
```

**2)Ho avviato Portainer**

```bash
sudo docker run -d \
  -p 8000:8000 \
  -p 9443:9443 \
  --name portainer \
  --restart=always \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v portainer_data:/data \
  portainer/portainer-ce:latest
```
* sudo docker run -d: Avvia Portainer in detached mode.  Questo significa che il terminale non rimarrà bloccato e potrai continuare a lavorare mentre il container è in esecuzione
* -p 8000:8000 -p 9443:9443: Mappa le porte 8000 (HTTP) e 9443 (HTTPS) per l'accesso al servizio web di Portainer.
* --name portainer: Nomina il container come "portainer".
* --restart=always: Riavvia automaticamente Portainer se il container si ferma o se il sistema viene riavviato.
* -v /var/run/docker.sock:/var/run/docker.sock: Consente a Portainer di comunicare con Docker sull'host per la gestione dei container.
* -v portainer_data:/data: Salva i dati di configurazione di Portainer in un volume persistente.
* portainer/portainer-ce:latest: Usa l'ultima versione di Portainer Community Edition.

3)**ho aperto Portainer da browser web in versione https**

```bash
https://IP-DELLA-VM:9443      # HTTPS se sulla 9443, altrimenti 8000 con http
```

![Portainer](../imgs/img2.png)

Portainer è pronto all'uso quindi procediamo al prossimo step, il 4, dove installerò container Wireguard per VPN e Adguard come filtro tracking e advertising.
