### Домашнее задание №21 (OSPF)
1. Создан и запущен Vagrantfile с 3 роутерами (VM): router1, router2, router3;
2. Настроено окружение для корректной работы Ansible;
3. Написан плейбук, устанавливающий необходимое ПО на 3 ВМ;
4. Сформирован файл конфигурации [daemons.j2](https://github.com/uNkindy/Otus_Unit_21_OSPF/blob/main/templates/daemons.j2) для корекктной работы OSPF на ВМ;
5. Сформирован файл конфигурации [frr.conf](https://github.com/uNkindy/Otus_Unit_21_OSPF/blob/main/templates/frr.conf.j2), раскатывающий конфигурацию на 3 ВМ в зависимости от hostname виртуальной машины;
- изначально файл конфигурации роутеров сформирован на работу в режиме ассиметричного рутинга. Для того на router1 стоимость интерфейса enp0s8 установлена 1000. Проверим Ассиметричный роутинг, для этого пингуем 192.168.20.1, на интерфейсах router2 запустим tcpdump.
  
Интерфейс enp0s9:
```console
12:17:06.712558 IP 192.168.10.1 > router2: ICMP echo request, id 5, seq 7, length 64
12:17:07.713567 IP 192.168.10.1 > router2: ICMP echo request, id 5, seq 8, length 64
12:17:08.714363 IP 192.168.10.1 > router2: ICMP echo request, id 5, seq 9, length 64
12:17:09.716227 IP 192.168.10.1 > router2: ICMP echo request, id 5, seq 10, length 64
```
Интерфейс enp0s8:
```console
12:20:49.291706 IP router2 > 192.168.10.1: ICMP echo reply, id 5, seq 1, length 64
12:20:50.293960 IP router2 > 192.168.10.1: ICMP echo reply, id 5, seq 2, length 64
12:20:51.340651 IP router2 > 192.168.10.1: ICMP echo reply, id 5, seq 3, length 64
```
Видим, что асинхронный роутинг работает.
___
Для реализации синхронного роутинга добавим стоимость интерфейса enp0s8 для роутера router2 1000. Также будем пинговать 192.168.20.1, в tcpdump видим, что роутинг симметричный:
```console
root@router2:~# tcpdump -i enp0s9 icmp
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on enp0s9, link-type EN10MB (Ethernet), capture size 262144 bytes
12:34:21.558370 IP 192.168.10.1 > router2: ICMP echo request, id 7, seq 1, length 64
12:34:21.558453 IP router2 > 192.168.10.1: ICMP echo reply, id 7, seq 1, length 64
12:34:22.572822 IP 192.168.10.1 > router2: ICMP echo request, id 7, seq 2, length 64
12:34:22.572896 IP router2 > 192.168.10.1: ICMP echo reply, id 7, seq 2, length 64
12:34:23.712407 IP 192.168.10.1 > router2: ICMP echo request, id 7, seq 3, length 64
12:34:23.712441 IP router2 > 192.168.10.1: ICMP echo reply, id 7, seq 3, length 64
12:34:24.714213 IP 192.168.10.1 > router2: ICMP echo request, id 7, seq 4, length 64
12:34:24.714284 IP router2 > 192.168.10.1: ICMP echo reply, id 7, seq 4, length 64
```
Как видно запросы и ответы идут через тот же самый интерфейс.
___
В целях автоматизации процесса перехода от асинхронного роутинга к симметричному, в defaults добавлена переменная для активации симметричного роутинга symmetric_routing. Переменная выставлена по умолчанию в false. Для активации симметричного роутинга выставляем переменную true и запускаем плейбук c тегом serup_ospf:
```console
[root@devops Otus_Unit_21_OSPF]# ansible-playbook playbook.yml -t setup_ospf
```