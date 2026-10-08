
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
 
- [x] Configuració de les interfícies de xarxa del firewall (netplan): enp1s0 i enp2s0 per DHCP; Kali, DMZ i LAN amb IP estàtica
- [x] Reenviament de paquets IP activat (`net.ipv4.ip_forward`)
- [x] Script `firewall.sh` creat i executat: NAT de sortida a Internet, aïllament de la LAN, publicació del servidor web i registre del tràfic descartat
### Pendent
 
- [ ] Fer les regles del firewall persistents (`netfilter-persistent save`)
- [ ] Verificar la connectivitat entre zones i la sortida a Internet des de la LAN
- [ ] Afegir captures de pantalla i completar l'esquema de xarxa
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
  ---- enp3s0 (.1) -----|    FIREWALL     |----- enp4s0 (.1) ---- Servidor web (.10)
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
| Gestió | Per DHCP | enp2s0 | Per DHCP | Firewall | Administració remota (SSH) |
| DMZ | 192.168.10.0/24 | enp4s0 | 192.168.10.1 | Servidor web | Serveis exposats |
| LAN | 192.168.20.0/24 | enp5s0 | 192.168.20.1 | Clients | Xarxa interna d'usuaris |
| Kali | 10.0.0.0/24 | enp3s0 | 10.0.0.1 | Kali Linux | Proves i auditoria |
 
El firewall és l'únic equip connectat a les quatre xarxes. La seva IP a cada segment és el gateway dels equips d'aquella zona.
 
### Equips i adreces IP
 
| Dispositiu | Zona | Adreça IP | Màscara | Gateway | Observacions |
|------------|------|-----------|---------|---------|--------------|
| Firewall | Totes | 192.168.122.241 (enp1s0, DHCP)<br>DHCP (enp2s0)<br>10.0.0.1 (enp3s0)<br>192.168.10.1 (enp4s0)<br>192.168.20.1 (enp5s0) | /24 | Per DHCP a enp1s0 i enp2s0; cap a la resta | Ubuntu 24.04 LTS |
| Servidor web | DMZ | 192.168.10.10 | /24 | 192.168.10.1 | Web (port 80) i OWASP Juice Shop (port 3000) |
| Client LAN | LAN | 192.168.20.10 | /24 | 192.168.20.1 | _..._ |
| Kali Linux | Kali | 10.0.0.x | /24 | 10.0.0.1 | Equip d'auditoria |
 
### Política de tràfic entre zones
 
| Origen → Destí | Permès | Observacions |
|----------------|--------|--------------|
| Firewall → Totes les xarxes | Sí | Política OUTPUT en ACCEPT |
| LAN → Internet | Sí | NAT (MASQUERADE) per enp1s0 |
| DMZ → Internet | Només DNS, HTTP i HTTPS | Per a actualitzacions; la resta de ports queda bloquejat |
| LAN → DMZ | Sí | |
| Kali → DMZ | Sí | |
| Kali → LAN | No | La LAN no accepta connexions noves des d'altres zones |
| DMZ → LAN | No | Bloquejat explícitament |
| Internet → DMZ | Només port 80 | DNAT cap a 192.168.10.10 |
| Kali → DMZ (per la IP del firewall, 10.0.0.1) | Ports 80 i 3000 | DNAT cap a 192.168.10.10 (web i OWASP Juice Shop) |
| Kali → Firewall | Ping | ICMP echo-request, només des de la interfície de Kali |
| LAN → Firewall | SSH (port 22) | Administració remota |
| Gestió (enp2s0) → Firewall | SSH (port 22) | Administració remota des de la interfície de gestió |
 
### Arquitectura de l'aplicació
 
- **Servidor web** a la DMZ (192.168.10.10), publicat pel port 80.
- **OWASP Juice Shop** al mateix servidor, port 3000, accessible des de la xarxa Kali.
_Afegeix aquí els components de l'aplicació SaaS de NordTec a mesura que es desplegui._
 
