# Configurazione firewall
mostra:
sudo ufw status verbose

se è active o inactive
policy di default
regole attive

Blocca tutto in ingresso e lascia uscire liberamente:
sudo ufw default deny incoming
sudo ufw default allow outgoing

Permetti solo i servizi che servono (Caddy come reverse-proxy)

sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

sudo ufw allow from 192.168.1.0/24 to any port 22 proto tcp


Regola SSH solo tramite VPN

Quando sei connesso alla VPN, il tuo client riceve un IP interno (es. 10.8.0.x).
Per far sì che solo chi è nella VPN possa fare SSH:

sudo ufw allow from 10.8.0.0/24 to any port 22 proto tcp

Spiegazione:
10.8.0.0/24 → subnet WireGuard
to any port 22 → accesso SSH
proto tcp → protocollo TCP

sudo ufw allow 53000/udpsudo ufw allow 53000/udp     #ipotizziamo una porta 53000 impostata nel port-forwarding del router
sudo ufw enable

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

UFW tiene un registro dei pacchetti bloccati o consentiti.

Livello “low” significa che logga solo pacchetti sospetti o bloccati, non ogni singolo pacchetto.
Permette di monitorare tentativi di intrusione o accessi sospetti senza saturare il disco.
