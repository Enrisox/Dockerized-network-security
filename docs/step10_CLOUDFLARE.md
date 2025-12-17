# Cloudflare come edge reverse proxy


## Difese del piano free incluse

1. Bot Fight Mode attivo con JS Detections: On: Cloudflare identifica traffico automatico e può applicare challenge “computazionali” ai bot, e la parte JS Detections gira su ogni richiesta per aiutare a riconoscere automazione.​
2. Browser Integrity Check: valuta header HTTP e può bloccare richieste considerate sospette.​
3. Challenge Passage 30 min: dopo che un client supera una challenge, resta “validato” per 30 minuti prima di riceverne un’altra.​
4. HTTP DDoS + Network-layer DDoS protection “Always active”: mitigazioni automatiche per DDoS HTTP e di rete sempre abilitate.​
5. Cloudflare managed ruleset (WAF) “Always active”: un ruleset gestito da Cloudflare contro exploit comuni; puoi entrare in “View ruleset” per vedere/abilitare regole in base al tuo stack.​
6. AI Labyrinth: aggiunge link “nofollow” invisibili ai bot e li intrappola per disturbare crawler che ignorano gli standard; inoltre aiuta a identificarli.​

Per vedere eventi reali (block/challenge e motivo), Security → Events (nel nuovo flusso spesso è “Security Analytics / Events”).

## Cloudflare configuration
Dal fornitore del dominio, nella sezione DNS settings:

![lingua italiana](../imgs/img5.png)

DNSSEC deve essere disattivato.
In Name server ho inserito i nomi host indicati da Cloudflare e salvato. D'ora in avanti gestirà tutto Cloudflare e non sarà necessario accedere al portale del fornitore di dominio.
Creare i record A DNS su cloudflare, tutti puntati al mio ip pubblico e con proxy status: proxied(nuvola arancione)
Creazione di un token API Cloudflare con permessi zone DNS:Edit per il proprio dominio, necessaario per il DDNS.

## Gestione IP dinamico con container Cloudflare DDNS
Il mio container Cloudflare DDNS si occupa di tenere aggiornati automaticamente i record DNS di Cloudflare con il mio IP pubblico reale, così i tuoi domini/subdomini puntano sempre al tuo Raspberry anche se l’ISP cambia IP.​

1. Interroga periodicamente il mio IP pubblico (IPv4 e/o IPv6) tramite servizi esterni o DNS‑over‑HTTPS.​
2. Confronta l’IP trovato con quello registrato nei record A/AAAA su Cloudflare per i domini che gli hai configurato.​
3. Se l’IP è cambiato, chiama le API Cloudflare e aggiorna i record DNS (A/AAAA) dei miei domini/subdomini.

**Dockerfile del container**
```bash
version: "3"

services:
  cloudflare-ddns:
    image: favonia/cloudflare-ddns:latest
    container_name: cloudflare-ddns
    restart: unless-stopped
    environment:
      # token appena creato
      - CF_API_TOKEN=TOKEN API CLOUDFLARE
      # zona Cloudflare
      - CF_ZONE=dominio personale
      - DOMAINS=dominio e sottodominii
      - CF_TTL=auto
      - IP_VERSION=4
      - IP6_PROVIDER=none
```

