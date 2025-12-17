# Cloudflare come edge reverse proxy
Cloudflare e Caddy svolgono entrambi funzione di reverse proxy, ma su strati e domini di responsabilità diversi: Cloudflare opera come proxy “edge” (Internet-facing), Caddy come proxy “origin” (in LAN/host) che effettua routing verso i container e nasconde dietro di se' i containers hostati.

## Difese del piano free incluse

1. Bot Fight Mode attivo con JS Detections: On: Cloudflare identifica traffico automatico e può applicare challenge “computazionali” ai bot, e la parte JS Detections gira su ogni richiesta per aiutare a riconoscere automazione.​
2. Browser Integrity Check: valuta header HTTP e può bloccare richieste considerate sospette.​
3. Challenge Passage 30 min: dopo che un client supera una challenge, resta “validato” per 30 minuti prima di riceverne un’altra.​
4. HTTP DDoS + Network-layer DDoS protection “Always active”: mitigazioni automatiche per DDoS HTTP e di rete sempre abilitate.​
5. Cloudflare managed ruleset (WAF) “Always active”: un ruleset gestito da Cloudflare contro exploit comuni; puoi entrare in “View ruleset” per vedere/abilitare regole in base al tuo stack.​
6. AI Labyrinth: aggiunge link “nofollow” invisibili ai bot e li intrappola per disturbare crawler che ignorano gli standard; inoltre aiuta a identificarli.​

**Bot Fight Mode + JS Detections**
Bot Fight Mode è un componente di bot mitigation a livello L7 che classifica il traffico HTTP come “probabilmente umano” vs “probabilmente automatizzato” sulla base di segnali comportamentali e di fingerprinting.
Con JS Detections: On Cloudflare inserisce una logica di rilevamento lato client (esecuzione JavaScript) per verificare che il richiedente si comporti come un browser reale; molti tool di scanning/scraping non eseguono JS o lo eseguono in modo anomalo.
In base al livello di confidenza Cloudflare può applicare azioni di mitigazione (challenge/managed challenge) o lasciar passare la richiesta.

**Browser Integrity Check**
È un controllo euristico sui metadati della richiesta HTTP (principalmente header e pattern tipici di client non conformi) per intercettare traffico “non-browser” o malevolo a bassa sofisticazione.
Quando rileva anomalie, può servire una pagina di blocco/challenge invece di inoltrare la richiesta all’origin.
Non è un WAF completo: è un filtro rapido basato su segnali superficiali, utile contro tooling banale e noise.

**Challenge Passage (30 min)**
È il TTL della “validazione” post-challenge: se un client supera una challenge (JavaScript/Interactive/Managed), Cloudflare emette un token/cookie di attestazione.
Per i successivi 30 minuti (nel tuo caso) le richieste dello stesso client possono essere accettate senza ripresentare la challenge, riducendo latenza e attrito per utenti legittimi.
Scaduto il TTL, il client può essere nuovamente sottoposto a challenge in base al rischio/ruleset.

**HTTP DDoS + Network-layer DDoS protection (Always active)**
Sono mitigazioni anti-DDoS “always-on” operate sull’infrastruttura edge di Cloudflare.

Network-layer (L3/L4): protezione contro flood e attacchi di trasporto (es. SYN/UDP/ICMP flood), con filtraggio e rate enforcement a livello rete prima che il traffico raggiunga il tuo IP di origin.

HTTP DDoS (L7): protezione contro attacchi che saturano l’applicazione con richieste HTTP (pattern ripetitivi, burst su endpoint, volumetrie anomale), con identificazione e mitigazione basata su segnali applicativi.
Il punto chiave è che queste mitigazioni avvengono prima dell’origin, preservando banda e risorse del tuo server.

**Cloudflare Managed Ruleset (WAF) (Always active)**
È un insieme di regole WAF L7 mantenute da Cloudflare per intercettare pattern di exploit comuni (request anomalies, payload tipici, endpoint noti, categorie di vulnerabilità) prima di inoltrare all’origin.
“Always active” indica che il ruleset è abilitato in modo predefinito; a seconda del piano puoi avere più/meno capacità di tuning (es. sensibilità, eccezioni, override per hostname/path, logging).
La logica è: valutazione richiesta → match su regole → azione (log, challenge, block) → eventualmente forward verso origin.

**AI Labyrinth**
È una misura anti-scraping/anti-crawler “non cooperativo”: Cloudflare inserisce nel markup link rel="nofollow" che puntano a contenuti generati e non rilevanti, tipicamente invisibili o irrilevanti per l’utente, ma “attrattivi” per crawler che ignorano gli standard.
Lo scopo operativo è: consumare risorse del bot, degradare la qualità dello scraping e aumentare la capacità di identificazione/attribuzione di crawler aggressivi.
Non è una protezione “anti-exploit” (non sostituisce WAF), ma una mitigazione specifica contro data harvesting e crawling non conforme.


Per vedere eventi reali (block/challenge e motivo), Security → Events (nel nuovo flusso spesso è “Security Analytics / Events”).

## Cloudflare configuration
Dal fornitore del dominio, nella sezione DNS settings:

![lingua italiana](../imgs/img5.png)

DNSSEC deve essere disattivato. <br>
In Name server ho inserito i nomi host indicati da Cloudflare e salvato.<br> D'ora in avanti gestirà tutto Cloudflare e non sarà necessario accedere al portale del fornitore di dominio.<br>
Ho poi creato i **record A DNS** su Cloudflare, tutti puntati al mio ip pubblico e con proxy status: proxied(nuvola arancione) e creato un token API Cloudflare con permessi zone DNS:Edit per il proprio dominio, necessario per il DDNS.

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

