# Plan d'adressage (laboratoire)

| VLAN / objet | Préfixe | Passerelle / VIP |
|--------------|---------|------------------|
| DMZ publique | `10.66.10.0/24` | Palo `.1`, F5 `.10`, HAProxy `.11`, `vs_web` `.100`, `vs_dvwa` `.101` |
| DMZ privée | `10.66.20.0/24` | Sophos `.1`, Zabbix `.10` |
| Interco | `10.66.30.0/24` | Palo `.1`, Sophos `.2` |
| LAN | `10.66.40.0/24` | Sophos `.1`, DC `.10` |
| Voix | `10.66.45.0/24` | Sophos `.1` |
| Serveurs HTTP | `10.66.50.0/24` | Sophos `.1`, web1 `.80`, web2 `.81`, DVWA `.82` |
| Télétravail | `10.66.80.0/24` | Palo tunnel `.1`, pool `.10` à `.20` |
