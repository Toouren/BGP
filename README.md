# BGP Homework
##### Новиков Василий. КН-202. Вариант 18.

### Задание
Hа практике познакомиться с некоторыми возможностями протокола BGP, научиться
настраивать протокол в простейших случаях:
1) анонсировать маршруты внешним соседям eBGP;
2) настраивать перераспределение маршрутной информации OSPF – BGP.

### Решение

1. Запускаем OSPF в AS 100 (PC-1, SERVER-1, AS-1, ASBR).
    ```
    AS-1:
        Router(config)#router ospf 1 
        Router(config-router)#network 213.1.1.0 0.0.0.255 area 100
        Router(config-router)#network 214.1.1.0 0.0.0.255 area 100 
        Router(config-router)#network 215.1.1.0 0.0.0.255 area 100
    ASBR1:
    	Router(config)#router ospf 1 
        Router(config-router)#network 213.1.1.0 0.0.0.255 area 100
    ```

2. Проверяем пингом с ASBR на PC-1 и SERVER-1.
![N|Solid](https://i.imgur.com/lenbDDo.png)
![N|Solid](https://i.imgur.com/IGTSPGU.png)

3. Настраиваем ASBR1 и AS3 и запускаем на них BGP.
    ```
    ASBR1:
        Router(config)#router bgp 100 
        Router(config-router)#bgp router-id 1.1.1.1
        Router(config-router)#neighbor 101.0.0.1 remote-as 300
    AS3:
        Router(config)#router bgp 300 
        Router(config-router)#bgp router-id 3.3.3.3
        Router(config-router)#neighbor 101.0.0. remote-as 300
    ```
    Если на данном этапе проверить таблицу маршрутизации по протоколу BGP на роутере ASBR1, то в ней ничего не будет, т.к в AS 300 не входят никакие сети, однако, эти AS будут друг у друга отображаться, в качестве BGP-соседей. Если проверить таблицу маршрутизации на роутере AS3, то там тоже не будет маршрутов до сетей в AS 100, связано это с тем, что мы не анонсировали сети из AS 100 её соседям.

4. Для полной убедительности настроим на в AS 300 loopback’и, дав им адреса 101.0.28.1/32 и 101.0.29.1/32. Осталось анонсировать эти сети всем BGP-соседам, чтобы те, в свою очередь, знали о существовании этих сетей.
    ```
	AS3: 
		Router(config)#int loopback 0
		Router(config-if)#ip address 101.0.28.1 255.255.255.255
		
		Router(config)#int loopback 1
		Router(config-if)#ip address 101.0.29.1 255.255.255.255

		Router(config)#router bgp 300 
		Router(config-router)#network 101.0.28.0 mask 255.255.255.255
        Router(config-router)#network 101.0.29.0 mask 255.255.255.255 
    ```
5. Убедимся в том, что ASBR1 знает теперь о сетях своего соседа. (На скрине версия после выполнения последнего пункта всей практики. После выполнения этого пункта будет выводится выделенное)
![N|Solid](https://i.imgur.com/jNatXT0.png)

6. Аналогично настраиваем loopback’и на AS2, AS4. Запускаем BGP-процесс на AS2 и AS4 и устанавливаем связи между граничными маршрутизаторами AS. Анонсируем соответствующие адреса сетей BGP-соседям.

7. Убеждаемся, что анонсированные сети появились в таблице маршрутизации ASBR1. (Снова скрин с последней версией таблицы маршрутизации).
![N|Solid](https://i.imgur.com/J2zo4SN.png)
Можно заметить, что в колонке path появились пути до сетей, которые были анонсированы AS 100.

8. Настраиваем редистрибуцию маршрутных данных OSPF в данные BGP на маршрутизаторе ASBR1. Делается это при помощи команды ``redistribute ospf 1``. По умолчанию, если в команде redistribute не указан тип маршрута, то перераспределяются внутризональные и межзональные маршруты OSPF.
    ```
	ASBR1:
        Router(config)#router bgp 100 
        Router(config-router)#redistribute ospf 1
    ```
    
    Теперь AS 300, 418, 218 знают о сетях в AS 100 и могут построить до них маршрут. Убедимся в этом при помощи ``show ip bgp`` и ``show ip route``
AS3:
![N|Solid](https://i.imgur.com/TJS7j2i.png)
![N|Solid](https://i.imgur.com/I375qd4.png)
AS2:
![N|Solid](https://i.imgur.com/TXn93AA.png)
![N|Solid](https://i.imgur.com/qegZGLI.png)
AS4:
![N|Solid](https://i.imgur.com/iLzKgM9.png)
![N|Solid](https://i.imgur.com/n3Yh4Ue.png)

9. Выделяем сеть 214.1.1.0/24 (SERVER-1 -- AS1) в отдельную OSPF область.
    ```
    AS1:
        Router(config)#router ospf 1
        Router(config-router)#no network 214.1.1.0 0.0.0.255 area 100
        Router(config-router)#network 214.1.1.0 0.0.0.255 area 1
    ```
    При выделении сети в отдельную область, всё равно происходит её трансляция в BGP. На сколько я понимаю, связано это с тем, что транслировали мы весь OSPF процесс в BGP, а не отдельную область. Убедиться в этом можно при помощи просмотра таблиц маршрутизации, например, на роутере AS2.
    ![N|Solid](https://i.imgur.com/9aT8avC.png)
    ![N|Solid](https://i.imgur.com/1S4u7Il.png)
    
### Результат
Научился настраивать протокол BGP в простейших случаях:
1) анонсировать маршруты внешним соседям eBGP;
2) настраивать перераспределение маршрутной информации OSPF – BGP.

bgp_finish_sborka.pkt -- финальная сборка сети.