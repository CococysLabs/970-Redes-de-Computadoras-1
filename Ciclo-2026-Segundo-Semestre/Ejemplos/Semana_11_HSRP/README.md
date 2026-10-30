# Configuración de la topología con HSRP y OSPF

## HSRP1
```bash
enable
configure terminal
hostname HSRP1

interface fa0/0
 ip address 192.168.1.2 255.255.255.0
 standby 1 ip 192.168.1.1
 standby 1 priority 110
 standby 1 preempt
 no shutdown

interface se2/0
 ip address 10.0.20.1 255.255.255.252
 no shutdown

router ospf 1
 network 192.168.1.0 0.0.0.255 area 0
 network 10.0.20.0 0.0.0.3 area 0
 passive-interface fa0/0

end
write memory
```

## HSRP2
```bash
enable
configure terminal
hostname HSRP2

interface fa0/0
 ip address 192.168.1.3 255.255.255.0
 standby 1 ip 192.168.1.1
 standby 1 priority 100
 standby 1 preempt
 no shutdown

interface se2/0
 ip address 10.0.10.1 255.255.255.252
 no shutdown

router ospf 1
 network 192.168.1.0 0.0.0.255 area 0
 network 10.0.10.0 0.0.0.3 area 0
 passive-interface fa0/0

end
write memory
```

## CENTRAL
```bash
enable
configure terminal
hostname CENTRAL

interface se3/0
 ip address 10.0.10.2 255.255.255.252
 no shutdown

interface se2/0
 ip address 10.0.20.2 255.255.255.252
 no shutdown

interface se6/0
 ip address 10.0.30.1 255.255.255.252
 no shutdown

router ospf 1
 network 10.0.10.0 0.0.0.3 area 0
 network 10.0.20.0 0.0.0.3 area 0
 network 10.0.30.0 0.0.0.3 area 0

end
write memory
```

## R1
```bash
enable
configure terminal
hostname R1

interface se2/0
 ip address 10.0.30.2 255.255.255.252
 no shutdown

interface fa0/0
 ip address 192.168.2.1 255.255.255.0
 no shutdown

router ospf 1
 network 10.0.30.0 0.0.0.3 area 0
 network 192.168.2.0 0.0.0.255 area 0
 passive-interface fa0/0

end
write memory
```

## SW1
```bash
enable
configure terminal
hostname SW1

vlan 10
 name LAN_HSRP

interface fa0/1
 switchport mode access
 switchport access vlan 10
 no shutdown

interface fa1/1
 switchport mode access
 switchport access vlan 10
 no shutdown

interface fa2/1
 switchport mode access
 switchport access vlan 10
 no shutdown

end
write memory
```

## SW2
```bash
enable
configure terminal
hostname SW2

vlan 10
 name LAN_DERECHA

interface fa0/1
 switchport mode access
 switchport access vlan 10
 no shutdown

interface fa2/1
 switchport mode access
 switchport access vlan 10
 no shutdown

end
write memory
```

## Configuración de las PCs

### PC2
- IP Address: `192.168.1.10`
- Subnet Mask: `255.255.255.0`
- Default Gateway: `192.168.1.1`

### PC3
- IP Address: `192.168.2.10`
- Subnet Mask: `255.255.255.0`
- Default Gateway: `192.168.2.1`

## Comandos de verificación
```bash
show ip route
show standby brief
```

## Pruebas de conectividad

### Desde PC2
```bash
ping 192.168.1.1
ping 192.168.2.10
```

### Desde PC3
```bash
ping 192.168.2.1
ping 192.168.1.10
```
