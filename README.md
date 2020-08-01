# Установка Schema Registry на DHP3.0.1

### Если ставить HDP 3.0.1, как советуют на Cloudera, то шаги следующие:

1. Качаем с Сloudera [HDP 3.0.1](https://www.cloudera.com/downloads/hortonworks-sandbox/hdp.html) под Docker 

2. Распаковываем его в папку

3. В файле docker-deploy-hdp30.sh в двух строчках $flavor == hdf и $flavor == hdp меняем '==' на '=' (может и не обязательно, но читал, что кому-то это помогло)

4. Запускаем
    ```bash
    sudo bash ./docker-deploy-hdp30.sh
    ```
    Появляется контейнер sandbox-hdp (используем bash, а не sh, потому, что sh не понимает квадратные скобки, которые есть в этом файле) 

5. Теперь выполняем 
    ```bash
    sudo bash ./assets/generate-proxy-deploy-script.sh
   ```

6. Выполняем 
    ```bash
   sudo bash ./sandbox/proxy/proxy-deploy.sh
   ```
   появился контейнер sandbox-proxy. 

    Этот sandbox-proxy, это nginx прокси-сервер, который запускается на определенном IP и перенаправляет запросы на HDP. Удобство в том, что когда надо будет открыть какой-то новый порт, убиваем контейнер sandbox-proxy, редактируем файл proxy-deploy.sh и запускаем его - появляется прокси с новыми портами. При этом HDP не трогаем. Однако неудобство в том, что он работает на определенном IP и надо делать туда маршрутизацию. Чтобы выяснить IP, заходим в контейнер:
     ```bash
   docker exec -it sandbox-hdp bash
   ```
   а потом 
   ```bash
   ifconfig
   ```
    или 
    ```bash
   cat /etc/hosts. Должно быть что-то типа 172.19.0.2.
    ```
   
### Поэтому мы пропустим все эти шаги, пробросим порты напрямую в HDP и не будем ставить прокси:
1. Качаем файл docker-deploy-hdp30.sh с этого репозитория и запускаем его (можно использовать sh, потому, что там уже нет квадратных скобок). Все, HDP стал со всеми нужными портами.

2. Заходим в контейнер:
    ```bash
    docker exec -it sandbox-hdp bash
    ```
    и делаем ресет админовского пароля Ambari: 
    ```bash
    ambari-admin-password-reset
    ``` 
	Главное не забыть новый пароль :) Дальше идем по инструкции, с некоторыми нюансами
3. Ставим mysql в контейнере: docker exec -it sandbox-hdp bash (если вы оттуда уже вышли), потом по инструкции от Сloudera:
	[install-mysql](https://docs.cloudera.com/HDPDocuments/HDF3/HDF-3.0.1.1/bk_installing-hdf-on-hdp/content/install-mysql.html).
	В пункте 4, если не получится сбросить пароль, то читаем [stackoverflow](https://stackoverflow.com/questions/41645309/mysql-error-access-denied-for-user-rootlocalhost)

4. Дальше по инструкции: [database-config-mysql](https://docs.cloudera.com/HDPDocuments/HDF3/HDF-3.0.1.1/bk_installing-hdf-on-hdp/content/database-config-mysql.html)

5. Читаем [документацию](https://docs.cloudera.com/HDPDocuments/HDF3/HDF-3.0.1.1/bk_installing-hdf-on-hdp/content/ch_install-mpack.html), но Management Pack качаем 
	версии 3.2 отсюда: [hdf_repository_locations](https://docs.cloudera.com/HDPDocuments/HDF3/HDF-3.2.0/release-notes/content/hdf_repository_locations.html) (для Linux/CentOS 7) 
	потому, что версия 3.0 не работает с Ambari 2.7, про это написано [здесь](https://community.cloudera.com/t5/Support-Questions/Need-help-installing-HDF-onto-HDP-3-0-HDF-Base-URL/td-p/187725). 
	Не забудьте Management Pack скопировать с локального компа в контейнер
	```bash
    sudo docker cp /tmp/hdf-ambari-mpack-3.2.0.0-520.tar.gz sandbox-hdp:/tmp/
    ```
	Если поставили не тот Management Pack, то удалять его вот так: [click me](https://community.cloudera.com/t5/Support-Questions/Remove-HDF-mpack/td-p/200140).
	
6. Пункт 5 в инструкции можно пропустить, потому, что там правильно прописаный HDF Base URL. Но если захотите поменять, то надо логиниться в Ambari под админом 

7. Добавляем нужный сервис и перезагружаем все сервисы, которые нужно.

 



