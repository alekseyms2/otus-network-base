Лабораторная работа - Конфигурация безопасности коммутатора 


Часть 1. Настройка основного сетевого устройства

Топология сети

Инициализация устройств

Пример инициализации иаршрутизатора R1:

```
enable
erase startup-config
delete vlan.dat
reload
```

Шаг 2. Настройте маршрутизатор R1

Загрузить конфигурационный скрипт на R1:

```
enable
configure terminal
hostname R1
no ip domain lookup
ip dhcp excluded-address 192.168.10.1 192.168.10.9
ip dhcp excluded-address 192.168.10.201 192.168.10.202
!
ip dhcp pool Students
 network 192.168.10.0 255.255.255.0
 default-router 192.168.10.1
 domain-name CCNA2.Lab-11.6.1
!
interface Loopback0
 ip address 10.10.1.1 255.255.255.0
!
interface GigabitEthernet0/0/1
 description Link to S1
 ip dhcp relay information trusted
 ip address 192.168.10.1 255.255.255.0
 no shutdown
!
line con 0
 logging synchronous
 exec-timeout 0 0
```

Проверить текущую конфигурацию:

```
R1#show ip interface brief 
Interface              IP-Address      OK? Method Status                Protocol 
GigabitEthernet0/0/0   unassigned      YES NVRAM  administratively down down 
GigabitEthernet0/0/1   192.168.10.1    YES manual up                    up 
Loopback0              10.10.1.1       YES manual up                    up 
Vlan1                  unassigned      YES NVRAM  administratively down down
```

Шаг 3. Настройка и проверка основных параметров коммутатора

Общие параметры для обоих коммутаторов:

```
enable
configure terminal
no ip domain-lookup
ip default-gateway 192.168.10.1
```

Выполним настройку интерфейсов на коммутатора S1:

```
hostname S1
interface fastEthernet 0/5
 description Link to R1
!
interface fastEthernet 0/1
 description Link to S2
!
interface fastEthernet 0/6
 description Link to PC1
 exit
```

Выполним настройку интерфейсов на коммутатора S2:

```
hostname S2
interface fastEthernet 0/1
 description Link to S1
!
interface fastEthernet 0/18
 description Link to PC2
 exit
```

Часть 2. Настройка сетей VLAN на коммутаторах

Задачи:   
1. Конфигурация VLAN 10.  
2. Конфигурация SVI для VLAN 10.  
3. Конфигурация VLAN 333 с именем Native.  
4. Конфигурация VLAN 999 с именем ParkingLot.  

Пример настройки сетей VLAN на коммутаторе S1:

```
vlan 10
 name Management
!
interface vlan 10
 ip address 192.168.10.201 255.255.255.0
 exit
!
vlan 333
 name Native
!
vlan 999
 name ParkingLot   
 exit
```

Часть 3. Настройки безопасности коммутатора

Пример конфигурации магистрального порта FastEthernet 0/1 на коммутаторе S1:

```
configure terminal
interface fastEthernet 0/1
 switchport mode trunk
 switchport trunk native vlan 333
 exit
```

Пример отключения согласование DTP FastEthernet 0/1 на коммутаторе S1:

```
configure terminal
interface fastEthernet 0/1
 switchport nonegotiate
 exit
```

Шаг 2. Настройка портов доступа

* На коммутаторе S1 настроим FastEthernet0/5 и FastEthernet0/6 в качестве портов доступа и свяжем их с VLAN 10.
* На коммутаторе S2 настроим порт доступа FastEthernet0/18 и свяжем его с VLAN 10.

Пример настройки коммутатора S1:

```
configure terminal
interface range FastEthernet0/5-6
 switchport mode access
 switchport access vlan 10
 exit
```

Пример настройки коммутатора S2:

```
configure terminal
interface range FastEthernet0/18
 switchport mode access
 switchport access vlan 10
 exit
```

На коммутаторах S1 и S2 переместим неиспользуемые порты из VLAN 1 в VLAN 999 и отключим их

Пример настройки коммутатора S1:

```
configure terminal
interface range FastEthernet0/2-4, FastEthernet0/7-24, GigabitEthernet0/1-2
 switchport mode access
 switchport access vlan 999
 shutdown
 exit
```

Пример настройки коммутатора S2:

