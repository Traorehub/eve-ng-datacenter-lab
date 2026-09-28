# Architecture du laboratoire

Préfixe unique : `10.66.0.0/16`. Rien n'est recopié d'un plan de production.

## Agencement

L'ordre des maillons fixe le rayon d'une compromission :

1. Client (WAN ou LAN)
2. NGFW frontal (PAN-OS)
3. ADC / WAF (BIG-IP, HAProxy en parallèle)
4. NGFW dorsal (SFOS) si le flux redescend vers le LAN
5. Serveur (`web1` / `web2` / DVWA) ou annuaire

Deux DMZ, deux pare-feu : la DMZ publique est accrochée au frontal, la DMZ privée au dorsal. Coller les deux au même équipement élargirait le même incident aux serveurs de supervision et aux sites publiés.

## Zones PAN-OS

| Zone | Interface type | Rôle |
|------|----------------|------|
| WAN | lien ISP du lab | Trafic non sollicité, clients télétravail |
| INTERCO | `eth1/1`, VLAN 30 | Unique chemin vers le dorsal |
| DMZ_PUB | `eth1/4`, VLAN 10 | VIP, Self IP F5, pfSense |
| VPN-RA | `tunnel.3`, `10.66.80.1/24` | Pool Mode Config |

Une règle de sécurité par VIP. NAT sortant INTERCO vers WAN. SNAT du VPN vers INTERCO pour que le retour des serveurs LAN trouve le tunnel.

## Dorsal SFOS

| VLAN | Réseau | Passerelle | Contenu |
|------|--------|------------|---------|
| 20 | `10.66.20.0/24` | `.1` | Supervision (Zabbix) |
| 40 | `10.66.40.0/24` | `.1` | AD, postes GUI |
| 45 | `10.66.45.0/24` | `.1` | Voix (Alpine), distinct du data |
| 50 | `10.66.50.0/24` | `.1` | web1, web2, DVWA |
| 30 | `10.66.30.0/24` | `.2` côté Sophos | Interco vers Palo `.1` |

Le LAN n'a pas de route par défaut vers le WAN du lab : un poste VLAN 40 n'est pas un client du VIP. Le chemin métier vers DVWA / `.101` passe par `INTERCO-TO-DMZ_PUB`.

## Publication HTTP

| Objet | Adresse | Fonction |
|-------|---------|----------|
| HAProxy | `10.66.10.11:8080` | Round robin `10.66.50.80` / `.81` |
| `vs_web` | `10.66.10.100:80` | LTM + APM + ASM Transparent |
| `vs_dvwa` | `10.66.10.101:80` | ASM Blocking devant DVWA |
| Origine DVWA | `10.66.50.82` | Porte sans WAF, pour le couple de preuves |

Auto Map : les backends n'ont pas de route vers `10.66.10.0/24`. Sans SNAT vers la Self IP, le monitor LTM reste Offline.

## Accès distant

Deux objets distincts :

- **IPsec de site** Palo / Fortinet (VLAN 70, tunnel `10.66.72.0/30`) : L3 validé, SA IKE non établie sur cette VE (licence Invalid).
- **Télétravail** : passerelle GlobalProtect sur Palo, client Alpine **strongSwan** (IKEv1 Aggressive, X-Auth, Mode Config). Split include `10.66.0.0/16`.

## Commutation

Hiérarchie accès / distribution / ToR, IOSv-L2, EtherChannel `mode on` (LACP `mode active` irrégulier sur cet émulateur). VTP transparent.

Spine-Leaf / EVPN n'est pas dans ce canevas : l'underlay BGP n'est pas ce que IOSv-L2 tient ici.
