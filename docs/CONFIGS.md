# Extraits de configuration

Aucun mot de passe, aucune PSK, aucune clé de licence. Les extraits suffisent à comprendre l'objet ; ils ne sont pas un dump d'appliance.

## HAProxy (round robin)

```haproxy
frontend fe_pub
    bind :8080
    default_backend be_web

backend be_web
    balance roundrobin
    server web1 10.66.50.80:80 check
    server web2 10.66.50.81:80 check
```

## LTM (idée Auto Map)

```text
ltm pool pool_web {
    members { 10.66.50.80:80 10.66.50.81:80 }
    monitor http
}
ltm virtual vs_web {
    destination 10.66.10.100:80
    pool pool_web
    source-address-translation { type automap }
    profiles { http tcp ap_web }
}
```

`vs_dvwa` écoute `10.66.10.101:80`, policy ASM en Blocking. Après un changement REST : `apply-policy`.

## strongSwan (client télétravail)

```text
conn palo
    keyexchange=ikev1
    aggressive=yes
    left=%defaultroute
    leftsourceip=%modeconfig
    leftauth=psk
    rightauth=psk
    rightauth2=xauth-generic
    right=<IP_WAN_PALO>
    ike=aes256-sha256-modp2048
    esp=aes256-sha256
    auto=add
```

La PSK n'est pas publiée. Le Mode Config installe `10.66.80.x`. Un `wget` vers un backend **sans** VIP tunnel est un raccourci d'émulateur, pas la preuve VPN.

## PAN-OS (principes)

- Zones deny-all par défaut, une règle explicite par VIP (`WAN-TO-F5`, `WAN-TO-DVWA`, `DMZPUB-TO-DVWA`, `INTERCO-TO-DMZ_PUB`).
- NAT : SNAT INTERCO vers WAN ; SNAT VPN-RA vers INTERCO.
- IKE de site (Palo / Fortinet) : IKEv2, AES-256, SHA-256, DH14. SA non établie sur la VE Forti en licence Invalid.
