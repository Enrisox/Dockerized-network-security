# Configurazione WIREGUARD stack

1) crea volume wireguard_config

2) crea nuovo stack wireguard incollando questo file Yaml

```bash
version: "3.8"
services:
  wireguard:
    image: linuxserver/wireguard:latest
    container_name: wireguard
    cap_add:
      - NET_ADMIN
      - SYS_MODULE
    network_mode: host        # tipo di rete è host
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/Rome
      - SERVERURL= auto o mettere ip pubblico    # mettere ip pubblico!
      - SERVERPORT=******    #mettere porta a piacere  , es: 51040
      - PEERS=2           
      - PEERDNS=*******       #ip privato ubuntu server
      - INTERNAL_SUBNET=       #rete privata diversa da rete privata della lan      es: 10.10.10.0
      - LOG_CONFS=true         #•	LOG_CONFS=true farà apparire QR code nel log  . Per esperienza non molto affidabile
    volumes:
      - wireguard_config:/config    #contiene le chiavi private dei peer
      -/lib/modules:/lib/modules
    restart: unless-stopped

    ports:
      - 50000:50000/udp        #la porta che abbiamo scelto
    sysctls:
      - net.ipv4.conf.all.src_valid_mark=1
    restart: unless-stopped

volumes:
  wireguard_config:
    external: true

```

Per concludere, inquadrare QR code su app Wireguard del dispositivo mobile e controllare che i dati siano corretti( ip e porte) 

Sul router è necessario configurare il port forwarding della porta scelta, sul router di casa!

## DDNS in caso di ip dinamico

Dato che mio ISP mi assegna un IP diverso ad ogni ravvio del "router" di casa, ho creato un dominio DDNS con un piano gratuito e sostituito l'endpoint con il dominio suddetto in modo che se l'ip varia, non dovrò modifcare manualmente l'ip dell'endpoint dal mio dispositivo mobile. <br>

Inoltre, per avere maggiore flessibilità e sottodomini illimitati, ho creato un dominio DuckDNS gratuito. DuckDNS fornisce un nome a dominio come enrisox.duckdns.org e permette di aggiungere più sottodomini senza limiti. Il Raspberry aggiorna automaticamente l'IP pubblico associato a questo dominio tramite un container DuckDNS o uno script cron, in modo che tutti i servizi ospitati sul dispositivo rimangano sempre raggiungibili anche se l'IP cambia.

Questo approccio permette di centralizzare la gestione dei sottodomini per i vari servizi, come web app o server interni, e semplifica la configurazione di un reverse proxy come Nginx Proxy Manager, che instraderà ogni sottodominio al container corretto mantenendo la sicurezza e il supporto per HTTPS tramite certificati Let’s Encrypt.

```bash
version: "3.9"

services:
  duckdns:
    image: linuxserver/duckdns
    container_name: duckdns
    environment:
      - PUID=1000              # lo trovi dal comando id <tuo user>
      - PGID=1000              # lo trovi dal comando id <tuo user>
      - TZ=Europe/Rome
      - SUBDOMAINS=enrisox     # tuo dominio principale
      - TOKEN=IL_TUO_TOKEN_DUCKDNS          #lo trovi su duckdns.org nella sezione profilo
    restart: unless-stopped
```
