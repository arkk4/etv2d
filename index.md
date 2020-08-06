# E-TRANSv2 Документация

Вся документация которая относится к проекту E-TRANSv2 (Edge-Translator)

## Полезные ссылки

* `https://github.com/arkk4/E-TRANSv2` - Гитхаб проекта
* `https://pyneng.readthedocs.io/ru/latest/` - Книга по пайтону
* `https://click.palletsprojects.com/en/7.x/` - Документация по click
* `http://vrds.arkk4.online:8443/` - CodeServer

## Структура проекта

    ETClier.py    # Модуль CLI Интерфейса, используется click
    ETDownloader.py # Модуль отвечающий за скачивание файла конфигурации
        ETConfpars.py  # Модуль парсинга необходимых данных из исходного конфига и складывания их в {swip}.yaml
        ETTranslator.py       # Модуль подстановки необходимых данных из {swip}.yaml в темлейт конфига (темлпейт определяется в зависимости от модели выбранной пользователем при взаимодействии с ETClier.py) и сохранение полученого файла как {swip}_ET.cfg
    ETComander.py # Модуль отвечающий за инициализацию com соединения с настраиваемым коммутатором и заливкой конфигурации {swip}_ET.cfg в него посредством работы с буффером обмена 

### Текстовое описание работы

* Пользователь вызывает скрипт с указанием флагов (опционально) и указанием ip адреса свича с    которого берется конфиг и модели настраиваемого свича (обязательно), на этом этапе работает модуль  ETClier
* IP переданый пользователем в ETClier передается в следующий модуль - ETDownloader, который скачивает файл конфигурации свича с указаным адресом название полученного файла - {IP СВИЧА}.cfg
* Скачаный файл конфигурации передается модулю ETConfpars который извлекает из этого файла необходимые данные и складывает их в словарь с названием {IP СВИЧА}.yaml
* Далее работает модуль ETTranslator которому от модуля ETClier передается переменная содержащая в себе модель свича, на основании этого, модуль подгружает текстовый файл {МОДЕЛЬ СВИЧА} который представляет из себя заготовку для будущей готовой конфигурации. Далее с помощью Jinja2 файлы {МОДЕЛЬ СВИЧА} и {IP СВИЧА}.yaml объединяются, полученый в итоге файл называется {IP СВИЧА}_ET.cfg
* Данные из полученого на прошлом этапе файла передаются в буфер обмена и модуль ETComander выполняет вставку содержимого файла в интерфейс коммутатора через com соединение
## Пример шаблона конфигурации

В примере используется конфигурация для модели ECS4100-28T

```!<stackingDB>00</stackingDB>
!<stackingMac>01_80-a2-35-9f-ac-79_03</stackingMac>
!
!
!
!---<InitPhaseConfig>
!---</InitPhaseConfig>
!
!
!
prompt {NAGIOS_LOCATION}
hostname 380072-8CEA1BFBB7301
!
!
!
!
!
!
!
!
!
!
snmp-server community public ro
snmp-server community private rw
!
!
!
enable password 7 1b3231655cebb7a1f783eddf27d254ca
!
!
logging history flash 5
!
!
!
!
vlan database
 VLAN {VID} name {VNAME} media ethernet
!
!
!
!
!
!
!
!
!
!
!
!
!
!
!
!
!
!
ip igmp filter
!
!
!
mvr profile 1 239.0.0.1 239.0.0.255
mvr domain 1
mvr domain 1 vlan {MCAST_VID}
mvr domain 1 associated-profile 1
!
!
!
!
no lldp
!
!
!
!
!
!
!
!
!
!
!
ip arp inspection vlan {A_VID}
ip arp inspection validate src-mac
!
!
no loopback-detection
loopback-detection action none
!
interface ethernet 1/{IFACE}
 ip igmp max-groups 10
 ip igmp max-groups action replace
 description {DESC}
 {IP_ARP_TR}
 {IP_ARP_LR}
 switchport allowed vlan add {A_VID} untagged
 switchport native vlan {A_VID}
 switchport allowed vlan remove 1
 spanning-tree spanning-disabled
 mvr domain 1 type {MVR_TYPE}
!
interface ethernet 1/{IFACE}
 {IP_ARP_TR}
 {IP_ARP_LR}
 switchport mode {SWPORT_MODE}
 {SWPORT_ACC_F-TYPES}}
 switchport allowed vlan add {T_VID} {F-TYPES}
 spanning-tree spanning-disabled
 mvr domain 1 type source
 {IP_DHCP_SNP}
!
!
!
!
!
!
interface vlan {MGMT_VID}
 ip address 10.1.{BRANCH}.{SWIP} 255.255.255.0
!
!
!
!
!
management snmp-client 10.1.0.1 10.1.0.254
management snmp-client 10.1.{BRANCH}.254 10.1.{BRANCH}.254
management telnet-client 10.1.0.1 10.1.0.254
management telnet-client 10.1.{BRANCH}.254 10.1.{BRANCH}.254
!
!
!
!
!
!
!
!
!
no ip http server
!
!
!
ip route 0.0.0.0 0.0.0.0 10.1.{BRANCH}.254
!
!
!
!
!
!
!
!
!
!
!
!
!
!
!
!
!
!
!
!
!
!
ip dhcp snooping
ip dhcp snooping vlan {A_VID}
ip dhcp snooping information option
ip dhcp snooping information option remote-id ip-address encode hex
!
!
!
!
!
!
!
!
!
!
!
!
!
!
!
interface vlan 1
!
interface vlan {MGMT_VID}
!
!
!
!
!
!
!
!
!
line console
!
!
line vty
!
!
!
end
!```

