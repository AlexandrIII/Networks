## Настройка маршрутизации между VLAN с использованием "роутера на палочке"
###  Задание:

  1. Создание сети и настройка основных параметров устройств
  2. Создание VLAN и назначение портов коммутаторов, настройка магистрали 802.1 Q между коммутаторами;
  3. Настройка маршрутизации между VLAN на маршрутизаторе;
  4. Проверка маршрутизации между VLAN.
###  Решение:

#### 1 Создание сети и настройка основных параметров устройств
##### 1.1 Топология сети
 

Cхема лабораторного стенда, выполненная в eve-ng:

![](lab_1_eve.png)

##### 1.2 Настройка основных параметров устройств
#####  Пример настройки на базовых параметров на маршрутизаторе R1:
```
hostname R1
!
  boot-start-marker
  boot-end-marker
!
!
  enable secret 5 $1$k/XQ$C1KMVybBEmx5284xWK4gQ1
!
  aaa new-model
!
!
  aaa authentication login default local
!
!
  aaa session-id common
  ethernet lmi ce
!
!
  clock timezone MSK 3 0
  mmi polling-interval 60
  no mmi auto-configure
  no mmi pvc
  mmi snmp-timeout 180
!
!
  no ip domain lookup
  ip cef
  no ipv6 cef
!
  multilink bundle-name authenticated
!
!
  username cisco privilege 15 secret 9 $9$Bp5GvMPutUfh2I$wsctJDWwH8/BLOC6juhB2Jc45qtYJ/gWy.JwLZ4zii.
!
  redundancy
  ip forward-protocol nd
!
!
  no ip http server
  no ip http secure-server
!
!
!
!
!
!
  control-plane
!
  banner exec ^CAnyone accessing the device that unauthorized access is prohibited^C
  banner incoming ^CAnyone accessing the device that unauthorized access is prohibited^C
  banner login ^CAnyone accessing the device that unauthorized access is prohibited^C
!
  line con 0
  line aux 0
  line vty 0 4
    transport input none
!
  no scheduler allocate
!
  end
```
Аналагично настраиваем коммутаторы S1 и S2
#### 2 Создание VLAN
##### 2.1 Коммутатор S1
```
!
interface GigabitEthernet0/0
 switchport trunk encapsulation dot1q
 switchport trunk native vlan 8
 switchport mode trunk
 switchport nonegotiate
 media-type rj45
 negotiation auto
!
interface GigabitEthernet0/1
 switchport trunk encapsulation dot1q
 switchport trunk native vlan 8
 switchport mode trunk
 switchport nonegotiate
 media-type rj45
 negotiation auto
!
interface GigabitEthernet0/2
 switchport access vlan 7
 media-type rj45
 negotiation auto
!
interface GigabitEthernet0/3
 switchport access vlan 3
 switchport mode access
 media-type rj45
 negotiation auto
!
interface GigabitEthernet1/0
 switchport access vlan 7
 switchport mode access
 media-type rj45
 negotiation auto
!
interface GigabitEthernet1/1
 switchport access vlan 7
 switchport mode access
 media-type rj45
 negotiation auto
!
interface GigabitEthernet1/2
 switchport access vlan 7
 switchport mode access
 media-type rj45
 negotiation auto
!
interface GigabitEthernet1/3
 switchport access vlan 7
 switchport mode access
 media-type rj45
 negotiation auto
```
Проверим созданные VLAN 
```
S1#sh vlan brief

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active
3    Management                       active    Gi0/3
4    Operations                       active
7    ParkingLot                       active    Gi0/2, Gi1/0, Gi1/1, Gi1/2
                                                Gi1/3
1002 fddi-default                     act/unsup
1003 token-ring-default               act/unsup
1004 fddinet-default                  act/unsup
1005 trnet-default                    act/unsup
```
Проверим созданные транковые порты:
```
S1#show interfaces trunk

Port        Mode             Encapsulation  Status        Native vlan
Gi0/0       on               802.1q         trunking      8
Gi0/1       on               802.1q         trunking      8

Port        Vlans allowed on trunk
Gi0/0       1-4094
Gi0/1       1-4094

Port        Vlans allowed and active in management domain
Gi0/0       1,3-4,7
Gi0/1       1,3-4,7

Port        Vlans in spanning tree forwarding state and not pruned
Gi0/0       1,3-4,7
Gi0/1       1,3-4,7
```
##### 2.1 Коммутатор S2
```
!
interface GigabitEthernet0/0
 switchport access vlan 7
 switchport mode access
 media-type rj45
 negotiation auto
!
interface GigabitEthernet0/1
 switchport trunk encapsulation dot1q
 switchport trunk native vlan 8
 switchport mode trunk
 switchport nonegotiate
 media-type rj45
 negotiation auto
!
interface GigabitEthernet0/2
 switchport access vlan 7
 switchport mode access
 media-type rj45
 negotiation auto
!
interface GigabitEthernet0/3
 switchport access vlan 4
 switchport mode access
 media-type rj45
 negotiation auto
!
interface GigabitEthernet1/0
 switchport access vlan 7
 switchport mode access
 media-type rj45
 negotiation auto
!
interface GigabitEthernet1/1
 switchport access vlan 7
 switchport mode access
 media-type rj45
 negotiation auto
!
interface GigabitEthernet1/2
 switchport access vlan 7
 switchport mode access
 media-type rj45
 negotiation auto
!
interface GigabitEthernet1/3
 switchport access vlan 7
 switchport mode access
 media-type rj45
 negotiation auto
```
Проверим созданные VLAN 

