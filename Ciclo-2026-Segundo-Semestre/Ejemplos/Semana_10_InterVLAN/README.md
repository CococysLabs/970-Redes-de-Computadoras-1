# Semana 10 · Ejemplo demostrativo: Enrutamiento InterVLAN

Laboratorio de Redes de Computadoras 1 (970) · Escuela de Ingeniería en Ciencias y Sistemas · USAC

Tres departamentos (ADMIN, RRHH e IT) están separados en VLAN y no pueden comunicarse entre sí. El problema se resuelve de dos formas para compararlas:

- **Parte A · Router-on-a-stick:** un router ISR4331 enruta las VLAN con subinterfaces sobre un solo enlace troncal.
- **Parte B · SVI:** un switch capa 3 (3560) enruta las VLAN internamente con interfaces `interface vlan`.

Las dos partes usan los mismos switches de acceso, VLAN y direcciones. Lo único que cambia es el equipo que enruta.

## Contenido de la carpeta

| Archivo | Descripción |
|---|---|
| `S10_A_ROAS.pkt` | Parte A configurada: router-on-a-stick con R1. |
| `S10_B_SVI.pkt` | Parte B configurada: SVI en MLS1. |
| `comandos_S10_InterVLAN.txt` | Todos los comandos: switches, R1, MLS1, verificación y errores comunes. |
| `topologia.png` | Diagrama de las dos partes. |

Requiere **Cisco Packet Tracer 8.2 o superior**.

## Topología

![Topología del ejemplo](topologia.png)

## VLAN y direccionamiento (VLSM sobre 192.168.10.0/24)

| VLAN | Nombre | Red | Máscara | Gateway |
|---|---|---|---|---|
| 10 | ADMIN | 192.168.10.0/26 | 255.255.255.192 | 192.168.10.1 |
| 20 | RRHH | 192.168.10.64/27 | 255.255.255.224 | 192.168.10.65 |
| 30 | IT | 192.168.10.96/28 | 255.255.255.240 | 192.168.10.97 |
| 99 | NATIVA | — | — | — |

| Equipo | IP | VLAN | Conectado a |
|---|---|---|---|
| PC1 | 192.168.10.10 | 10 | S1 F0/1 |
| PC2 | 192.168.10.70 | 20 | S1 F0/2 |
| PC3 | 192.168.10.100 | 30 | S2 F0/1 |
| PC4 | 192.168.10.11 | 10 | S2 F0/2 |

Enlaces: S1 G0/2 ↔ S2 G0/1 (troncal) · S1 G0/1 ↔ R1 G0/0/0 en la Parte A, o ↔ MLS1 G0/1 en la Parte B (troncal). Todos los cables son de cobre directo.

## Parte A · Router-on-a-stick

```
interface GigabitEthernet0/0/0
 no shutdown
interface GigabitEthernet0/0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.192
interface GigabitEthernet0/0/0.20
 encapsulation dot1Q 20
 ip address 192.168.10.65 255.255.255.224
interface GigabitEthernet0/0/0.30
 encapsulation dot1Q 30
 ip address 192.168.10.97 255.255.255.240
interface GigabitEthernet0/0/0.99
 encapsulation dot1Q 99 native
```

## Parte B · SVI en switch capa 3

```
ip routing
interface GigabitEthernet0/1
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk native vlan 99
interface vlan 10
 ip address 192.168.10.1 255.255.255.192
 no shutdown
interface vlan 20
 ip address 192.168.10.65 255.255.255.224
 no shutdown
interface vlan 30
 ip address 192.168.10.97 255.255.255.240
 no shutdown
```

La configuración completa de S1, S2, R1 y MLS1 está en `comandos_S10_InterVLAN.txt`.

## Comparación

| | Router-on-a-stick | SVI (switch capa 3) |
|---|---|---|
| Equipo que enruta | Router externo | El mismo switch |
| Gateway de cada VLAN | Subinterfaz `G0/0/0.X` | `interface vlan X` |
| Comando clave | `encapsulation dot1Q X` | `ip routing` |
| Tráfico entre VLAN | Sube y baja por un solo enlace troncal (cuello de botella) | Se enruta dentro del switch, sin salir a otro equipo |
| Uso típico | Redes pequeñas o con pocos recursos | Redes medianas y grandes |

## Verificación

| Comando | Qué se espera |
|---|---|
| `show vlan brief` (S1, S2) | Cada puerto de acceso en su VLAN. |
| `show interfaces trunk` (S1, S2) | Troncales activas con VLAN nativa 99. |
| `show ip interface brief` (R1 o MLS1) | Subinterfaces o SVI en `up/up` con su IP. |
| `show ip route connected` (R1 o MLS1) | Tres redes `C`, una por VLAN. |
| `ping 192.168.10.70` desde PC1 | Exitoso (ADMIN → RRHH). |
| `tracert 192.168.10.100` desde PC1 | Primer salto 192.168.10.1 y luego el destino. |

## Errores comunes

- El número en `encapsulation dot1Q` no coincide con la VLAN del switch.
- Falta `no shutdown` en la interfaz física del router.
- Falta `ip routing` en el switch capa 3.
- La SVI queda `down/down` porque la VLAN no existe en el switch capa 3 o no hay una troncal activa.
- Máscara equivocada en una PC (por ejemplo /24 en lugar de /26).
