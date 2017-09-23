# Docker Symfony Starter (PHP7-FPM - NGINX - MySQL - PHPMyAdmin)


## Wstępna konfiguracja

1. Przed rozpoczęciem pracy nad nowym projektem przejżyj plik `.env`. Należy go dostosować do wymagań projektu.
Jako wymóg, należy zmienić parametr `APP_NAME` z domyślnego `szkolenie` na unikatowy. Np nazwa proejktu, bez spacji, same litery. Inaczej zduplikją się nazwy kontenerów jeśli użyjesz tego startera do innego projektu.

2. Domyślnie strona dostępna jest pod adresem http://szkolenie oraz https://szkolenie:81 (jeśli dodałeś https). Należy go także zmienić.
Lokalizacja pliku to `etc/nginx/symfony.conf` - parametr `server_name`. Pamiętaj, że występuje on w tym pliku dwa razy.
Możesz wpisać `localhost` wtedy, po odpaleniu dockera strony będą dostępne pod adresem http://localhost. Możesz wtedy pominąć następny krok.

3. Uaktualnij plik hosts (dodaj wpisany wcześniej adres)

    ```bash
    # UNIX only: get containers IP address and update host (replace IP according to your configuration) (on Windows, edit C:\Windows\System32\drivers\etc\hosts)
    $ sudo echo $(docker network inspect bridge | grep Gateway | grep -o -E '[0-9\.]+') "TWOJA_NAZWA_Z_SERVER_NAME" >> /etc/hosts
    ```

    **Note:** Dla **OS X** =>  [here](https://docs.docker.com/docker-for-mac/networking/) a dla **Windows** => [this](https://docs.docker.com/docker-for-windows/#/step-4-explore-the-application-and-run-examples) (4 krok).



4. Stwórz/uruchom kontenery za pomocą:

    ```bash
    $ docker-compose build
    $ docker-compose up
    ```
---

## Konfiguracja Nginx oraz SSL

1. Wygeneruj certyfikat SSL

    ```sh
    sudo docker run --rm -v $(pwd)/etc/ssl:/certificates -e "SERVER=szkolenie" jacoelho/generate-certificate
    ```
Pamiętaj, aby parametr `SERVER=szkolenie` odpowiadał nazwie twojego hosta z punktu 2

2. Aktualizacja Nginx

    Edit nginx file `etc/nginx/default.conf` and uncomment the SSL server section :

    ```sh
    # server {
    #     server_name localhost;
    #
    #     listen 443 ssl;
    #     ...
    # }
    ```

---

## Konfiguracja Xdebug

Jesli używasz innego IDE niż PHPStorem (np. Netbeans), [zapraszam tuaj](https://xdebug.org/docs/remote)

1. Sprawdź swoje IP :

    ```sh
    sudo ifconfig
    ```

2. Uaktualnij plik `etc/php/php.ini` i odkomentuj/zakomentuj potrzebny fragment.

3. Ustaw parameter `remote_host` na zgodny z twoim IP:

    ```sh
    xdebug.remote_host=192.168.0.1 # your IP
    ```

---

## Instalacja aplikacji

1. Domyślnie, strona musi znaleźć się w katalogu `web`. Możesz to zmienić w pliku `.env`, parametr `APP_PATH`.

2. Z katalogu głównego, gdzie jest docker odpal 

    ```bash
    $ git clone LINK_DO_REPO web
    ```

3. Prepare Symfony app
    1. Połącz się z konsolą za pomocą komendy:

        ```bash
        $ docker-compose exec php bash
        ```

    2. Popraw app/config/parameters.yml

        ```yml
        # path/to/your/symfony-project/app/config/parameters.yml
        parameters:
            database_host: database
        ```

    3. Nadaj odpowiednie uprawnienia do katalogów:

        ```bash
        $ chmod 077 var/logs* var/cache* var/sessions*
        ```

    3. Zainstaluj zależności:

        ```bash
        $ composer install
        # Symfony2
        $ sf doctrine:database:create
        $ sf doctrine:schema:update --force
        # Jeśli używasz`doctrine/doctrine-fixtures-bundle`
        $ sf doctrine:fixtures:load --no-interaction
        # Symfony3
        $ sf3 doctrine:database:create
        $ sf3 doctrine:schema:update --force
        # Jeśli używasz`doctrine/doctrine-fixtures-bundle`
        $ sf3 doctrine:fixtures:load --no-interaction
        ```
Gotowe ;-)


## Przydatne polecenia

```bash
# polecenia konsolowe
$ docker-compose exec php bash

# Composer
$ docker-compose exec php composer install

# Polecenia dla Symfony (Tip: Z automatu jest ustawiony alias dla sf2/sf3)
$ docker-compose exec php php /var/www/symfony/app/console cache:clear # Symfony2
$ docker-compose exec php php /var/www/symfony/bin/console cache:clear # Symfony3

# Przykładowe użycie aliasów
$ docker-compose exec php bash
$ sf cache:clear
$ sf3 c:c --no-warmup

# Sprawdzanie adresu IP (dla przykładu szkolenie_nginx)
$ docker inspect $(docker ps -f name=szkolenie_web -q) | grep IPAddress

# MySQL
$ docker-compose exec db mysql -uroot -p"root"

# Uprawnienia do cache i logów:
$ chmod -R 777 app/cache* app/logs* # Symfony2
$ chmod -R 777 var/cache* var/logs* var/sessions* # Symfony3

# Check CPU consumption
$ docker stats $(docker inspect -f "{{ .Name }}" $(docker ps -q))

# Usunięcie wszystkich kontenerów
$ docker rm $(docker ps -aq)

# Usunięcie wszystkich obrazów
$ docker rmi $(docker images -q)
```
