# ADGUARD configurazione con Portainer

Quando crei un container in Advanced container settings → Restart policy, le opzioni sono:

1 No	Il container non si riavvia mai automaticamente.
2 Always	Il container si riavvia sempre se si ferma, incluso al riavvio del server.
3 Unless-stopped	Si riavvia sempre, tranne se lo fermi manualmente.
4 On-failure	Si riavvia solo se il container termina con errore. Puoi anche impostare un numero massimo di tentativi.

Sia per Wireguard che ADguard, è consigliato scegliere opzione **unless-stopped o always**.

Problema: mi era impossibile deployare container su porta 53 perchè è già occupata in macchina Ubuntu server.

```bash
sudo netstat -tulnp | grep :53
```

**systemd-resolved** sta occupando la porta 53 sulle interfacce locali. E' un servizio di gestione della risoluzione dei nomi di dominio (DNS) presente nelle distribuzioni Linux moderne

soluzione:

**Disabilitare systemd-resolved**
```bash
sudo systemctl stop systemd-resolved
sudo systemctl disable systemd-resolved
```

**Sistemare /etc/resolv.conf**
Systemd-resolved usa un link simbolico per resolv.conf, quindi dobbiamo rimuoverlo e creare un file nuovo:

```bash
sudo rm /etc/resolv.conf
sudo bash -c "echo 'nameserver 1.1.1.1' > /etc/resolv.conf"
sudo systemctl restart docker     #Riavviare Docker (opzionale ma consigliato)
```

## ADGUARD

1)**Ho creato volumi per Adguard dal menù a sx di Portainer**

adguard_config
adguard_data

2)**Da Portainer --> stack --> nuovo stack --> incolla questo docker compose
```bash
version: "3.8"

services:
  adguardhome:
    image: adguard/adguardhome:latest
    container_name: adguardhome
    restart: always
    ports:
      - "3000:3000"          # porta solo per primo Setup 
      - "80:80"              # Dashboard web
      - "53:53/tcp"          # DNS TCP
      - "53:53/udp"          # DNS UDP
    volumes:
      - adguard_config:/opt/adguardhome/conf
      - adguard_data:/opt/adguardhome/work
    networks:
      - default

volumes:
  adguard_config:
  adguard_data:
```

![ADGUARD](../imgs/img3.png)

### Configurazione ADguard Home

http://IPSERVER:3000       

![ADGUARD](../imgs/img4.png)

Seguire la procedura guidata:
Lascia cartelle come sono (sono i due volumi)
Lascia porte 80 / 443
Crea login

Poi sarà possibile entrare normalmente in:
http://IPSERVER:80
Attiva HTTPS (direttamente da AdGuard più avanti)da settings → Encryption


**Se si ha un dominio:**
scegli Let’s Encrypt
inserisci dominio
inserisci email

**Se NON si ha un dominio:**
scegli Self-signed certificate

**Il container userà la porta:**
https://IPSERVER:8443

