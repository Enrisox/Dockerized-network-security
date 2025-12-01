# WIREGUARD VPN configurazione con Portainer

Quando crei un container in Advanced container settings → Restart policy, le opzioni sono:

1 No	Il container non si riavvia mai automaticamente.
2 Always	Il container si riavvia sempre se si ferma, incluso al riavvio del server.
3 Unless-stopped	Si riavvia sempre, tranne se lo fermi manualmente.
4 On-failure	Si riavvia solo se il container termina con errore. Puoi anche impostare un numero massimo di tentativi.

Sia per Wireguard che ADguard più in seguito, è consigliato scegliere opzione **unless-stopped**.

Problema: 

sudo netstat -tulnp | grep :53