```
configure terminal
interface range FastEthernet0/2-17, FastEthernet0/19-24, GigabitEthernet0/1-2
 switchport mode access
 switchport access vlan 999
 shutdown
 exit
```

Конфигурирование безопасность интерфейсов на коммутаторах

Сконфигурировать безопасность интерфейса FastEthernet0/6 на коммутаторе S1 и интерфейс FastEthernet0/18 на коммутаторе S2. Конфигурацию выполним на примере интерфейса FastEthernet0/6.

Конфигурация безопасности интерфейса FastEthernet0/6 по умолчанию:

```
Port Security              : Disabled
Port Status                : Secure-down
Violation Mode             : Shutdown
Aging Time                 : 60 mins
Aging Type                 : Absolute
SecureStatic Address Aging : Disabled
Maximum MAC Addresses      : 1
Total MAC Addresses        : 0
Configured MAC Addresses   : 0
Sticky MAC Addresses       : 0
Last Source Address:Vlan   : 0000.0000.0000:0
Security Violation Count   : 0
```

Пример настройки безопасности интерфейса FastEthernet0/6 со следующими параметрами:
* Максимальное количество записей MAC-адресов: 3
* Режим безопасности: restrict
* Aging time: 60 мин.

```
configure terminal
interface FastEthernet0/6
 switchport port-security maximum 3
 switchport port-security violation restrict 
 switchport port-security aging time 60
 switchport port-security
```

```
show port-security interface f0/6
Port Security              : Enabled
Port Status                : Secure-up
Violation Mode             : Restrict
Aging Time                 : 60 mins
Aging Type                 : Absolute
SecureStatic Address Aging : Disabled
Maximum MAC Addresses      : 3
Total MAC Addresses        : 1
Configured MAC Addresses   : 0
Sticky MAC Addresses       : 0
Last Source Address:Vlan   : 0030.A3E8.1ADD:10
Security Violation Count   : 0
```

```
S1#show port-security address 
               Secure Mac Address Table
-----------------------------------------------------------------------------
Vlan    Mac Address       Type                          Ports   Remaining Age
                                                                   (mins)
----    -----------       ----                          -----   -------------
10	0030.A3E8.1ADD	DynamicConfigured	FastEthernet0/6		-
-----------------------------------------------------------------------------
Total Addresses in System (excluding one mac per port)     : 0
Max Addresses limit in System (excluding one mac per port) : 1024
```

Сконфигурировать безопасность DHCP Snooping

* На коммутаторе S2 включим DHCP snooping и настром DHCP snooping во VLAN 10.
* Настроим магистральные порты на коммутаторе S2 как доверенные порты.
* Ограничим ненадежный порт FastEthernet0/18 на коммутаторе S2 пятью DHCP-пакетами в секунду.
* Проверка DHCP Snooping на коммутаторе S2.

Пример включения DHCP snooping:

```
configure terminal
ip dhcp snooping
ip dhcp snooping vlan 10
!
interface FastEthernet 0/1
 ip dhcp snooping trust
!
interface FastEthernet 0/18
 ip dhcp snooping limit rate 5 
!
no ip dhcp snooping information option 
```

Пример настройки DHCP snooping:

```
S2#show ip dhcp snooping
Switch DHCP snooping is enabled
DHCP snooping is configured on following VLANs:
10
Insertion of option 82 is disabled
Option 82 on untrusted port is not allowed
Verification of hwaddr field is enabled
Interface                  Trusted    Rate limit (pps)
-----------------------    -------    ----------------
FastEthernet0/1            yes        unlimited       
FastEthernet0/18           no         5      
```

Обновим IP-адрес компьютера PC-B и проверим привязку отслеживания DHCP:

```
S2#sh ip dhcp snooping binding 
MacAddress          IpAddress        Lease(sec)  Type           VLAN  Interface
------------------  ---------------  ----------  -------------  ----  -----------------
00:0A:F3:7E:EE:2C   192.168.10.11    0           dhcp-snooping  10    FastEthernet0/18
Total number of bindings: 1
```

Сконфигурируем PortFast и BPDU Guard

* Настроим PortFast на всех портах доступа, которые используются на обоих коммутаторах.
* Включиим защиту BPDU на портах доступа VLAN 10 на коммутаторах S1 и S2, подключенных к компьютерам PC-A и PC-B.
* Проверим, что защита BPDU и PortFast включены на соответствующих портах.

Пример настройки на коммутаторе S1:

```

```