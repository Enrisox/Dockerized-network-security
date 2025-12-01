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
    network_mode: host        # tipo di rete è host
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/Rome
      - SERVERURL= *****    # mettere ip pubblico!
      - SERVERPORT=******    #mettere porta a piacere  , es: 51040
      - PEERS=1           
      - PEERDNS=*******       #ip ubuntu server
      - INTERNAL_SUBNET=       #ip/subnet in cidr   , diversa da rete privata della lan
      - LOG_CONFS=true         #•	LOG_CONFS=true farà apparire QR code nel log  . Per esperienza non molto affidabile
    volumes:
      - wireguard_config:/config
    restart: unless-stopped

volumes:
  wireguard_config:
```