[⬆ Tornar a l'índex](#índex)
 
---
 
## 3. Configuracions
 
Explicació dels fitxers de configuració utilitzats al projecte.
 
### `/etc/sysctl.conf`
 
- **Per a què serveix:** activa de manera permanent el reenviament de paquets IPv4. Sense això el firewall no enruta tràfic entre les xarxes.
- **Paràmetre clau:**
  - `net.ipv4.ip_forward`: amb valor `1` permet reenviar paquets entre interfícies.
```conf
net.ipv4.ip_forward=1
```
 
L'script `firewall.sh` també l'activa en calent (`sysctl -w`), però només fins al proper reinici.
 
### `/etc/netplan/60-nordtec.yaml`
 
- **Per a què serveix:** configura les interfícies del firewall. enp1s0 i enp2s0 reben l'adreça per DHCP; les interfícies internes (Kali, DMZ i LAN) tenen IP estàtica.
- **Paràmetres clau:**
  - `dhcp4: true`: l'adreça s'obté per DHCP.
  - `route-metric`: prioritat de la ruta per defecte. enp1s0 (mètrica 100) és la preferida per sortir a Internet; enp2s0 (mètrica 200) queda com a reserva.
  - `addresses`: IP del firewall a cada xarxa interna; és el gateway dels equips d'aquella zona.
  - Cap interfície interna porta `gateway`.
```yaml
network:
  version: 2
  ethernets:
    enp1s0:
      dhcp4: true
      dhcp4-overrides:
        route-metric: 100
    enp2s0:
      dhcp4: true
      dhcp4-overrides:
        route-metric: 200
    enp3s0:
      addresses: [10.0.0.1/24]
    enp4s0:
      addresses: [192.168.10.1/24]
    enp5s0:
      addresses: [192.168.20.1/24]
```
 
### `firewall.sh` (regles iptables)
 
- **Per a què serveix:** defineix la política de seguretat del firewall: NAT, aïllament entre zones, publicació de serveis i registre del tràfic descartat.
- **Execució:** `sudo bash firewall.sh`
- **Blocs principals:**
| Bloc | Què fa |
|------|--------|
| Variables | Defineix les interfícies (enp1s0 Internet, enp2s0 gestió, enp3s0 Kali, enp4s0 DMZ, enp5s0 LAN), les xarxes i la IP del servidor web, per canviar-ho tot en un sol lloc |
| Reenviament i polítiques temporals | Activa `ip_forward` en calent i posa les polítiques en ACCEPT mentre s'apliquen les regles, per no perdre la sessió |
| Neteja | `iptables -F`, `iptables -t nat -F` i `iptables -X` |
| INPUT | Loopback; tràfic de retorn (`ESTABLISHED,RELATED`); descarta `INVALID`; respostes DHCP a enp1s0 i enp2s0; ping només des de Kali; SSH des de la LAN i des de la interfície de gestió |
| NAT | MASQUERADE de la LAN i la DMZ per enp1s0; DNAT del port 80 des d'Internet i des de Kali (via 10.0.0.1), i del port 3000 (OWASP Juice Shop) des de Kali, cap a 192.168.10.10 |
| FORWARD | Descarta `INVALID` i accepta el tràfic de retorn; LAN → Internet; DMZ → Internet només DNS, HTTP i HTTPS; LAN → DMZ; Kali → DMZ; Internet → servidor web (port 80) |
| Aïllament de la LAN | Registra i descarta les connexions noves cap a 192.168.20.0/24 des de qualsevol zona |
| Registre | `LOG` amb el prefix `IPTABLES-DROP: `, limitat a 5 per minut |
| Polítiques per defecte | Al final: `INPUT` i `FORWARD` en DROP, `OUTPUT` en ACCEPT |
 
### Comandes utilitzades
 
```bash
# Reenviament de paquets permanent: descomentar net.ipv4.ip_forward=1
sudo nano /etc/sysctl.conf
sudo sysctl -p
cat /proc/sys/net/ipv4/ip_forward
 
# Configuració de xarxa (netplan) i comprovació d'IP i rutes
sudo nano /etc/netplan/60-nordtec.yaml
sudo chmod 600 /etc/netplan/60-nordtec.yaml
sudo netplan apply
ip -br a
ip route
 
# Aplicar les regles del firewall
sudo bash firewall.sh
 
# Comprovar les regles carregades
sudo iptables -L -n -v --line-numbers
sudo iptables -t nat -L -n -v
 
# Fer les regles persistents
sudo apt install iptables-persistent
sudo netfilter-persistent save
 
# Consultar el tràfic descartat
sudo journalctl -k | grep IPTABLES-DROP
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