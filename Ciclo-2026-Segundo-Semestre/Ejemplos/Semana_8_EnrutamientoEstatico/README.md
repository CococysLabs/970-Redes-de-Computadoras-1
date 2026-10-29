# Semana 8 — Ejemplo demostrativo: enrutamiento estático

**Curso:** Redes de Computadoras 1  
**Unidad 3:** Dispositivos y protocolos de la capa 3  
**Herramienta:** Cisco Packet Tracer

## Objetivo

Configurar y comprobar la comunicación entre tres LAN conectadas por tres routers mediante rutas estáticas, observando las rutas conectadas, locales y estáticas de la tabla de enrutamiento. Practicar el diagnóstico de fallas de conectividad.

## Topología

Topología lineal: `PC0 — SW1 — R1 — R2 — R3 — SW3 — PC2`, con una LAN adicional conectada a R2 (`R2 — SW2 — PC1`).

- R1 `Serial2/0` — R2 `Serial2/0`: `10.0.12.0/30`.
- R2 `Serial3/0` — R3 `Serial2/0`: `10.0.23.0/30`.
- LAN izquierda: `192.168.10.0/24`.
- LAN central: `192.168.20.0/24`.
- LAN derecha: `192.168.30.0/24`.

> **Nota de interfaces:** se utilizan los nombres que muestra la topología compartida. Los extremos con reloj DCE son R1 `Serial2/0` y R2 `Serial3/0`; si el cableado cambia, verifica DCE con `show controllers serial 2/0` o `show controllers serial 3/0` y aplica `clock rate` solo en el extremo DCE.

## Tabla de direccionamiento

| Dispositivo | Interfaz | IPv4 | Máscara |
|---|---|---|---|
| R1 | Fa0/0 | 192.168.10.1 | 255.255.255.0 |
| R1 | Se2/0 | 10.0.12.1 | 255.255.255.252 |
| R2 | Se2/0 | 10.0.12.2 | 255.255.255.252 |
| R2 | Se3/0 | 10.0.23.1 | 255.255.255.252 |
| R2 | Fa0/0 | 192.168.20.1 | 255.255.255.0 |
| R3 | Se2/0 | 10.0.23.2 | 255.255.255.252 |
| R3 | Fa0/0 | 192.168.30.1 | 255.255.255.0 |
| PC0 | Fa0 | 192.168.10.10 | 255.255.255.0 |
| PC1 | Fa0 | 192.168.20.10 | 255.255.255.0 |
| PC2 | Fa0 | 192.168.30.10 | 255.255.255.0 |

Gateways de las computadoras: PC0 `192.168.10.1`, PC1 `192.168.20.1`, PC2 `192.168.30.1`.

## 1. Configuración de interfaces

Para observar el efecto del enrutamiento estático, **primero configura las interfaces y realiza las pruebas iniciales sin agregar rutas**. Más adelante configura las rutas del apartado 3.

### R1

```cisco
enable
configure terminal
hostname R1
interface FastEthernet0/0
 ip address 192.168.10.1 255.255.255.0
 no shutdown
exit
interface Serial2/0
 ip address 10.0.12.1 255.255.255.252
 clock rate 64000
 no shutdown
exit
end
write memory
```

### R2

```cisco
enable
configure terminal
hostname R2
interface FastEthernet0/0
 ip address 192.168.20.1 255.255.255.0
 no shutdown
exit
interface Serial2/0
 ip address 10.0.12.2 255.255.255.252
 no shutdown
exit
interface Serial3/0
 ip address 10.0.23.1 255.255.255.252
 clock rate 64000
 no shutdown
exit
end
write memory
```

### R3

```cisco
enable
configure terminal
hostname R3
interface FastEthernet0/0
 ip address 192.168.30.1 255.255.255.0
 no shutdown
exit
interface Serial2/0
 ip address 10.0.23.2 255.255.255.252
 no shutdown
exit
end
write memory
```

## 2. Switches y computadoras

Los switches solo trabajan en capa 2 y no necesitan enrutamiento ni VLAN adicionales. El puerto `Fa0/1` conecta al router y el `Fa0/2` conecta a la PC.

### SW1

```cisco
enable
configure terminal
hostname SW1
interface FastEthernet0/1
 switchport mode access
 no shutdown
exit
interface FastEthernet0/2
 switchport mode access
 no shutdown
exit
end
write memory
```

### SW2

```cisco
enable
configure terminal
hostname SW2
interface FastEthernet0/1
 switchport mode access
 no shutdown
exit
interface FastEthernet0/2
 switchport mode access
 no shutdown
exit
end
write memory
```

### SW3

```cisco
enable
configure terminal
hostname SW3
interface FastEthernet0/1
 switchport mode access
 no shutdown
exit
interface FastEthernet0/2
 switchport mode access
 no shutdown
exit
end
write memory
```

