## Маршрутизация на основе политик (PBR)

#### Маршрутизация в Чокурдах

Распределим трафик между VLAN  200 и 201. 
Трафик из VLAN 200 пустим через R26.
Трафик из VLAN 201 пустим через R25.

```
ip access-list extended ACL_FROM_V200
 permit ip 10.200.0.0 0.0.0.255 any
 exit

ip access-list extended ACL_FROM_V201
 permit ip 10.201.0.0 0.0.0.255 any
 exit

route-map PBR_ISP permit 10
 remark Traffic from VLAN 200 to R26
 math ip address ACL_FROM_V200
 set ip next-hop 80.10.0.6

route-map PBR_ISP permit 20
 remark traffic from VLAN 201 to R25
 match ip address ACL_FROM_V201
 set ip next-hop 80.10.0.2
 exit

interface range Ethernet0/0-1
 ip nat enable
 ip policy route-map PBR_ISP
```