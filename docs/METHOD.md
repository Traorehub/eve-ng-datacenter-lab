# Démarche

## Construction

1. Toile EVE-NG, nœuds éteints, collisions lues (un Cloud d'admin par GUI, pas un spaghetti central).
2. Commutateurs : VLAN, trunks, EtherChannel, STP. Ping L2 avant tout NAT.
3. SFOS : VLAN d'accès, ACL, passerelles. Un VPC par VLAN doit ping sa passerelle.
4. PAN-OS : zones, NAT, une règle par flux Nord. Commit.
5. HAProxy : frontend / backend, démonstration `curl` alterné.
6. BIG-IP : Self IP, route, pool, Auto Map, `vs_web`.
7. AD : forêt `pfa.local`, DNS intégré, compte de test.
8. APM : AAA AD, policy, cookie non Secure (VIP HTTP).
9. ASM : policy REST, Transparent sur `vs_web`, Blocking + `apply-policy` sur `vs_dvwa`.
10. strongSwan : X-Auth, Mode Config, SNAT VPN-RA, démonstration `ping -I 10.66.80.x`.

## Démonstrations minimales

| Test | Attendu |
|------|---------|
| `curl` HAProxy deux fois | web1 puis web2 |
| GET `http://10.66.10.100/` | 302 `/my.policy`, formulaire APM |
| SQLi / XSS sur `10.66.10.101` | Support ID, Event Log Blocking |
| GET `http://10.66.50.82/dvwa/` | application joignable sans ASM |
| `ipsec up` Alpine | `installing new virtual IP`, `ESTABLISHED` |

Sans `apply-policy`, TMM garde l'ancienne policy ASM. Sans règle Palo vers le VIP DVWA, le Blocking n'a pas de trafic à montrer.
