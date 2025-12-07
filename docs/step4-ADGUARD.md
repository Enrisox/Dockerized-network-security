# ADGUARD configurazione con Portainer

Quando crei un container in Advanced container settings → Restart policy, le opzioni sono:

* No	Il container non si riavvia mai automaticamente.
* Always	Il container si riavvia sempre se si ferma, incluso al riavvio del server.
* Unless-stopped	Si riavvia sempre, tranne se lo fermi manualmente.
* On-failure	Si riavvia solo se il container termina con errore. Puoi anche impostare un numero massimo di tentativi.

Sia per Wireguard che ADguard, è consigliato scegliere opzione **unless-stopped o always**.

Problema: mi era impossibile deployare container su porta 53 perchè è già occupata in macchina Ubuntu server.

```bash
sudo netstat -tulnp | grep :53
```

**systemd-resolved** sta occupando la porta 53 sulle interfacce locali. E' un servizio di gestione della risoluzione dei nomi di dominio (DNS) presente nelle distribuzioni Linux moderne

**Soluzione:**

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

adguard_config <br>
adguard_data

2)**Da Portainer --> stack --> nuovo stack --> incolliamo il docker compose desiderato**
```bash
version: "3"
services:
  adguardhome:
    image: adguard/adguardhome:latest
    container_name: adguardhome
    restart: unless-stopped
    ports:
      - "53:53/tcp"
      - "53:53/udp"
      - "8092:80/tcp"
      - "3000:3000/tcp"
      - "4464:443/tcp"
      - "4464:443/udp"
      - "853:853/tcp"
      - "853:853/udp"
    volumes:    #I dati del container montati su questi volumi saranno persistenti, anche se il container viene eliminato.
      - adguard_work:/opt/adguardhome/work  
      - adguard_conf:/opt/adguardhome/conf

volumes:    #Docker crea automaticamente dei volumi gestiti internamente
  adguard_work:
  adguard_conf:

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

### Da windows e dispositivi mobile ho impostato il container Adguard come server DNS preferito

![ADGUARD](../imgs/img6.png)

**wind+R → ncpa.cpl → proprietà → ipv4 → imposta dns manuale**

#### Per far funzionare ADguard anche con dispositivi mobile collegati in VPN è necessario modificare il file AdGuardHome.yaml mettendo in bind_host: 0.0.0.0
```bash
find / -name AdGuardHome.yaml    #per trovare file
```
![ADGUARD](../imgs/img7.png)

```bash
docker restart adguardhome
```

Per far funzionare Adguard come DNS può essere necessario collegarsi a proprio router di casa e disattivare l'impostazione che impone il DNS server del proprio internet provider su tutti i dispositivi connessi, rendendo di fatto possibile così usare ADguard.

Dopo averlo fatto, controllare su siti come DNSleak.com se è presente un dns dell'ISP o i vari dns server cloudflare di Adguard.
