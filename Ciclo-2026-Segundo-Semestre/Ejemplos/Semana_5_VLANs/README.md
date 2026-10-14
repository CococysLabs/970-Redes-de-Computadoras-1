# Configuración de VLANs + VTP + Trunks/Access

Objetivo: Crear 4 VLANs (Ventas=10, Admin=20, Soporte=30, Laboratorio=40), configurar **1 switch Server VTP**, **3 switches Client VTP** y **1 switch Transparent VTP (SW_Laboratorio)** en el dominio **semana4**, **VTP v2**, contraseña **123**, y dejar ejemplos de **puertos trunk** (Fa0/2) y **puertos access**.

---

## VLANs a crear

| VLAN | Nombre         | Dónde se crea                                   |
|------|----------------|--------------------------------------------------|
| 10   | VENTAS         | Switch Server (se propaga por VTP a los Clients) |
| 20   | ADMIN          | Switch Server (se propaga por VTP a los Clients) |
| 30   | SOPORTE        | Switch Server (se propaga por VTP a los Clients) |
| 40   | LABORATORIO2   | Switch Server (se propaga por VTP a los Clients)      |
| 40   | LABORATORIO    | SW_Laboratorio (local, modo transparente) |

>**Nota:** la VLAN 40 existe en ambos switches pero con nombres distintos (`laboratorio2` en el Server y `laboratorio` en SW_Laboratorio). Esto es normal y esperado porque **SW_Laboratorio está en modo VTP transparente** y no sincroniza su base de datos de VLANs con el dominio: cada switch mantiene su propio nombre para la VLAN 40.

> **Nota de Capa 2:** Aunque la VLAN 40 tiene nombres distintos en cada switch, ambas pertenecen al mismo dominio de difusión de Capa 2 (el protocolo 802.1Q etiqueta el tráfico por el ID 40, no por el nombre). Se configuró de esta forma a modo de ejemplo para demostrar que un switch en modo VTP Transparente gestiona su base de datos de VLANs de manera totalmente local e independiente del dominio.

---

## Switch SERVER (VTP Server)

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
 name VENTAS
exit

vlan 20
 name ADMIN
exit

vlan 30
 name SOPORTE
exit

vlan 40
 name laboratorio2
exit
```

### Ejemplo puerto TRUNK
```bash
# Esta configuración es para los enlaces troncales entre switches.

enable
configure terminal

interface fa0/2
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30,40
 no shutdown
exit
```

---

## Switches CLIENT (VTP Client)

### Configurar VTP (Client)
```bash
enable
configure terminal

vtp version 2
vtp mode client
vtp domain semana4
vtp password 123
```

> Las VLANs 10, 20, 30 y 40 (`laboratorio2`) llegan automáticamente por VTP desde el Server. No se crean manualmente aquí.

### Ejemplo puerto ACCESS
```bash
# Esta configuración es para los enlaces que van a los dispositivos finales

enable
configure terminal

interface fa0/3
 description PC Ventas
 switchport mode access
 switchport access vlan 10
 no shutdown
exit
```

---

## Switch SW_LABORATORIO (VTP Transparent)

>**Importante:** en un switch transparente hay que configurar primero el **modo**, y después el dominio/versión/password. En modo transparente el switch no valida ni sincroniza su base de datos de VLANs contra el dominio VTP, solo reenvía los anuncios sin procesarlos. Por eso conviene dejarlo en modo transparente antes de asociarlo al dominio, para evitar que intente sincronizar su tabla de VLANs si venía en otro modo por defecto.

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
 name laboratorio
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

---

## Comandos de Verificación

```bash
show vtp status
show vlan brief
show interfaces trunk
```

### Resultado esperado — Switch Server / Clients
```
VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Fa0/5, Fa0/6, Fa0/7, Fa0/8
                                                 Fa0/9, Fa0/10, Fa0/11, Fa0/12
                                                 Fa0/13, Fa0/14, Fa0/15, Fa0/16
                                                 Fa0/17, Fa0/18, Fa0/19, Fa0/20
                                                 Fa0/21, Fa0/22, Fa0/23, Fa0/24
                                                 Gig0/1, Gig0/2
10   ventas                           active
20   admin                            active
30   soporte                          active
40   laboratorio2                     active
1002 fddi-default                     active
1003 token-ring-default               active
```

### Resultado esperado — SW_Laboratorio (Transparent)
```
VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Fa0/5, Fa0/6, Fa0/7, Fa0/8
                                                 Fa0/9, Fa0/10, Fa0/11, Fa0/12
                                                 Fa0/13, Fa0/14, Fa0/15, Fa0/16
                                                 Fa0/17, Fa0/18, Fa0/19, Fa0/20
                                                 Fa0/21, Fa0/22, Fa0/23, Fa0/24
                                                 Gig0/1, Gig0/2
40   laboratorio                      active    Fa0/3, Fa0/4
1002 fddi-default                     active
1003 token-ring-default               active
1004 fddinet-default                  active
```