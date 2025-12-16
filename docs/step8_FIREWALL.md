# Configurazione firewall UFW
Comando per mostrare stato firewall UFW Linux:
```bash
sudo ufw status verbose
```

**Ho aggiunto le rules per bloccare tutto in ingresso e lasciare uscire traffico liberamente:**
```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

**Ho permesso solo i servizi che servono: Caddy e Wireguard**
```bash
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw allow 53000/udpsudo ufw allow 53000/udp     #ipotizziamo una porta 53000 impostata nel port-forwarding del router
sudo ufw allow from 192.168.4.0/24 to any port 22 proto tcp       #accesso in SSH da dispositivi nella LAN
```
## Regola SSH solo tramite VPN
Quando sei connesso alla VPN, il tuo client riceve un IP interno (es. 10.8.0.x).
Per far sì che solo chi è nella VPN possa fare SSH:

```bash
sudo ufw allow from 10.8.0.0/24 to any port 22 proto tcp
```

Spiegazione:
10.8.0.0/24 → subnet WireGuard
to any port 22 → accesso SSH
proto tcp → protocollo TCP

**Attiviamo firewall
```bash
sudo ufw enable
```

```bash
Status: active
Logging: on (low)
Default: deny (incoming), allow (outgoing), deny (routed)
New profiles: skip

To                         Action      From
--                         ------      ----
80/tcp                     ALLOW IN    Anywhere
443/tcp                    ALLOW IN    Anywhere
22/tcp                     ALLOW IN    rete_privata-del-server/S.M
22/tcp                     ALLOW IN    rete-privata-virtuale(VPN)/S.M
80/tcp (v6)                ALLOW IN    Anywhere (v6)
443/tcp (v6)               ALLOW IN    Anywhere (v6)
53000/udp (v6)             ALLOW IN    Anywhere (v6)       #porta della VPN
```
UFW tiene un registro dei pacchetti bloccati o consentiti.
Livello “low” significa che logga solo pacchetti sospetti o bloccati, non ogni singolo pacchetto.
Permette di monitorare tentativi di intrusione o accessi sospetti senza saturare il disco.

## Riassunto
Il firewall è configurato con politica di default deny in ingresso e routed, permettendo solo il traffico necessario ai servizi (SSH LAN/VPN, Web, WireGuard), mentre il traffico in uscita è libero. Questo approccio riduce drasticamente la superficie di attacco e garantisce monitoraggio tramite log

Meglio concentrarmi su potenziali attacchi dall’esterno, e filtrare solo pacchetti provenienti da IP fuori dalla mia LAN
```bash
sudo grep "BLOCK" /var/log/ufw.log | grep -v "192.168.4."           #se server è sulla 192.168.4.0/24 ad esempio
```
