# Ejemplo demostrativo — Direccionamiento IPv4, FLSM y VLSM

**Curso:** Redes de Computadoras 1  
**Unidad 3:** Dispositivos y protocolos en la Capa 3  
**Semana:** 7  
**Tema:** Direccionamiento IP, máscaras, CIDR y subnetting  
**Valor de la sesión:** Precisión

## Objetivo

Resolver y verificar cálculos de direccionamiento IPv4, identificar la red a la que pertenece una dirección, dividir un bloque mediante FLSM y asignar subredes de diferentes tamaños mediante VLSM. Cada resultado se registrará en la plantilla de Excel correspondiente.

## Archivos del repositorio

- `Plantilla_Base_Ejemplo_Semana_7_Subnetting.xlsx`: archivo inicial con celdas por completar.
- `Plantilla_Resuelta_Ejemplo_Semana_7_Subnetting.xlsx`: solución de referencia para comparar los resultados al finalizar.

Trabaja **en una copia del archivo base**. No utilices el archivo resuelto hasta concluir los ejercicios.

## Conceptos y fórmulas que necesitarás

- **Prefijo `/n`:** `n` bits representan la red; `32 - n` bits quedan para hosts.
- **Direcciones del bloque:** `2^(32 - n)`.
- **Hosts utilizables en subredes IPv4 convencionales:** `2^(32 - n) - 2` (se excluyen dirección de red y broadcast; hay excepciones especiales como `/31` y `/32`).
- **Tamaño de salto:** `256 - valor del octeto de la máscara donde ocurre la división`.
- **Dirección de red:** primera dirección del bloque correspondiente.
- **Broadcast:** última dirección del bloque.
- **Primer y último host:** dirección de red + 1 y broadcast - 1, respectivamente, en estas subredes convencionales.
- **FLSM:** todas las subredes utilizan el mismo prefijo.
- **VLSM:** los prefijos se ajustan a los requisitos de cada segmento, sin solapar rangos.

### Cómo calcular la red con bits (operación AND)

1. Convierte la dirección IP y la máscara a binario.
2. Aplica `AND` bit por bit: el resultado es `1` solo cuando **ambos** bits son `1`.
3. Convierte el resultado nuevamente a decimal: es la **dirección de red**.

## Hoja «01 CIDR» — Identificar una subred

**Datos iniciales:** `192.168.40.70/26`.

1. Escribe la dirección IPv4 y el prefijo que aparecen en la plantilla.
2. Convierte `/26` a máscara decimal: 26 bits en `1` equivalen a `255.255.255.192`.
3. Calcula los bits de host: `32 - 26 = 6`.
4. Calcula el tamaño del bloque: `2^6 = 64` direcciones, equivalente a un salto de `256 - 192 = 64` en el último octeto.
5. Identifica los bloques del último octeto: `0–63`, `64–127`, `128–191`, `192–255`.
6. Localiza el valor `70`: corresponde al intervalo `64–127`.
7. Completa red, primer host, último host, broadcast y cantidad de hosts utilizables.
8. En la columna «Procedimiento / verificación» anota cómo llegaste a cada resultado.

**Comprobación después de resolver:**

| Dato | Resultado |
|---|---|
| IP | `192.168.40.70/26` |
| Máscara | `255.255.255.192` |
| Bits de host | `6` |
| Tamaño del bloque | `64` |
| Dirección de red | `192.168.40.64` |
| Primer host | `192.168.40.65` |
| Último host | `192.168.40.126` |
| Broadcast | `192.168.40.127` |
| Hosts utilizables | `62` |

## Hoja «02 FLSM» — Dividir una red en cuatro partes iguales

**Red base:** `192.168.40.0/24`  
**Requerimiento:** obtener **4 subredes iguales**.

1. Para formar cuatro subredes, toma `2` bits de la porción de host, porque `2² = 4`.
2. El nuevo prefijo es `/24 + 2 = /26` y la máscara es `255.255.255.192`.
3. Cada subred tiene `64` direcciones totales y `62` hosts utilizables.
4. Completa las cuatro filas de la tabla, avanzando de 64 en 64 en el último octeto.
5. Para cada fila, anota red/prefijo, máscara, primer host, último host, broadcast y hosts utilizables.

**Resultados de comprobación:**

| No. | Red/prefijo | Primera IP | Última IP | Broadcast | Hosts |
|---:|---|---|---|---|---:|
| 1 | `192.168.40.0/26` | `192.168.40.1` | `192.168.40.62` | `192.168.40.63` | 62 |
| 2 | `192.168.40.64/26` | `192.168.40.65` | `192.168.40.126` | `192.168.40.127` | 62 |
| 3 | `192.168.40.128/26` | `192.168.40.129` | `192.168.40.190` | `192.168.40.191` | 62 |
| 4 | `192.168.40.192/26` | `192.168.40.193` | `192.168.40.254` | `192.168.40.255` | 62 |

