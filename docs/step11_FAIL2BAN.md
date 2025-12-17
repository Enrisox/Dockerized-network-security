# Ulteriore hardening SSH server
Fail2Ban e CrowdSec possono coesistere senza conflitti:
- Fail2Ban: reattivo e rapido su SSH (ban locale immediato).
- CrowdSec: analisi più complessa, community blocklist, decisioni condivise
- 
## Installazione Fail2Ban
```bash
sudo apt update
sudo apt install fail2ban -y
```

**Fail2Ban usa file di configurazione in /etc/fail2ban/. Non modificare mai i file .conf direttamente; crea sempre override**
```bash
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
sudo nano /etc/fail2ban/jail.local
```

**Cerca la sezione [DEFAULT] (circa riga 90-100) e modifica:**
```bash

[DEFAULT]
#Ban per 1 ora (3600 secondi)
bantime = 3600

#Finestra temporale di 10 minuti per contare i tentativi
findtime = 600

#Numero massimo di tentativi prima del ban
maxretry = 3

#Backend per leggere log (systemd per Ubuntu/Debian moderni)
backend = systemd
```

**Cerca la sezione [sshd] (circa riga 280-290) e verifica/modifica:**
```bash
[sshd]
enabled = true
port = ssh
logpath = %(sshd_log)s
backend = systemd
maxretry = 3
bantime = 3600
findtime = 600
```
**Abilita e avvia Fail2Ban:**
```bash
sudo systemctl enable fail2ban
sudo systemctl start fail2ban
sudo systemctl status fail2ban
```
### comandi utili
```bash
sudo fail2ban-client status               #Verifica jail SSH attiva
sudo fail2ban-client status sshd         #Vedere IP attualmente bannati
sudo fail2ban-client set sshd unbanip <IP>  #Sbannare manualmente un IP (se ti blocchi per errore)
sudo fail2ban-client set sshd banip <IP>    #Bannare manualmente un IP
sudo tail -f /var/log/fail2ban.log          #Log Fail2Ban
```
