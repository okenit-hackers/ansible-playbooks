Программа ansible установлена на машине 192.168.5.130.
Желательно использовать для установки только Semaphore.
Пакеты с программами программа получает с репозитория 192.168.5.119:8000

## Ansible Semaphore

На **192.168.5.130:3000** установлен UI для Ansible - _**Semaphore**_.
Там можно смотреть состояние плейбуков и запускать их.
Редактирование плейбуков (ролей) необходимо выполнять в этом репозитории, семафор подключен к нему и получает все файлы отсюда.

**Как изменить хост установки?**

1. Заходим во вкладочку `Inventory`, открываем запись `altar_hosts` и ищем там нужный нам хост.   


2. Если его нет, то добавляем свой хост в формате:


 name ansible_host=ip ansible_user=user ansible_password=password ansible_sudo_pass=sudo_password

3. Переходим во вкладку `Environment` и ищем нужную нам программу.


4. Изменяем значение у поля `hosts` на нужный нам хост (так же можно указать группу).

пример:

 {
  "hosts": "my_host"
 }

меняем на:

 {
  "hosts": "name"
 }

5. Переходим в `Task Templates`, запускаем плейбук с нужной программой.


## ~~Ansible CLI~~ - не используется

В папке /home/smelov/install_programs лежат представленные выше плейбуки, которые используются для установки.
Ниже описаны команды для установки разных программ.

Для изменения машины, на которую необходимо поставить программу: изменить файл hosts.ini
(на opensearch_proxmox устанавливаются os+os-db+plugin)

На машинах proxmox обязательно должен стоять gateway 192.168.0.100, иначе к ним не присоединиться

### ~~Как устанавливать программы с помощью ansible~~ - не используется

#### 1. ПАУМР
1.1 `cd /home/smelov/install_programs`          
1.2 `ansible-playbook -i hosts.ini paumr/paumr_setup_proxmox.yml`   
1.3 Программа доступна по адресу 0.0.0.0:8000 (извне по ip машины)   
1.4 Для перезапуска программы:    
`python3 .local/lib/python3.7/site-packages/paumr/manage.py runserver 0.0.0.0:8000`   


#### 2. PSDOI-PARSING
2.1 `cd /home/smelov/install_programs`              
2.2 `ansible-playbook -i hosts.ini psdoi_parsing/psdoi_parsing_setup_proxmox.yml`      
2.3 Необходимо зайти на машину `192.168.0.101` (ID 101) и запустить программу командой `psdoi-parsing`      
(если занят 8080 порт, зайти в папку `~/.local/lib/python3.7/site-packages/psdoi_parsing`, в `config.cfg` поменять порт)      
2.4 По адресу 0.0.0.0:порт будет доступно API        
2.5 Создать источники по умолчанию: найти эндпоинт datasources/create_default открыть его, кликнуть Try it out и затем Execute        


#### 3. INDEXER
ВЕРСИЯ БЕЗ ЗАЩИТЫ И БЕЗ ПЛАГИНОВ        
3.1 `cd /home/smelov/install_programs`           
3.2 `ansible-playbook -i hosts.ini indexer/indexer.yml`              
3.3 Программа будет установлена в `/opt/indexer`         
3.4 Конфиг находится в папке `/opt/indexer/config`, файл `opensearch.yml`.           
3.5 Изначально он настроен так, что работает на localhost, для внешнего доступа нужно поменять `network.host`      
3.6 После смены конфига перезапустить сервис `sudo systemctl restart indexer`           


#### 4. DASHBOARD
ВЕРСИЯ БЕЗ ЗАЩИТЫ И БЕЗ ПЛАГИНОВ      
4.1 `cd /home/smelov/install_programs`           
4.2 `ansible-playbook -i hosts.ini dashboard/dashboard.yml`           
4.3 Программа будет установлена в `/opt/dashboard`              
4.4 Конфиг находится в папке `/opt/indexer/config`, файл `opensearch.yml`.           
4.5 Изначально он настроен так, что работает на localhost, для внешнего доступа нужно поменять `server.host`      
4.6 Так же если меняли `network.host` на indexer, нужно поменять `opensearch.hosts`           
4.7 После смены конфига перезапустить сервис `sudo systemctl restart dashboard`              


#### 5. PLUGIN
5.1 Установить dashboard                                
5.2 `cd /home/smelov/install_programs`                           
5.3 `ansible-playbook -i hosts.ini plugin/plugin.yml`                 
5.4 Ansible устанавливает плагин wazuh и перевод самого дашборда              
5.5 Чтобы подключить wazuh-manager к плагину нужно перейти в папку `/opt/dashboard/data/wazuh/config`, и в конфиге
`wazuh.yml` в конце исправить ip на нужный.                      
5.6 Перезапустить дашборд `sudo systemctl restart dashboard`                 


#### 6. ELASTALERT
6.1 `cd /home/smelov/install_programs`                            
6.2 `ansible-playbook -i hosts.ini elastalert/elastalert.yml`                      
6.3 Для запуска: `elastalert --config /path/to/config.cfg`                   


#### 7. INDEXER
ВЕРСИЯ БЕЗ ЗАЩИТЫ      
7.1 `cd /home/smelov/install_programs`              
7.2 `ansible-playbook -i hosts.ini indexer/indexer_plugins.yml`
7.3 Программа будет установлена в `/opt/indexer`      
7.4 Конфиг находится в папке `/opt/indexer/config`, файл `opensearch.yml`.         
7.5 Изначально он настроен так, что работает на localhost, для внешнего доступа нужно поменять `network.host`      
7.6 После смены конфига перезапустить сервис `sudo systemctl restart indexer`        



#### 8. DASHBOARD
ВЕРСИЯ БЕЗ ЗАЩИТЫ        
8.1 `cd /home/smelov/install_programs`          
8.2 `ansible-playbook -i hosts.ini dashboard/dashboard_plugins.yml`          
8.3 Программа будет установлена в `/opt/dashboard`         
8.4 Конфиг находится в папке `/opt/indexer/config`, файл `opensearch.yml`.       
8.5 Изначально он настроен так, что работает на localhost, для внешнего доступа нужно поменять server.host     
8.6 Так же если меняли `network.host` на indexer, нужно поменять `opensearch.hosts`    
8.7 После смены конфига перезапустить сервис `sudo systemctl restart dashboard`    


#### Можно установить опенсерч и дашборд с плагином командой:      
`ansible-playbook -i hosts.ini indexer/indexer.yml dashboard/dashboard.yml plugin/plugin.yml`      

#### 9.WAZUH
Есть большой набор плейбуков и прочего от авторов Вазуха для установки через энсибл. Я все это скачал нам. У меня получалось установить Вазух и Файлбит командой
ansible-playbook -i hosts.ini /home/smelov/wazuh-ansible/playbooks/wazuh-manager-oss.yml

