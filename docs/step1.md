# Preparazione del nostro ambiente virtuale su virtualbox

Come primo step ho preparato una VM su virtual Box, sul mio PC desktop, ma in futuro sposterò tutta la struttura su raspberry 5 da 8 gb

Ho installato Ubuntu server sulla VM, dopo aver scelto lingua inglese ( quella italiana non è disponibile durante installazione, la installeremo successivamente dal terminale.
nessun servizio aggiuntivo selezionato eccetto il server SSH. scheda in bridge da impostazioni virtual box.

Una volta conclusa installazione ho proceduto ad aggiornare i paccchetti con

```bash
sudo apt update
sudo apt upgrade
ip a     # per vedere l'ip della VM e collegarmi in ssh da mia macchina host 
```
ssh non funzionava perchè il server ssh non trovava le chiavi host necessarie a far partire il servizio ssh.
Con un po' di troubleshooting ho trovato la soluzione:

Quando avvii un server SSH, il server ha bisogno di chiavi host per poter stabilire connessioni sicure con i client SSH. Le chiavi host sono un insieme di chiavi crittografiche che identificano in modo univoco il server e vengono utilizzate per stabilire un canale sicuro durante la connessione SSH.

```bash
systemctl status ssh 
#no hostkeys available – exiting
```

**no hostkeys available - exiting** significa che il server SSH non riesce a trovare le chiavi host necessarie per avviarsi correttamente. Senza queste chiavi, sshd (il demone SSH) non può avviarsi e rifiuterà le connessioni in entrata.

Le chiavi host vengono generate automaticamente quando il server SSH viene installato o configurato per la prima volta, ma se mancano (ad esempio, dopo una nuova installazione o la rimozione accidentale), sshd non può avviarsi e darà questo errore.

```bash
sudo ssh-keygen -A          # -A dice a ssh-keygen di generare tutte le chiavi host mancanti (rsa, ecdsa, ed25519, dsa).
sudo systemctl restart ssh     #restartiamo il server ssh
sudo systemctl status ssh       #ora si dovrebbe vedere active (running)
```

