# Configuración de Port-Channel (EtherChannel)

## Descripción

Port-Channel (EtherChannel) permite agrupar varios enlaces físicos en **un solo enlace lógico**, incrementando ancho de banda y redundancia. Para STP, el Port-Channel se ve como **un único enlace**, reduciendo bloqueos innecesarios.

Esta configuración se hace **primero**, porque agrupar los enlaces físicos en Po1 es el paso base antes de analizar cómo STP trata la topología: una vez que el EtherChannel está levantado, STP ya no ve cables individuales entre los switches, sino un solo enlace lógico por cada par de switches, lo cual simplifica el árbol de spanning-tree resultante.

**VLAN de ejemplo**

* VLAN 10 → lab

**Escenario**

* Topología: **SW_Server ↔ Switch1 ↔ Switch2** (triángulo, con SW_Server también conectado directo a Switch2)
* SW_Server es el **VTP Server** del dominio `semana4` y además está **forzado como Root Bridge** para VLAN 10
* Switch1 y Switch2 estan configurados como **VTP Client**
* Enlace a agrupar en este ejemplo: **SW_Server ↔ Switch1**
* Puertos físicos a agrupar: **Fa0/2 y Fa0/4**
* Port-Channel: **Po1**

---

## Reglas para que levante (muy importante)

* Los puertos deben tener **misma velocidad/duplex** y ser del **mismo tipo** (no mezclar Fa con Gi).
* La configuración debe ser **igual en ambos lados**:
  * trunk/access
  * allowed VLANs
* En Packet Tracer, si algo se "suspende", resetea interfaces y vuelve a crear el grupo.

---

# Opción A — LACP (IEEE, recomendado)

## SW_Server (LACP Active)

```bash
enable
conf t

interface range fa0/2 , fa0/4
 channel-protocol lacp
 channel-group 1 mode active
 no shutdown
exit

interface port-channel 1
 switchport mode trunk
 switchport trunk allowed vlan 10
 no shutdown
end
wr
```

## Switch1 (LACP Passive)

```bash
enable
conf t

interface range fa0/2 , fa0/4
 channel-protocol lacp
 channel-group 1 mode passive
 no shutdown
exit

interface port-channel 1
 switchport mode trunk
 switchport trunk allowed vlan 10
 no shutdown
end
wr
```

**Combinación válida:** `active + passive` (o `active + active`)  

**No forma:** `passive + passive`

---

# Opción B — PAgP (Cisco)

## SW_Server (PAgP Desirable)

```bash
enable
conf t

interface range fa0/2 , fa0/4
 channel-protocol pagp
 channel-group 1 mode desirable
 no shutdown
exit

interface port-channel 1
 switchport mode trunk
 switchport trunk allowed vlan 10
 no shutdown
end
wr
```

## Switch1 (PAgP Auto)

```bash
enable
conf t

interface range fa0/2 , fa0/4
 channel-protocol pagp
 channel-group 1 mode auto
 no shutdown
exit

interface port-channel 1
 switchport mode trunk
 switchport trunk allowed vlan 10
 no shutdown
end
wr
```

**Combinación válida:** `desirable + auto` (o `desirable + desirable`)  

**No forma:** `auto + auto`

---

# Verificación (para ambos)

## 1. Ver estado del EtherChannel

```bash
show etherchannel summary
```

## 2. Ver que el Port-Channel está en trunk

```bash
show interfaces port-channel 1 trunk
```

## 3. Ver vecinos LACP (solo si usaste LACP)

```bash
show lacp neighbor
```

## 4. Ver STP sobre el Port-Channel

```bash
show spanning-tree
```

STP debería ver **Po1** como un solo enlace lógico (y no cada cable, Fa0/2 y Fa0/4, por separado).

---

# Troubleshooting rápido (si no levanta)

## Reset limpio (recomendado)

En ambos switches:

```bash
conf t
default interface range fa0/2 , fa0/4
no interface port-channel 1
end
wr
```

Luego repite la configuración.

---

# Extra: Forzar el Root Bridge para VLAN 10 (SW_Server)

Una vez que el Port-Channel está levantado, STP sigue corriendo sobre la topología (SW_Server, Switch1 y Switch2 forman un triángulo, así que hay un loop físico que hay que controlar). Se puede **forzar manualmente** cuál switch es el Root Bridge para una VLAN específica, en vez de dejar que gane por el ID de switch más bajo por defecto.

En este ejemplo, **SW_Server** se fuerza como Root Bridge de la **VLAN 10**:

## Opción 1 — `root primary` (recomendada, más simple)

```bash
enable
conf t

spanning-tree vlan 10 root primary
end
wr
```

Este comando ajusta automáticamente la prioridad de SW_Server a un valor lo suficientemente bajo (normalmente 24576, o menos si hace falta) para garantizar que gane la elección frente a Switch1 y Switch2.

## Opción 2 — Prioridad manual

```bash
enable
conf t

spanning-tree vlan 10 priority 4096
end
wr
```

> La prioridad debe ser múltiplo de 4096. Mientras más bajo el número, mayor prioridad para ser Root Bridge.

## Verificación

```bash
show spanning-tree vlan 10
```

En la salida, SW_Server debe aparecer como:

```
Root ID    Priority    24576 (o el valor configurado)
           This bridge is the root
```

Y en Switch1 / Switch2, al correr el mismo comando, el **Root ID** debe mostrar la MAC/prioridad de SW_Server, confirmando que ambos lo reconocen como raíz del árbol para VLAN 10.