En cada computadora: **Desktop → IP Configuration**.

| PC | IP | Máscara | Puerta de enlace |
|---|---|---|---|
| PC0 | 192.168.10.10 | 255.255.255.0 | 192.168.10.1 |
| PC1 | 192.168.20.10 | 255.255.255.0 | 192.168.20.1 |
| PC2 | 192.168.30.10 | 255.255.255.0 | 192.168.30.1 |

### Prueba inicial antes de configurar rutas

En los routers ejecuta:

```cisco
show ip interface brief
show ip route
```

Comprueba que las interfaces estén **up/up**. En `show ip route` deben aparecer inicialmente las redes conectadas (`C`) y direcciones locales (`L`). Prueba `ping` entre interfaces seriales vecinas; un ping de PC0 a PC2 no debería funcionar todavía porque faltan rutas a las redes remotas y de retorno.

## 3. Configurar rutas estáticas

Sintaxis general:

```cisco
ip route RED_DESTINO MASCARA SIGUIENTE_SALTO
```

### R1

```cisco
enable
configure terminal
ip route 192.168.20.0 255.255.255.0 10.0.12.2
ip route 192.168.30.0 255.255.255.0 10.0.12.2
end
write memory
```

### R2

```cisco
enable
configure terminal
ip route 192.168.10.0 255.255.255.0 10.0.12.1
ip route 192.168.30.0 255.255.255.0 10.0.23.2
end
write memory
```

### R3

```cisco
enable
configure terminal
ip route 192.168.10.0 255.255.255.0 10.0.23.1
ip route 192.168.20.0 255.255.255.0 10.0.23.1
end
write memory
```

**Interpretación:** R1 entrega el tráfico remoto a R2; R2 dirige el tráfico a R1 o R3 según el destino; R3 usa R2 para llegar a las dos LAN restantes. Cada router necesita rutas de ida y de regreso para que el intercambio funcione.

## 4. Validar la conectividad

En cada router:

```cisco
show ip route
show ip interface brief
show running-config
```

En la tabla de rutas, interpreta: `C` red conectada, `L` dirección local del router y `S` ruta estática.

En **PC0 → Desktop → Command Prompt**:

```text
ping 192.168.10.1
ping 192.168.20.10
ping 192.168.30.10
tracert 192.168.30.10
```

En **PC1**:

```text
ping 192.168.10.10
ping 192.168.30.10
```

En **PC2**:

```text
ping 192.168.10.10
tracert 192.168.10.10
```

Los pings entre las tres LAN deben responder cuando las interfaces, gateways y rutas estén correctamente configurados. En `tracert` observa el recorrido, teniendo en cuenta que algunos dispositivos pueden no responder a los mensajes del rastreo.

## 5. Diagnóstico para demostrar en clase

1. **Eliminar temporalmente una ruta.** En R2:

   ```cisco
   configure terminal
   no ip route 192.168.30.0 255.255.255.0 10.0.23.2
   end
   show ip route
   ```

   Prueba de nuevo la comunicación entre PC0 y PC2. Después restablece:

   ```cisco
   configure terminal
   ip route 192.168.30.0 255.255.255.0 10.0.23.2
   end
   ```

2. **Revisar ruta de retorno.** Explica que definir solo la ruta de ida no garantiza que la respuesta pueda volver al origen.
3. **Revisar interfaces seriales.** Si muestran `administratively down`, falta `no shutdown`; si el estado del enlace sigue caído, revisa el cable y el reloj DCE.
4. **Revisar gateways.** Una PC con gateway erróneo puede comunicarse localmente, pero no con otras LAN.

## 6. Archivos sugeridos para el repositorio

```text
semana-08-rutas-estaticas/
├── README.md
├── topologia_rutas_estaticas_base.pkt
└── topologia_rutas_estaticas_resuelta.pkt
```

- **Base:** conexiones e IP de interfaces y PCs; **sin rutas estáticas**.
- **Resuelta:** mismas conexiones e IP; con las seis rutas estáticas y pruebas satisfactorias.

## Preguntas de repaso

- ¿Por qué PC0 no puede alcanzar PC2 antes de crear las rutas estáticas?
- ¿Qué representa el siguiente salto `10.0.12.2` configurado en R1?
- ¿Por qué R3 necesita una ruta hacia `192.168.10.0/24`?
- ¿Qué diferencia hay entre rutas `C`, `L` y `S`?
- ¿Qué comando permite verificar la ruta elegida por un router?
- ¿Qué ocurriría si falla el enlace serial entre R2 y R3?

---

**Criterio de éxito:** se observan las rutas estáticas en las tablas correspondientes, y PC0, PC1 y PC2 alcanzan las otras LAN mediante `ping`.