Todas las filas utilizan la misma máscara: `255.255.255.192`.

## Hoja «03 VLSM» — Distribuir bloques según los hosts

**Red base:** `192.168.40.0/24`.

| Segmento | Hosts necesarios |
|---|---:|
| Administración | 60 |
| Ventas | 30 |
| Soporte | 12 |
| Gerencia | 6 |
| Enlace punto a punto | 2 |

1. Ordena los requisitos de mayor a menor (ya están ordenados en la plantilla).
2. Para cada segmento, encuentra el **bloque más pequeño** cuya capacidad utilizable alcance la cantidad requerida.
3. Asigna primero el bloque mayor, empezando en `192.168.40.0`.
4. Alinea cada nueva subred al tamaño de su bloque; comienza **después del broadcast de la anterior**.
5. Completa red/prefijo, máscara, primer y último host, broadcast y capacidad utilizable.
6. Verifica que ninguna dirección pertenezca a dos segmentos.

**Resultados de comprobación:**

| Segmento | Red/prefijo | Máscara | Primera IP | Última IP | Broadcast | Capacidad |
|---|---|---|---|---|---|---:|
| Administración | `192.168.40.0/26` | `255.255.255.192` | `192.168.40.1` | `192.168.40.62` | `192.168.40.63` | 62 |
| Ventas | `192.168.40.64/27` | `255.255.255.224` | `192.168.40.65` | `192.168.40.94` | `192.168.40.95` | 30 |
| Soporte | `192.168.40.96/28` | `255.255.255.240` | `192.168.40.97` | `192.168.40.110` | `192.168.40.111` | 14 |
| Gerencia | `192.168.40.112/29` | `255.255.255.248` | `192.168.40.113` | `192.168.40.118` | `192.168.40.119` | 6 |
| Enlace punto a punto | `192.168.40.120/30` | `255.255.255.252` | `192.168.40.121` | `192.168.40.122` | `192.168.40.123` | 2 |

> Para el enlace punto a punto se usa `/30` porque este ejercicio emplea la convención de dos direcciones IPv4 utilizables. `/31` también existe para enlaces punto a punto compatibles, pero no se utiliza aquí.

**Rango aún no asignado en este ejercicio:** `192.168.40.124` a `192.168.40.255`. Esta porción queda disponible para planificación futura; no equivale a una sola subred asignada automáticamente.

## Hoja «04 Validación» — Comprobar precisión

Registra **Sí/No** y una justificación breve para estas comprobaciones:

- Todas las subredes permanecen dentro de `192.168.40.0/24`.
- No existe solapamiento entre subredes.
- Cada segmento soporta su cantidad requerida de hosts.
- No se reservó como host la dirección de red ni la de broadcast.
- Las máscaras corresponden a los prefijos seleccionados.
- Cada bloque comienza en una dirección alineada a su tamaño.

## Errores frecuentes y cómo corregirlos

| Error | Cómo detectarlo o corregirlo |
|---|---|
| Confundir dirección IP con dirección de red | Aplicar la máscara AND a la IP y localizar el inicio del bloque. |
| Utilizar la dirección de broadcast como host | Reservar siempre el último valor del bloque para broadcast en estos ejercicios. |
| Contar todas las direcciones como hosts | Restar 2 a las direcciones totales para estas subredes IPv4 convencionales. |
| Elegir una subred muy pequeña en VLSM | Comprobar que `2^(bits de host) - 2` alcance los hosts solicitados. |
| Solapar subredes en VLSM | Empezar la siguiente asignación en el límite válido posterior al bloque anterior. |
| Confundir máscara decimal y prefijo | Por ejemplo, `/27` equivale a `255.255.255.224`. |

## Preguntas para la demostración

1. ¿Por qué `192.168.40.70/26` pertenece a la subred `192.168.40.64/26` y no a `192.168.40.0/26`?
2. Si necesitamos cuatro subredes iguales a partir de un `/24`, ¿cuántos bits debemos tomar?
3. ¿Por qué VLSM evita reservar bloques idénticos para departamentos de tamaños distintos?
4. ¿Qué error ocurre si asignamos `192.168.40.95` a un equipo de la subred `192.168.40.64/27`?

## Resultado esperado

Al finalizar, el archivo base debe contener **todas las tablas completadas**, cálculos verificables en la columna de procedimiento y la lista de validación revisada. Se compararán los resultados con el archivo de solución después de la demostración.

## Referencias

- RFC 1918 — Address Allocation for Private Internets: https://www.rfc-editor.org/rfc/rfc1918
- RFC 4632 — Classless Inter-domain Routing (CIDR): https://www.rfc-editor.org/rfc/rfc4632
- Cisco Networking Academy — Material de direccionamiento IPv4 y subnetting.
