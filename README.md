### ALT
1. Выполните базовую настройку всех устройств:

	a. Соберите топологию согласно рисунку. Все устройства работают на OC Linux – ALT

· ISP - Альт Сервер 10.2 (CLI)

· CLI - Альт Рабочая станция 10.2 (GUI)

· HQ-R - Альт Сервер 10.2 (CLI)

· HQ-SRV - Альт Сервер 10.2 (GUI)

· BR-R - Альт Сервер 10.2 (CLI)

· BR-SRV - Альт Сервер 10.2 (CLI)

b. Присвоить имена в соответствии с топологией

c. Рассчитайте IP-адресацию IPv4. Необходимо заполнить таблицу №1. При необходимости отредактируйте таблицу.

d. Пул адресов для сети офиса BRANCH - не более 16. Для IPv6 пропустите этот пункт.

e. Пул адресов для сети офиса HQ - не более 64.

Таблица №1


![image](https://github.com/Julia666666666666666666/ALT/assets/148867585/0d007e45-b890-4145-bb39-ab9170d6b64d)

## Таблица сети 

|Имя устройства    | Интерфейсы| IPv4/IPv6   | Маска/Префикс|  	Шлюз      |
|------------------|-----------|-------------|--------------|-------------|
|ISP(net-5,3,4+кол)|ens192     |192.168.0.170| 	/30 	    |             |
|                  |ens224     |192.168.0.162| 	/30         |             |	
|                  |ens256     |192.168.0.166| 	/30 	    |             |
|                  |ens161     |10.12.16.10  |	/24         |10.12.16.254 |
|HQ-R(net-1,3)     |ens192     |192.168.0.2  |	/25 	    |             |
|                  |ens224     |192.168.0.161| 	/30 	    |192.168.0.162|
|BR-R(net-2,4)     |ens192     |192.168.0.130| 	/27 	    |             |
|                  |ens224     |192.168.0.165| 	/30 	    |192.168.0.166|
|HQ-SRV(net-1)     |ens192     |192.168.0.1  |	/25 	    |192.168.0.2  |
|BR-SRV(net-2)     |ens192     |192.168.0.129| 	/27 	    |192.168.0.130|
|CLI(net-5)        |ens192     |192.168.0.169| 	/30 	    |192.168.0.170|


## Выполнение задания 

Сначала просмотрим интерфейсы

```
ip a
```
Переходим если это необх. в режим супер поль.
```
su
```

Далее нужно открыть файл options для нужного интерфейса

```
vim /etc/net/ifaces/ens192/options
```
Там редактируем файл

![image](https://github.com/Julia666666666666666666/ALT/assets/148867585/f6f82dee-0af3-436c-9244-f713bae1ba07)


Сохраняем изменения

```
Esc: wq
```

Далее задаем нужный адрес на интр:

```
echo 192.168.0.170/30 > /etc/net/ifaces/ens192/ipv4address
```

Добавляем шлюз

```
echo default via x.x.x.x > /etc/net/ifaces/ens__/ipv4address
```

Настройка DNS-сервера

```
echo nameserver 8.8.8.8 > /etc/resolv.conf
```

Перезагружаем сеть

```
service network restart
```

Проверяем результат

```
ip a
```

Чтобы добавить интерфейс

```
mkdir /etc/net/ifaces/ens__
```

Далее нужно открыть файл options для нужного интерфейса

```
vim /etc/net/ifaces/ens__/options
```
Клавиша Insert чтобы редактировать


## NAT (ISP,HQ-R,BR-R)


У интерфейсов в папке options должны быть значения:
```
NM_CONTROLLED=no

DISABLED=no
```

Утанавливаем FIREWALLD

```
apt-get update
```
Далее

```
apt-get -y install firewalld
```

Вкл автозагрузку

```
systemctl enable --now firewalld
```

Добавл. правила к вход. пакетам

```
firewall-cmd --permanent --zone=public --add-interface=ens33

firewall-cmd --permanent --zone=trusted --add-interface=ens33
```

Включ. NAT

```
firewall-cmd --permanent --zone=public --add-masquerade
```

Сохр. правила

```
firewall-cmd --reload
```

Если не работает то 

```
systemctl disable network.service NetworkManager
```
## Настройка туннеля на HQ-R и BR-R

Создаем директорию
```
mkdir /etc/net/ifaces/tun1
```
Далее открываем файл options с использ. vim или nano

```
vim /etc/net/ifaces/tun1/options
```
В нем записываем следщ.

![image](https://github.com/Julia666666666666666666/ALT/assets/148867585/4bf042f7-d12f-4d79-8648-6cfba381f99a)


![image](https://github.com/Julia666666666666666666/ALT/assets/148867585/6b6da3b9-bddd-441e-a421-6391df0f8df3)

## OSPF(ISP, HQ-R, BR-R)

На устройстве ISP устанавливаем ffr:  
```
apt update 

apt install frr
```
После отк файл: 
```
nano /etc/frr/daemons
```
Меняем значения: 
```
ospfd=yes 
```
Перезагружаем: 
```
systemctl restart frr
```
Переходим в среду роутера:
```
vtysh
```
Далее настраиваем frr для ближ. устр:
```
conf t

router ospf

net  192.168.0.166/30 area 0

net 192.168.0.162/30 area 0

do sh ip ospf neighbor

ip forwarding

do wr

exit
```
После настраиваем FRR на HQ-R-1 & BR-R-2
```

1- 192.168.0.162/30

192.168.0.1/25

2-192.168.0.166/30

192.168.0.129/27
```
Проверяем настройки пингом:

HQ-SRV c BR-SRV

## Настройка DHCP(HQ-R)

Настройте автоматическое распределение IP-адресов на роутере HQ-R.
a. Учтите, что у сервера должен быть зарезервирован адрес.


Установливаем пакеты на HQ-R:

```
apt-get -y install dhcp-server
```
Переходим в файл:
```
/etc/sysconfig/dhcpd
```
И указываем интерфейс внутренней сети к (HQ-SRV):

```
DHCPDARGS=ens192
```

Копируем образец:

```
cp /etc/dhcp/dhcpd.conf.sample /etc/dhcp/dhcpd.conf
```
Далее переходим в файл:

```
/etc/dhcp/dhcpd.conf 
```
И пишем там:

![изображение](https://github.com/Julia666666666666666666/ALT/assets/148867585/f62d7344-b3b3-43f5-a450-568f8213d725)

После перезагружаем службу:

```
systemctl restart dhcpd
```
Проверяем статус службы
```
systemctl status dhcpd.service
```
Автозагрузка:
```
chkconfig dhcpd on

service dhcpd start
```
### Настройка на HQ-SRV (клиент):

Переходим в файл:
```
vim /etc/net/ifaces/ens18/ipv4address
```
Прописываем в файле:
```
#192.168.0.40
```
Далее открываем файл для нужного интерфейса:
```
nano /etc/net/ifaces/ens18/options
```
Меняем значения на
```
BOOTROTO=dhcp

NM_CONTROLLED=yes

DISABLED=no

CONFIG_IPV4=yes
```
Перезагружаем службу
```
systemctl reboot
```

Проверка:

![изображение](https://github.com/Julia666666666666666666/ALT/assets/148867585/89dac9cb-e20a-48c0-ae32-c2c198cbaaf3)

Адрес поменялся


## Настройка учетных записей

Настройте локальные учётные записи на всех устройствах в соответствии с таблицей.

![284669642-f65afe56-6b32-40ce-82be-d2ab53ac2717](https://github.com/Julia666666666666666666/ALT/assets/148867585/1e1230b7-4c91-4b40-afac-20d60fac56a6)


Добавляем пользователя(с маленькой буквы, без пробелов)
```
adduser admin
```
Далее
```
usermod -aG wheel admin
```
Задаем пароль
```
passwd (имя пользователя)

P@ssw0rd

P@ssw0rd
```
Чтобы просмотреть список созданных пользователей, нужно прописать команду:
```
nano /etc/passwd
```
Должно выйти

![изображение](https://github.com/Julia666666666666666666/ALT/assets/148867585/f547818d-24e1-4372-8eb6-914838d411b6)

## IPERF 3

Измерьте пропускную способность сети между двумя узлами HQ-R-ISP по средствам утилиты iperf 3. Предоставьте описание пропускной способности канала со скриншотами.

# Модуль 2

1. Установила виртуальную машину на ALT Linux (должна иметь доступ к интернету)

2. Установила Docker.

```
apt-get update && apt-get install -y docker-engine
```

```
systemctl enable --now docker
```

3. Запустила контейнер «Hello World»
```
docker pull hello-world
```
4. Смотрим список контейнеров:

```
docker images
```
5. Запускаем контейнер Hello-World
   ```
   docker run hello-world
   ```
6.Удаляем контейнер
```
docker image rm -f hello-world
```


   
