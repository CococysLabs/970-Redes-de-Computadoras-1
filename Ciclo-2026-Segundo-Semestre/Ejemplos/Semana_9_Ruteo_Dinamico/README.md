# Semana 9 · Ejemplo demostrativo: OSPF de área única

Laboratorio de Redes de Computadoras 1 (970) · Escuela de Ingeniería en Ciencias y Sistemas · USAC

Tres sucursales sin comunicación entre sí se interconectan configurando **OSPF de área única (área 0)**. Al terminar, cada router aprende de forma dinámica las redes de las otras sucursales y las tres PC se comunican de extremo a extremo.

## Contenido de la carpeta

| Archivo | Descripción |
|---|---|
| `S9_OSPF_inicial.pkt` | Topología con direccionamiento IP listo y **sin** enrutamiento (punto de partida de la demostración). |
| `S9_OSPF_final.pkt` | Topología con OSPF configurado y verificado. |
| `comandos_S9_OSPF.txt` | Todos los comandos: configuración base, OSPF, verificación y errores comunes. |
| `topologia.png` | Diagrama de la topología. |

Requiere **Cisco Packet Tracer 8.2 o superior**.

## Topología

![Topología del ejemplo](topologia.png)

## Tabla de direccionamiento

| Equipo | Interfaz | Dirección IP | Máscara | Gateway | Conecta con |
|---|---|---|---|---|---|
| R1 | G0/0 | 192.168.1.1 | 255.255.255.0 | — | S1 G0/1 |
| R1 | G0/1 | 10.0.12.1 | 255.255.255.252 | — | R2 G0/1 |
| R2 | G0/0 | 192.168.2.1 | 255.255.255.0 | — | S2 G0/1 |
| R2 | G0/1 | 10.0.12.2 | 255.255.255.252 | — | R1 G0/1 |
| R2 | G0/2 | 10.0.23.1 | 255.255.255.252 | — | R3 G0/1 |
| R3 | G0/0 | 192.168.3.1 | 255.255.255.0 | — | S3 G0/1 |
| R3 | G0/1 | 10.0.23.2 | 255.255.255.252 | — | R2 G0/2 |
| PC1 | NIC | 192.168.1.10 | 255.255.255.0 | 192.168.1.1 | S1 F0/1 |
| PC2 | NIC | 192.168.2.10 | 255.255.255.0 | 192.168.2.1 | S2 F0/1 |
| PC3 | NIC | 192.168.3.10 | 255.255.255.0 | 192.168.3.1 | S3 F0/1 |

Todos los cables son de cobre directo (*Copper Straight-Through*).

## Configuración OSPF

Wildcard = `255.255.255.255 − máscara` → /24 = `0.0.0.255`, /30 = `0.0.0.3`

```
! R1
router ospf 1
 router-id 1.1.1.1
 network 192.168.1.0 0.0.0.255 area 0
 network 10.0.12.0 0.0.0.3 area 0
 passive-interface GigabitEthernet0/0

! R2
router ospf 1
 router-id 2.2.2.2
 network 192.168.2.0 0.0.0.255 area 0
 network 10.0.12.0 0.0.0.3 area 0
 network 10.0.23.0 0.0.0.3 area 0
 passive-interface GigabitEthernet0/0

! R3
router ospf 1
 router-id 3.3.3.3
 network 192.168.3.0 0.0.0.255 area 0
 network 10.0.23.0 0.0.0.3 area 0
 passive-interface GigabitEthernet0/0
```

`passive-interface` evita enviar mensajes Hello de OSPF hacia las PC, pero la red de la LAN se sigue anunciando a los demás routers.

## Verificación

| Comando | Qué se espera |
|---|---|
| `show ip ospf neighbor` | En R2, dos vecinos (1.1.1.1 y 3.3.3.3) en estado **FULL**. |
| `show ip route ospf` | En R1, rutas `O` hacia 192.168.2.0/24, 192.168.3.0/24 y 10.0.23.0/30. |
| `show ip protocols` | Router ID, redes declaradas e interfaz pasiva. |
| `ping 192.168.3.10` (desde PC1) | Respuestas exitosas. |
| `tracert 192.168.3.10` (desde PC1) | 192.168.1.1 → 10.0.12.2 → 10.0.23.2 → 192.168.3.10 |

## Errores comunes

- Usar la máscara de subred en lugar de la wildcard en el comando `network`.
- Olvidar declarar los enlaces entre routers: no se forma la adyacencia.
- Configurar áreas distintas en los dos extremos de un enlace.
- Poner como pasiva la interfaz hacia otro router en lugar de la interfaz de la LAN.
