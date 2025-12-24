# Preparazione dell'ambiente virtuale su virtualbox

Come primo step ho preparato una VM su virtual Box, sul mio PC desktop, ma in futuro sposterò tutta la struttura su UN Raspberry pi 5 da 8 gb.

Ho installato Ubuntu server sulla VM, dopo aver scelto lingua inglese( quella italiana non è disponibile durante installazione, la installeremo successivamente dal terminale).<br>
Nessun servizio aggiuntivo selezionato eccetto il server SSH.  <br>
Scheda in bridge da impostazioni virtual box.<br>

**Una volta conclusa installazione ho proceduto ad aggiornare i pacchetti con:**

```bash
sudo apt update
sudo apt upgrade
ip a     # per vedere l'ip della VM e collegarmi in ssh da mia macchina host 
```
**Ssh non funzionava perchè il server ssh non trovava le chiavi host necessarie a far partire il servizio ssh.
Con un po' di troubleshooting ho trovato la soluzione:**

**Quando avvii un server SSH, il server ha bisogno di chiavi host per poter stabilire connessioni sicure con i client SSH. Le chiavi host sono un insieme di chiavi crittografiche che identificano in modo univoco il server e vengono utilizzate per stabilire un canale sicuro durante la connessione SSH.**

```bash
systemctl status ssh 
#no hostkeys available – exiting
```

**No hostkeys available - exiting** significa che il server SSH non riesce a trovare le chiavi host necessarie per avviarsi correttamente. Senza queste chiavi, sshd (il demone SSH) non può avviarsi e rifiuterà le connessioni in entrata.

**Le chiavi host vengono generate automaticamente quando il server SSH viene installato o configurato per la prima volta, ma se mancano (ad esempio, dopo una nuova installazione o la rimozione accidentale), sshd non può avviarsi e darà questo errore.**

```bash
sudo ssh-keygen -A          # -A dice a ssh-keygen di generare tutte le chiavi host mancanti (rsa, ecdsa, ed25519, dsa).
sudo systemctl restart ssh     #restartiamo il server ssh
sudo systemctl status ssh       #ora si dovrebbe vedere active (running)
```

# Configurazione della tastiera e lingua italiana su Ubuntu Server
Ho impostato la lingua italiana impostando la tastiera italiana da riga di comando Ubuntu server

**Per configurare correttamente la tastiera, ho eseguito il seguente comando:**

```bash
sudo dpkg-reconfigure keyboard-configuration
```
**Durante la procedura, scegliere le seguenti opzioni:**

Keyboard model: Generic 105-key PC (o il modello che corrisponde alla tua tastiera)
Country of origin: Italy
Keyboard layout: Italian
AltGr: default
Compose key: None
Ctrl+Alt+Backspace: No

**Dopo aver configurato la tastiera, bisogna ricaricare la configurazione con:**

```bash
sudo service keyboard-setup restart
sudo setupcon
```
**Per aggiungere la lingua italiana e impostare l’italiano come locale:**

```bash
sudo apt install language-pack-it   
sudo update-locale LANG=it_IT.UTF-8
source /etc/default/locale         #ricaricare
locale       #verifica
```
![lingua italiana](../imgs/img1.png)



# Assegnare IP statico a Ubuntu server

Un file Netplan è un file YAML che definisce la configurazione della rete su Ubuntu e altre distribuzioni Linux moderne. Serve a configurare interfacce, IP, gateway, DNS e altre impostazioni di rete, sostituendo i vecchi metodi come /etc/network/interfaces.

50-cloud-init.yaml è il file generato automaticamente da cloud-init durante l’installazione del mio Ubuntu Server.

Il numero 50 indica che è letto dopo eventuali file “base” (ad esempio 01-netcfg.yaml) e può quindi sovrascriverne alcune impostazioni.

**Se vuoi modificare l’IP statico, conviene modificare questo file oppure creare un nuovo file con un numero più alto (es. 99-custom.yaml) per sovrascrivere le impostazioni senza toccare l’originale.**

1)**Io ho prima fatto un backup del file originale con:**
```bash
sudo cp /etc/netplan/50-cloud-init.yaml /etc/netplan/50-cloud-init.yaml.bak
```

2)**modificato il file con:**
```bash
sudo nano /etc/netplan/50-cloud-init.yaml
```
```bash
network:
    version: 2
    ethernets:
        enp0s3:     #Controllare prima con ip a il nome dell’interfaccia es:eth0 per porta ethernet Raspberry o wlan0 per quella wifi
            dhcp4: no
            addresses:
              - 192.168.1.20/24
            gateway4: 192.168.1.1
            nameservers:
                addresses:
                  - 1.1.1.1
                  - 8.8.8.8
```
•	192.168.1.20 # l’IP che vogliamo assegnare al server
•	192.168.1.1 → il gateway della nostra LAN


3)**Infine ho applicato la configurazione netplan**
```bash
sudo netplan apply
ip a show enp0s3
```



**Nel prossimo step mostrerò l'installazione di Docker sulla VM Ubuntu server.**



