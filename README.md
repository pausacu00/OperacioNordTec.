<a id="inici"></a>

# Operació Nord Tec

**Autor:** Pau Simó Sánchez Cuervo
**Grup classe:** ASIX1A
**Empresa:** SecureSys Consulting
**Client:** NordTec

---

## Context del projecte

Som un equip d'administradors de sistemes de **SecureSys Consulting**, empresa especialitzada en desplegament i protecció d'infraestructures TIC. **NordTec**, una empresa tecnològica que acaba d'iniciar la seva activitat, ens ha contractat per construir tota la seva infraestructura informàtica des de zero.

| Camp | Dada |
|------|------|
| Empresa | NordTec |
| Plantilla | 25 treballadors |
| Sector | Empresa tecnològica · serveis SaaS |
| Ubicació | Barcelona |
| Departament TIC | No en té |
| Previsió | Creixement esperat durant els propers mesos |

Aquest document s'actualitza durant el desenvolupament del projecte, no al final: cada sessió de treball acaba amb una actualització del repositori i commits descriptius.

---

## Índex

1. [Estat del projecte](#1-estat-del-projecte)
2. [Arquitectura de xarxa](#2-arquitectura-de-xarxa)
3. [Configuracions](#3-configuracions)
4. [Incidències i solucions](#4-incidències-i-solucions)
5. [Decisions tècniques](#5-decisions-tècniques)
6. [Reflexions tècniques](#6-reflexions-tècniques)

---

## 1. Estat del projecte

### Peticions del client

#### Petició 1: Necessitem connexió a Internet

- **Sol·licitant:** Marta Illa (NordTec)
- **Resum:** els primers equips ja han arribat a l'oficina i cal que estiguin connectats en xarxa i puguin sortir a Internet. El client també demana que la configuració quedi ben ordenada i documentada, perquè preveu ampliar la xarxa aviat.
- **Estat:** en curs

### Fet

- [x] _Tasca completada 1_
- [x] _Tasca completada 2_
- [x] _Tasca completada 3_

### Pendent

- [ ] _Tasca pendent 1_
- [ ] _Tasca pendent 2_
- [ ] _Tasca pendent 3_

[⬆ Tornar a l'índex](#índex)

---

## 2. Arquitectura de xarxa

![Esquema de xarxa](img/esquema-xarxa.png)

> Substitueix la ruta per la teva imatge (per exemple, una captura de Packet Tracer o un diagrama).

### Informació rellevant

La xarxa està formada per quatre segments connectats a través del firewall, que és l'únic element amb accés a totes les xarxes:

```text
                              Internet
                                 |
                       enp1s0 (192.168.122.241, DHCP)
                                 |
                        +--------+--------+
  Kali 10.0.0.0/24      |                 |      DMZ 192.168.10.0/24
  ---- enp3s0 (.1) -----|    FIREWALL     |----- enp4s0 (.1) ---- Servidor web (.2)
                        |                 |
                        +--------+--------+
                                 |
                          enp5s0 (192.168.20.1)
                                 |
                        LAN 192.168.20.0/24
```

### Segments de xarxa

| Zona | Xarxa | Interfície del firewall | IP del firewall | Equips | Funció |
|------|-------|-------------------------|-----------------|--------|--------|
| Firewall (Internet) | 192.168.122.0/24 | enp1s0 | 192.168.122.241 (DHCP) | Firewall | Sortida a Internet |
| DMZ | 192.168.10.0/24 | enp4s0 | 192.168.10.1 | Servidor web | Serveis exposats |
| LAN | 192.168.20.0/24 | enp5s0 | 192.168.20.1 | Clients | Xarxa interna d'usuaris |
| Kali | 10.0.0.0/24 | enp3s0 | 10.0.0.1 | Kali Linux | Proves i auditoria |

El firewall és l'únic equip connectat a les quatre xarxes. La seva IP a cada segment és el gateway dels equips d'aquella zona.

### Equips i adreces IP

| Dispositiu | Zona | Adreça IP | Màscara | Gateway | Observacions |
|------------|------|-----------|---------|---------|--------------|
| Firewall | Totes | 192.168.122.241 (enp1s0, DHCP)<br>10.0.0.1 (enp3s0)<br>192.168.10.1 (enp4s0)<br>192.168.20.1 (enp5s0) | /24 | Per DHCP a enp1s0; cap a la resta | Ubuntu 24.04 LTS |
| Servidor web | DMZ | 192.168.10.2 | /24 | 192.168.10.1 | Web (port 80) i OWASP Juice Shop (port 3000) |
| Client LAN | LAN | 192.168.20.x | /24 | 192.168.20.1 | _..._ |
| Kali Linux | Kali | 10.0.0.x | /24 | 10.0.0.1 | Equip d'auditoria |

### Política de tràfic entre zones

| Origen → Destí | Permès | Observacions |
|----------------|--------|--------------|
| Firewall → Totes les xarxes | Sí | Política OUTPUT en ACCEPT |
| LAN → Internet | Sí | NAT (MASQUERADE) per enp1s0 |
| DMZ → Internet | Sí | Obert; pendent de restringir a DNS i HTTP/HTTPS per a actualitzacions |
| LAN → DMZ | Sí | |
| Kali → DMZ | Sí | |
| Kali → LAN | No | La LAN no accepta connexions noves des d'altres zones |
| DMZ → LAN | No | Bloquejat explícitament |
| Internet → DMZ | Només port 80 | DNAT cap a 192.168.10.2 |
| Kali → DMZ (per la IP del firewall, 10.0.0.1) | Ports 80 i 3000 | DNAT cap a 192.168.10.2 (web i OWASP Juice Shop) |
| Kali → Firewall | Ping | ICMP echo-request, ara mateix acceptat des de qualsevol interfície |
| LAN → Firewall | SSH (port 22) | Administració remota |

### Arquitectura de l'aplicació

- **Servidor web** a la DMZ (192.168.10.2), publicat pel port 80.
- **OWASP Juice Shop** al mateix servidor, port 3000, accessible des de la xarxa Kali.

_Afegeix aquí els components de l'aplicació SaaS de NordTec a mesura que es desplegui._

[⬆ Tornar a l'índex](#índex)

---

## 3. Configuracions

Explicació dels fitxers de configuració utilitzats al projecte.

### `/etc/netplan/60-nordtec.yaml`

- **Per a què serveix:** assigna una IP estàtica a les interfícies internes del firewall (Kali, DMZ i LAN). La interfície d'Internet (enp1s0) rep l'adreça per DHCP des del fitxer netplan que ja existia.
- **Paràmetres clau:**
  - `addresses`: IP del firewall a cada xarxa; és el gateway dels equips d'aquella zona.
  - Cap interfície interna porta `gateway`: només enp1s0 té sortida a Internet.

```yaml
network:
  version: 2
  ethernets:
    enp3s0:
      addresses: [10.0.0.1/24]
    enp4s0:
      addresses: [192.168.10.1/24]
    enp5s0:
      addresses: [192.168.20.1/24]
```

### `firewall.sh` (regles iptables)

- **Per a què serveix:** defineix la política de seguretat del firewall: NAT, aïllament entre zones, publicació de serveis i registre de tràfic descartat.
- **Requisit previ:** reenviament de paquets activat (`net.ipv4.ip_forward=1`).
- **Blocs principals:**

| Bloc | Què fa |
|------|--------|
| Neteja i polítiques per defecte | Esborra regles anteriors; `INPUT` i `FORWARD` en DROP, `OUTPUT` en ACCEPT |
| NAT de sortida | MASQUERADE de 192.168.10.0/24 i 192.168.20.0/24 per enp1s0 |
| Tràfic de retorn | Accepta `ESTABLISHED,RELATED` a `INPUT` i `FORWARD`; descarta paquets `INVALID` |
| Accés al firewall | Loopback, ping i SSH des de la LAN (enp5s0) |
| Sortida a Internet | Permet que la LAN i la DMZ surtin per enp1s0 |
| Publicació de serveis (DNAT) | Web (port 80) des d'Internet i des de Kali, i OWASP Juice Shop (port 3000) des de Kali, cap a 192.168.10.2 |
| Aïllament de la LAN | Descarta connexions noves cap a 192.168.20.0/24; permet LAN → DMZ i Kali → DMZ; bloqueja DMZ → LAN |
| Registre | `LOG` amb el prefix `IPTABLES-DROP: ` al final de `FORWARD` |

### Comandes utilitzades

```bash
# Activar el reenviament de paquets (necessari perquè el firewall enruti)
echo "net.ipv4.ip_forward=1" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p

# Aplicar la configuració de xarxa i comprovar les IP
sudo chmod 600 /etc/netplan/60-nordtec.yaml
sudo netplan apply
ip -br a

# Executar les regles del firewall i fer-les persistents
sudo bash firewall.sh
sudo apt install iptables-persistent
sudo netfilter-persistent save
```

### Captures de pantalla

![Descripció de la captura](img/captura-1.png)

_Explica breument què es veu a la captura i a quina configuració correspon._

[⬆ Tornar a l'índex](#índex)

---

## 4. Incidències i solucions

> Repeteix el bloc següent per a cada incidència.

### Incidència 1: _títol breu_

- **Missatge d'error exacte:**
  ```text
  Enganxa aquí l'error tal com apareix
  ```
- **Quan:** _fase del projecte i context (què estàvem fent)_
- **Causa:** _per què passava_
- **Solució:** _què vam fer per resoldre-ho_
- **Detectada per:** @usuari

### Incidència 2: _títol breu_

- **Missatge d'error exacte:**
  ```text
  Enganxa aquí l'error tal com apareix
  ```
- **Quan:** _fase i context_
- **Causa:** _per què passava_
- **Solució:** _què vam fer_
- **Detectada per:** @usuari

[⬆ Tornar a l'índex](#índex)

---

## 5. Decisions tècniques

| Decisió | Què hem triat | Per què | Alternatives considerades |
|---------|---------------|---------|---------------------------|
| _Tema 1_ | _Opció triada_ | _Motiu_ | _Altres opcions_ |
| _Tema 2_ | _Opció triada_ | _Motiu_ | _Altres opcions_ |
| _Tema 3_ | _Opció triada_ | _Motiu_ | _Altres opcions_ |

[⬆ Tornar a l'índex](#índex)

---

## 6. Reflexions tècniques

- **Què ha funcionat bé:** _..._
- **Què faria diferent:** _..._
- **Què he après:** _..._
- **Possibles millores futures:** _..._

[⬆ Tornar a l'índex](#índex)