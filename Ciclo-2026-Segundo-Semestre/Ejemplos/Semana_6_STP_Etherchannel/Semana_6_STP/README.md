# Configuración de STP (Spanning Tree Protocol)

## Descripción

STP (Spanning Tree Protocol) evita loops en topologías con enlaces redundantes, bloqueando lógicamente los puertos que generarían un bucle y dejando un solo camino activo hacia el Root Bridge por VLAN.

En este ejemplo se trabaja sobre **STP**, sin EtherChannel: cada enlace físico entre los switches se deja tal cual, y es STP el que decide qué puerto queda en forwarding y cuál queda bloqueado (blocking).

**Escenario**

* Dominio VTP: `semana4` | contraseña `123`
* Topología: **SW_Server - Switch1 - Switch2** (triángulo con loop físico)
* **SW_Server** es el VTP Server y se va a forzar como **Root Bridge** para las VLANs 10 y 50
* **Switch1** y **Switch2** son VTP Client y uno de sus puertos hacia el loop debería terminar en estado **blocking** por cada VLAN

**VLANs de ejemplo**

* VLAN 10 → `lab`
* VLAN 50 → `stp`

Se usan dos VLANs para demostrar que el Root Bridge **se configura por VLAN**, y que un mismo switch puede forzarse como root de varias VLANs a la vez.

---

## 0. VTP + Creación de VLANs (SW_Server)

```bash
enable
conf t

vtp version 2
vtp mode server
vtp domain semana4
vtp password 123

vlan 10
 name lab
exit

vlan 50
 name stp
exit
```

### Switch1 y Switch2 (VTP Client)

```bash
enable
conf t

vtp version 2
vtp mode client
vtp domain semana4
vtp password 123
```

### Trunk y Access de interfaces
```bash
enable
conf t

interface fa0/1 

# Trunk
switchport mode trunk
switchport trunk allowed vlan 10,50

# Access
switchport mode access
switchport access vlan 10
```

---

## 1. Forzar el Root Bridge para varias VLANs (SW_Server)

Por defecto, STP elige el Root Bridge según el Bridge ID más bajo (prioridad + MAC), lo cual no siempre coincide con el switch que uno quiere que sea la raíz. Por eso se fuerza manualmente, **VLAN por VLAN**, línea por línea. Además de las VLANs 10 y 50, también se dejó configurada la **VLAN 1** (la VLAN por defecto) como root en SW_Server:

```bash
enable
conf t

spanning-tree vlan 1 root primary
spanning-tree vlan 10 root primary
spanning-tree vlan 50 root primary
```

> `root primary` ajusta automáticamente la prioridad de SW_Server (normalmente a 24576, o menos si hace falta) para que gane la elección frente a Switch1 y Switch2, en cada VLAN indicada.

---

## 2. Verificar el Switch Root y el estado de STP

### Ver el Root Bridge de cada VLAN

```bash
show spanning-tree vlan 1
show spanning-tree vlan 10
show spanning-tree vlan 50
```

En SW_Server, las tres salidas deben mostrar:

```
Root ID    Priority    24576 (o el valor configurado)
           This bridge is the root
```

En Switch1 y Switch2, el mismo comando (para VLAN 1, VLAN 10 y VLAN 50) debe mostrar el **Root ID** apuntando a la MAC/prioridad de SW_Server, confirmando que lo reconocen como raíz en las tres VLANs.

### Ver detalle completo de STP

```bash
show spanning-tree detail
```

---

## 3. Cambiar el modo a Rapid PVST+ (opcional)

Por defecto Packet Tracer usa **PVST+** para poder cambiarlo a **Rapid PVST+** se utiliza la siguiente configuracio:


```bash
# Este cambio se hace en los tres switches
# Si solo se cambia en uno, ese switch sigue interoperando con los deás pero cae a modo de compatibilidad PVST+

enable
conf t

spanning-tree mode rapid-pvst
end
wr
```

> No hace falta volver a forzar el root: las VLANs 1, 10 y 50 mantienen a SW_Server como Root Bridge, solo cambia el protocolo que usa STP para calcular y converger la topología.

---

## Troubleshooting rápido

* Si ningún switch queda bloqueado, revisa que realmente exista el loop físico (los tres enlaces del triángulo conectados).
* Si el Root Bridge no es el esperado en alguna VLAN, confirma la prioridad con `show spanning-tree vlan <número>` en los tres switches — puede que otro switch tenga una prioridad aún más baja para esa VLAN en particular.
* Recuerda que forzar el root para una VLAN **no afecta automáticamente** a las demás: cada VLAN tiene su propia instancia de STP (PVST+), por eso hay que repetir el comando `spanning-tree vlan <número> root primary` para cada una (1, 10 y 50).
* Si un puerto queda en estado **err-disabled** en vez de **blocking**, revisa que no haya un problema de duplex/velocidad entre los switches, no relacionado con STP.




