# Configuración de VLANs + VTP + Trunks/Access

## Objetivo

Crear 4 VLANs (**Ingeniería=10, Odontología=20, Arquitectura=30, Rectoría=40**), configurar **1 switch Server VTP** (SW_Server), **4 switches Client VTP** (SW_Arquitectura, SW_Ingenieria, SW_Odontologia1 y SW_Odontologia2) y **1 switch Transparent VTP** (SW_Rectoria), todos en el dominio **semana4**, **VTP v2**, contraseña **123**, dejando ejemplos de **puertos trunk** entre switches y **puertos access** hacia los dispositivos finales. Además, se agrega un **Access Point** conectado a un switch para dar acceso inalámbrico (WiFi) a una laptop y un celular, dentro de la misma VLAN que los equipos cableados de esa área.

## Topología del escenario

- **SW_Server** (VTP Server, switch central): se conecta por enlaces troncales a SW_Arquitectura, SW_Rectoria y SW_Ingenieria.
- **SW_Arquitectura** (VTP Client): conecta dos equipos a la VLAN 30 (Arquitectura).
- **SW_Ingenieria** (VTP Client): conecta dos equipos a la VLAN 10 (Ingeniería) y un **Access Point** que da acceso inalámbrico a la VLAN 10.
- **SW_Rectoria** (VTP Transparent): queda entre SW_Server y SW_Odontologia1 (reenvía los anuncios VTP sin procesarlos) y además conecta localmente dos equipos a la VLAN 40 (Rectoría).
- **SW_Odontologia1** (VTP Client): conecta una equipo a la VLAN 20 (Odontología) y enlaza hacia SW_Odontologia2.
- **SW_Odontologia2** (VTP Client): conecta una equipo a la VLAN 20 (Odontología).

Este dominio VTP tiene un Server que centraliza la base de datos de VLANs, varios Clients que la reciben automáticamente, y un switch Transparent que mantiene su propia base de datos local sin sincronizarse con el dominio, aunque sigue reenviando los anuncios VTP a los switches que tiene detrás.

## VLANs a crear

| VLAN | Nombre        | Dónde se crea                                      |
|------|---------------|-----------------------------------------------------|
| 10   | INGENIERIA    | Switch Server (se propaga por VTP a los Clients)    |
| 20   | ODONTOLOGIA   | Switch Server (se propaga por VTP a los Clients)    |
| 30   | ARQUITECTURA  | Switch Server (se propaga por VTP a los Clients)    |
| 40   | RECTORIA      | SW_Rectoria (local, modo transparente)              |

---

## Switch SERVER (VTP Server) — SW_Server

### Configurar VTP (Server)

```bash
enable
configure terminal

vtp version 2
vtp mode server
vtp domain semana4
vtp password 123
```

### Crear VLANs

```bash
enable
configure terminal

vlan 10
 name ingenieria
exit

vlan 20
 name odontologia
exit

vlan 30
 name arquitectura
exit
```

### Puertos TRUNK hacia los switches vecinos

```bash
# SW_Server tiene tres enlaces troncales: hacia SW_Arquitectura, SW_Rectoria y SW_Ingenieria

enable
configure terminal

interface fa0/1
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30
 no shutdown
exit

interface fa0/2
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30
 no shutdown
exit

interface fa0/3
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30
 no shutdown
exit
```

---

## Switches CLIENT (VTP Client) — SW_Arquitectura, SW_Ingenieria, SW_Odontologia1, SW_Odontologia2

### Configurar VTP (Client)

```bash
enable
configure terminal

vtp version 2
vtp mode client
vtp domain semana4
vtp password 123
```

> Las VLANs 10, 20 y 30 llegan automáticamente por VTP desde el Server. No se crean manualmente en ningún switch Client.

### Puertos TRUNK entre switches (cadena de Odontología)

```bash
# SW_Odontologia1 se conecta hacia SW_Rectoria (uplink) y hacia SW_Odontologia2 (downlink)

enable
configure terminal

interface fa0/1
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30
 no shutdown
exit

interface fa0/3
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30
 no shutdown
exit
```

```bash
# SW_Odontologia2 se conecta hacia SW_Odontologia1

enable
configure terminal

interface fa0/1
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30
 no shutdown
exit
```

### Ejemplo puertos ACCESS (equipos cableados)

```bash
# SW_Arquitectura - PCs de Arquitectura (VLAN 30)

enable
configure terminal

interface range fa0/2 - 3
 switchport mode access
 switchport access vlan 30
 no shutdown
exit
```

```bash
# SW_Odontologia1 - PC de Odontología (VLAN 20)

enable
configure terminal

interface fa0/2
 description PC Odontologia
 switchport mode access
 switchport access vlan 20
 no shutdown
exit
```

### Puerto ACCESS especial: Access Point en Ingeniería

El Access Point no es un switch, es un dispositivo final más desde el punto de vista del switch: se conecta por cable a un puerto de SW_Ingenieria y ese puerto se deja como **access** en la VLAN 10. Así, todo lo que entra por WiFi al AP (la laptop y el celular) queda dentro de la VLAN 10 y puede comunicarse con el resto de los equipos de Ingeniería.

```bash
# SW_Ingenieria - puerto hacia el Access Point (VLAN 10)

enable
configure terminal

interface fa0/2
 description AP_Ingenieria (WiFi - laptop y celular)
 switchport mode access
 switchport access vlan 10
 no shutdown
exit
```

---

## Switch SW_RECTORIA (VTP Transparent)

> **Importante:** en un switch transparente hay que configurar primero el **modo**, y después el dominio/versión/password. En modo transparente el switch no valida ni sincroniza su base de datos de VLANs contra el dominio VTP, solo reenvía los anuncios sin procesarlos. Por eso conviene dejarlo en modo transparente antes de asociarlo al dominio, para evitar que intente sincronizar su tabla de VLANs si venía en otro modo por defecto o hacerlo antes de truncar los puertos.

### 1) Configurar el modo (primero)

```bash
enable
configure terminal

vtp mode transparent
```

### 2) Configurar versión, dominio y password (después)

```bash
vtp version 2
vtp domain semana4
vtp password 123
```

### 3) Crear la VLAN localmente

En modo transparente las VLANs **no llegan por VTP**, hay que crearlas a mano en cada switch transparente:

```bash
configure terminal

vlan 40
 name RECTORIA
exit
```

### 4) Puertos ACCESS para la VLAN 40 (Fa0/3 y Fa0/4)

```bash
configure terminal

interface range fa0/3 - 4
 switchport mode access
 switchport access vlan 40
 no shutdown
exit
```

### Puertos TRUNK (hacia SW_Server y hacia SW_Odontologia1)

Aunque SW_Rectoria no usa las VLANs 10, 20 ni 30, sus puertos troncales deben dejarlas pasar, porque es el switch que queda **en medio** del camino entre SW_Server y SW_Odontologia1/SW_Odontologia2. Si no permitiera esas VLANs en el trunk, Odontología se quedaría sin recibir los anuncios VTP ni el tráfico de su propia VLAN 20.

```bash
enable
configure terminal

interface fa0/1
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30
 no shutdown
exit

interface fa0/2
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30
 no shutdown
exit
```

---

## Comandos de Verificación

```bash
show vtp status
show vlan brief
show interfaces trunk
```