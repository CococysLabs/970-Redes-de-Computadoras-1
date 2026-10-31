# Semana 2: Conceptos generales y prueba de ping

## Objetivo

Comprender cómo se comprueba la comunicación básica entre dos equipos mediante
`ping`, utilizando una topología punto a punto formada únicamente por dos PC
con direcciones IP dentro de la misma red.

El archivo [`EjemploPing.pkt`](./EjemploPing.pkt) contiene la práctica para
realizar esta comprobación en Cisco Packet Tracer.

## Topología

La topología tiene solo dos computadoras conectadas directamente:

```text
PC1  ───────────────────  PC2
      enlace punto a punto
```

No se utiliza switch, router ni ningún otro dispositivo intermedio. El enlace
puede representarse con un cable cruzado, ya que las dos PC se conectan
directamente entre sí.

## Direccionamiento IP

| Dispositivo | Dirección IP | Máscara de subred | Puerta de enlace |
|-------------|--------------|-------------------|------------------|
| PC0         | `192.168.1.2` | `255.255.255.0` (`/24`) | No se configura |
| PC1         | `192.168.1.3` | `255.255.255.0` (`/24`) | No se configura |

Las dos direcciones pertenecen a la red `192.168.1.0/24`. Como la
comunicación ocurre directamente entre los equipos y no existe un router, no
es necesario configurar una puerta de enlace predeterminada.

## Configuración en Packet Tracer

En cada PC se debe abrir **Desktop > IP Configuration** y asignar los valores
correspondientes:

### PC0

- **IP Address:** `192.168.1.2`
- **Subnet Mask:** `255.255.255.0`

### PC1

- **IP Address:** `192.168.1.3`
- **Subnet Mask:** `255.255.255.0`

Después de configurar las direcciones, se debe esperar a que el enlace físico
aparezca activo. Una conexión activa indica que existe comunicación en la
capa física, pero todavía se debe comprobar la comunicación lógica con
`ping`.

## ¿Qué es `ping`?

`ping` es una herramienta de diagnóstico que comprueba si un equipo puede
alcanzar a otro mediante la red. Utiliza mensajes **ICMP**:

1. El equipo origen envía un **ICMP Echo Request** a la dirección IP destino.
2. El equipo destino responde con un **ICMP Echo Reply**.
3. El equipo origen mide el tiempo de respuesta y muestra si los paquetes
   llegaron correctamente.

En una primera comunicación, el equipo también puede utilizar **ARP** para
averiguar la dirección MAC asociada a la IP destino. Después, el paquete ICMP
se entrega mediante el enlace directo entre las dos PC.

## Realizar la prueba

Desde **PC1**, abrir **Desktop > Command Prompt** y ejecutar:

```text
ping 192.168.1.3
```

También se puede comprobar la comunicación en sentido contrario desde PC2:

```text
ping 192.168.1.2
```

Una respuesta correcta debe mostrar mensajes similares a:

```text
Reply from 192.168.1.3: bytes=32 time<1ms TTL=128
```

El resultado esperado es recibir respuestas de los cuatro paquetes enviados,
sin pérdida de paquetes. El tiempo puede variar según la simulación.

## Cómo explicar el resultado

- **Reply from...**: la PC destino respondió al mensaje ICMP.
- **bytes=32**: tamaño de los datos enviados en la solicitud.
- **time<1ms**: tiempo aproximado que tardó la respuesta; en un enlace
  directo normalmente es muy bajo.
- **TTL**: valor de tiempo de vida incluido en la respuesta IP.
- **Lost = 0 (0% loss)**: todos los paquetes llegaron correctamente.

## Si el ping falla

Revisar los siguientes puntos:

1. Confirmar que el cable esté conectado entre las dos PC y que los
   indicadores de enlace estén activos.
2. Verificar que PC0 tenga `192.168.1.2` y PC1 tenga `192.168.1.3`.
3. Confirmar que ambas PC utilicen la máscara
   `255.255.255.0`.
4. Comprobar que no se haya configurado una IP duplicada.
5. Repetir el comando `ping` después de que el enlace termine de activarse.