S2#show vlan brief
```
VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active
3    Management                       active
4    Operations                       active    Gi0/3
7    ParkingLot                       active    Gi0/0, Gi0/2, Gi1/0, Gi1/1
                                                Gi1/2, Gi1/3
1002 fddi-default                     act/unsup
1003 token-ring-default               act/unsup
1004 fddinet-default                  act/unsup
1005 trnet-default                    act/unsup
```
Проверим созданные транковые порты:
```
S2#show interfaces trunk

Port        Mode             Encapsulation  Status        Native vlan
Gi0/1       on               802.1q         trunking      8

Port        Vlans allowed on trunk
Gi0/1       1-4094

Port        Vlans allowed and active in management domain
Gi0/1       1,3-4,7

Port        Vlans in spanning tree forwarding state and not pruned
Gi0/1       1,3-4,7
```
#### 3 Настройка маршрутизации
На маршрутизаторе создадим sub-интерфейсы для каждого VLAN.
```
interface GigabitEthernet0/0
 no ip address
 duplex auto
 speed auto
 media-type rj45
!
interface GigabitEthernet0/0.3
 encapsulation dot1Q 3
 ip address 192.168.3.1 255.255.255.0
!
interface GigabitEthernet0/0.4
 encapsulation dot1Q 4
 ip address 192.168.4.1 255.255.255.0
!
interface GigabitEthernet0/0.7
 encapsulation dot1Q 7
 ip address 192.168.7.1 255.255.255.0
```
Проверим созданныеи субинтерфейсы:
```
R1#sh ip interface brief
Interface                  IP-Address      OK? Method Status                Protocol
GigabitEthernet0/0         unassigned      YES NVRAM  up                    up
GigabitEthernet0/0.3       192.168.3.1     YES NVRAM  up                    up
GigabitEthernet0/0.4       192.168.4.1     YES NVRAM  up                    up
GigabitEthernet0/0.7       192.168.7.1     YES manual up                    up
```
#### 4. Проверка маршрутизации между VLAN.
Проверка работы маршрутизации с PC_A

![](Ping_PC_A.png)

Проверка работы маршрутизации с PC_B

![](Ping_PC_B.png)


Все файлы изменений приведены [здесь](conf/).






