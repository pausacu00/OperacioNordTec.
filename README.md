
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
                      +----------------+
        Xarxa Kali ---|                |--- DMZ
                      |    FIREWALL    |
        LAN ----------|                |--- (Xarxa del firewall / WAN)
                      +----------------+
```
 
### Segments de xarxa
 
| Zona | Xarxa | Interfície del firewall | IP del firewall | Equips | Funció |
|------|-------|-------------------------|-----------------|--------|--------|
| Firewall | _x.x.x.0/24_ | _..._ | _x.x.x.x_ | Firewall | Accés a totes les xarxes |
| DMZ | _x.x.x.0/24_ | _..._ | _x.x.x.x_ | _Servidor(s)_ | Serveis exposats |
| LAN | _x.x.x.0/24_ | _..._ | _x.x.x.x_ | _Clients_ | Xarxa interna d'usuaris |
| Kali | _x.x.x.0/24_ | _..._ | _x.x.x.x_ | Kali Linux | Proves i auditoria |
 
### Equips i adreces IP
 
| Dispositiu | Zona | Adreça IP | Màscara | Gateway | Observacions |
|------------|------|-----------|---------|---------|--------------|
| _Firewall_ | Totes | _x.x.x.x_ | _/24_ | — | _..._ |
| _Servidor DMZ_ | DMZ | _x.x.x.x_ | _/24_ | _x.x.x.x_ | _..._ |
| _Client LAN_ | LAN | _x.x.x.x_ | _/24_ | _x.x.x.x_ | _..._ |
| _Kali Linux_ | Kali | _x.x.x.x_ | _/24_ | _x.x.x.x_ | _..._ |
 
### Política de tràfic entre zones
 
| Origen → Destí | Permès | Observacions |
|----------------|--------|--------------|
| Firewall → Totes les xarxes | Sí | Accés complet |
| LAN → DMZ | _..._ | _..._ |
| Kali → DMZ / LAN | _..._ | _..._ |
| DMZ → LAN | _..._ | _..._ |
 
### Arquitectura de l'aplicació
 
_Descriu aquí els components de l'aplicació SaaS i en quina zona de la xarxa es desplega cada un._
 
[⬆ Tornar a l'índex](#índex)
 
---
 
## 3. Configuracions
 
Explicació dels fitxers de configuració utilitzats al projecte.
 
### `ruta/del/fitxer-1.conf`
 
- **Per a què serveix:** _descripció breu_
- **Paràmetres clau:**
  - `parametre1`: _què fa_
  - `parametre2`: _què fa_
```conf
# Exemple del contingut rellevant
parametre1 = valor
parametre2 = valor
```
 
### `ruta/del/fitxer-2.conf`
 
- **Per a què serveix:** _descripció breu_
- **Paràmetres clau:**
  - `parametre1`: _què fa_
```conf
# Exemple del contingut rellevant
```
 
### Comandes utilitzades
 
```bash
# Comanda 1: què fa i per què s'ha executat
comanda --opcio
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