# Installazione e configurazione Crowdsec

**CrowdSec** è un software di sicurezza open source che monitora i log (es. SSH e web server), riconosce pattern di attacco (come brute force e scansioni) e poi applica contromisure, tipicamente bloccando gli IP malevoli tramite componenti di “enforcement” (bouncer) o integrazioni con firewall/proxy


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
