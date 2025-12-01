# Installazione Docker 

**Per cominciare è necessario:
1)installare i prerequisiti**

```bash
sudo apt install ca-certificates curl gnupg -y
```

•	ca-certificates → serve a Ubuntu per gestire i certificati SSL/TLS. Necessario per scaricare pacchetti sicuri via HTTPS.
•	curl → strumento per scaricare file da internet da linea di comando.
•	gnupg → permette di gestire chiavi GPG, utili per verificare l’autenticità dei pacchetti.
•	-y → conferma automatico, evita che ti chieda “Vuoi continuare?”.

**2)Aggiungere la GPG key di Docker**

```bash
sudo install -m 0755 -d /etc/apt/keyrings
```

•	Crea la cartella /etc/apt/keyrings con permessi 0755 (leggibile/eseguibile da tutti, scrivibile solo dal proprietario).
•	Serve per mettere la chiave GPG in un posto sicuro e standard.

```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg \
 | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```
•	Scarica la chiave pubblica GPG di Docker.
•	--dearmor converte la chiave in un formato leggibile da apt.
•	La salva in /etc/apt/keyrings/docker.gpg.

```bash
sudo chmod a+r /etc/apt/keyrings/docker.gpg         #Assicura che tutti possano leggere la chiave (necessario per apt).
```

**3)Aggiungere il repository Docker**
```bash
echo \
"deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
 https://download.docker.com/linux/ubuntu \
 $(. /etc/os-release && echo $VERSION_CODENAME) stable" \
 | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```
*	dpkg --print-architecture → restituisce la tua architettura (amd64, arm64, ecc.).
*	$(. /etc/os-release && echo $VERSION_CODENAME) → prende il nome della tua versione Ubuntu (es. focal, jammy).
*	Il tutto scrive in /etc/apt/sources.list.d/docker.list una riga che dice a Ubuntu da dove scaricare Docker.
*	signed-by=/etc/apt/keyrings/docker.gpg → specifica di usare la chiave appena scaricata.

**4)installare Docker**
```bash
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
```
•	docker-ce → il “Docker Engine” vero e proprio.
•	docker-ce-cli → l’interfaccia a riga di comando (docker ...).
•	containerd.io → gestore dei container sottostante.
•	docker-buildx-plugin → estensione per build avanzate.
•	docker-compose-plugin → permette di usare docker compose
```bash
docker --version     #Mostra la versione installata di Docker 
```

**5)Opzionale- eseguire comandi Docker senza "sudo"**
```bash
sudo groupadd docker           # crea il gruppo 'docker', se non esiste
sudo usermod -aG docker $USER  # aggiungi il tuo utente al gruppo
newgrp docker                  # applica subito i cambiamenti senza logout
docker run hello-world        #se funziona senza sudo, è tutto pronto.
```



