# Installazione e configurazione Crowdsec

**CrowdSec** è un software di sicurezza open source che monitora i log (es. SSH e web server), riconosce pattern di attacco (come brute force e scansioni) e poi applica contromisure, tipicamente bloccando gli IP malevoli tramite componenti di “enforcement” (bouncer) o integrazioni con firewall/proxy.
CrowdSec protegge il mio server in due modi: blocca proattivamente IP noti malevoli (community blocklist) e reagisce, ai comportamenti sospetti che vede nei miei log, generando ban (temporanei o permanenti).

## Aggiungere il repository ufficiale

```dockerfile
curl -s https://packagecloud.io/install/repositories/crowdsec/crowdsec/script.deb.sh | sudo bash

#The repository is setup! You can now install packages.
```

## Installare CrowdSec + lettura log

```dockerfile
sudo apt install -y crowdsec crowdsec-firewall-bouncer-iptables

sudo reboot          #quando finisce

sudo systemctl enable crowdsec
sudo systemctl start crowdsec
sudo systemctl status crowdsec

```
Se lo stato è enabled, e active, procediamo.

## Verificare che il motore funzioni

Mostra statistiche live
Es: pacchetti analizzati, ban attivi, alert

```dockerfile
sudo cscli metrics
```

## Mostra quali parser sono disponibili
I parser (in CrowdSec) sono “regole” descritte in file YAML che spiegano come interpretare una stringa di log (una riga di log, o un campo estratto da un parser precedente) e trasformarla in campi strutturati su cui poi lavorano gli scenari.

```dockerfile
sudo cscli parsers list
```


## Mostra lista scenarios
```dockerfile
sudo cscli scenarios list
```

Gli “scenari” in CrowdSec sono le regole di rilevamento comportamento: definiscono una euristica (in YAML) che correla eventi già parsati (es. tanti tentativi di login falliti da uno stesso IP in poco tempo) e decide quando quell’attività è sospetta/attacco.
Gli scenari confrontano i log normalizzati dai parser con soglie/finestre temporali (count, timeframe, ecc.) per riconoscere pattern come brute force, scan, crawl aggressivi

Confrontano i log normalizzati dai parser con soglie/finestre temporali (count, timeframe, ecc.) per riconoscere pattern come brute force, scan, crawl aggressivi.

Quando uno scenario “scatta”, CrowdSec genera un alert (per tracciabilità) e può generare una o più decisions (es. ban temporaneo di un IP), che poi verranno applicate da un bouncer come iptables

## log live
```dockerfile
sudo journalctl -u crowdsec -f
```

## Bouncer Crowdsec
Non è servito impostare manualmente la chiave perché il bouncer iptables, installato via apt, può auto‑registrarsi sulla Local API quando CrowdSec è sulla stessa macchina e generare da solo la API key (di solito scrivendola in un file .yaml.local)
```dockerfile
sudo cat /etc/crowdsec/bouncers/crowdsec-firewall-bouncer.yaml.local
```

Il **bouncer** di CrowdSec è il componente che mette in pratica le decisioni prese dal motore CrowdSec (ban, captcha, ecc.): il motore rileva l’attacco dai log e salva una “decisione” nella Local API, mentre il bouncer interroga la Local API e applica la contromisura nel punto giusto (firewall, Cloudflare, reverse proxy, ecc.)
Nel mio caso, il crowdsec-firewall-bouncer è un bouncer “firewall”: applica i ban a livello di iptables/nftables sul server, bloccando il traffico in ingresso dagli IP “cattivi”.​

Differenza:
- **CrowdSec (engine/agent)**: analizza log → genera alert/decisioni. <br>​
- **Bouncer**: legge decisioni dalla Local API (autenticandosi con una API key) → le applica (blocca).

## Vedere IP bannati (CrowdSec)
Per vedere gli IP che CrowdSec ha deciso di bannare (indipendentemente da iptables), usa:
La **decision list** è il comando che mostra le “decisioni” attive salvate nella Local API di CrowdSec: in pratica l’elenco delle azioni da applicare e su quale target (IP, range, etc)
```dockerfile
sudo cscli decisions list

sudo ipset list crowdsec-blacklists-0 | head -n 30       #Per elencare gli IPv4 attualmente nel set (quelli che iptables sta droppando)

sudo ipset list crowdsec6-blacklists-0 | head -n 20      #Per elencare gli IPv6 attualmente nel set (quelli che iptables sta droppando):

```

