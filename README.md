
# Operació Nord Tec
 
**Autor:** Pau Simó Sánchez Cuervo
**Grup classe:** ASIX1A
 
---
 
## Índex
 
- [Operació Nord Tec](#operació-nord-tec)
  - [Índex](#índex)
  - [1. Estat del projecte](#1-estat-del-projecte)
    - [Fet](#fet)
    - [Pendent](#pendent)
  - [2. Arquitectura de xarxa](#2-arquitectura-de-xarxa)
    - [Informació rellevant](#informació-rellevant)
  - [3. Configuracions](#3-configuracions)
    - [`ruta/del/fitxer-1.conf`](#rutadelfitxer-1conf)
    - [`ruta/del/fitxer-2.conf`](#rutadelfitxer-2conf)
  - [4. Incidències i solucions](#4-incidències-i-solucions)
    - [Incidència 1: _títol breu_](#incidència-1-títol-breu)
    - [Incidència 2: _títol breu_](#incidència-2-títol-breu)
  - [5. Decisions tècniques](#5-decisions-tècniques)
  - [6. Reflexions tècniques](#6-reflexions-tècniques)
---
 
## 1. Estat del projecte
 
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
 
| Dispositiu | Interfície | Adreça IP | Màscara | Gateway | Rol / Observacions |
|------------|-----------|-----------|---------|---------|--------------------|
| _Router_   | _Gi0/0_   | _x.x.x.x_ | _/24_   | —       | _Sortida a Internet_ |
| _Switch_   | _VLAN 1_  | _x.x.x.x_ | _/24_   | _x.x.x.x_ | _Gestió_ |
| _Servidor_ | _eth0_    | _x.x.x.x_ | _/24_   | _x.x.x.x_ | _Serveis_ |
| _Client_   | _eth0_    | _DHCP_    | _/24_   | _x.x.x.x_ | _Equip d'usuari_ |
 
- **Xarxa:** _x.x.x.0/24_
- **Rang DHCP:** _x.x.x.x – x.x.x.x_
- **DNS:** _x.x.x.x_
- **VLANs (si n'hi ha):** _ID – nom – xarxa_
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