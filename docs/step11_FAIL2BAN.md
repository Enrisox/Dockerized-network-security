# Ulteriore hardening SSH server

## Installazione Fail2Ban

sudo apt update
sudo apt install fail2ban -y

**Fail2Ban usa file di configurazione in /etc/fail2ban/. Non modificare mai i file .conf direttamente; crea sempre override**

sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
sudo nano /etc/fail2ban/jail.local

**Cerca la sezione [DEFAULT] (circa riga 90-100) e modifica:**

[DEFAULT]
#Ban per 1 ora (3600 secondi)
bantime = 3600

#Finestra temporale di 10 minuti per contare i tentativi
findtime = 600

#Numero massimo di tentativi prima del ban
maxretry = 3

#Backend per leggere log (systemd per Ubuntu/Debian moderni)
backend = systemd

**Cerca la sezione [sshd] (circa riga 280-290) e verifica/modifica:**
[sshd]
enabled = true
port = ssh
logpath = %(sshd_log)s
backend = systemd
maxretry = 3
bantime = 3600
findtime = 600

**Abilita e avvia Fail2Ban:**
sudo systemctl enable fail2ban
sudo systemctl start fail2ban
sudo systemctl status fail2ban
