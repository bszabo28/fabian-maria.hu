# Fábián Mária weboldala

## Előkészületek, docker környezet alaphelyzetbe állítása

Docker fejlesztői környezetben csak egy projektet szoktam megtartani, emiatt indulásképp törlöm a konténereket, az image-ket, volumes bejegyzéseket, majd a cache-t is.

A fentieket én a docker felületéről klikkeléssel szoktam törölni, illetve kiadom az alábbi parancsot:
```sh
docker system prune -a
```

[Itt találsz bővebb leírást a törlés menetéről parancssorban](https://www.digitalocean.com/community/tutorials/how-to-remove-docker-images-containers-and-volumes)


## Projekt telepítése és futtatása helyi környezetben
### Friss Drupal telepítése
A composer.* fájlok elkészítéséhez a módosított docker compose fájlt használom. (docker-compose.fresh.yml).
```sh
docker compose -f docker-compose.fresh.yml build
docker compose -f docker-compose.fresh.yml up -d

mkdir drupal-dev
mkdir drupal-dev/private
mkdir drupal-dev/config
mkdir drupal-dev/web
mkdir  drupal-dev/web/themes
mkdir drupal-dev/web/modules
mkdir drupal-dev/web/themes/custom
mkdir drupal-dev/web/modules/custom
docker cp fabianmaria_php:/var/www/html/composer.json ./drupal-dev/
docker cp fabianmaria_php:/var/www/html/composer.lock ./drupal-dev/
```

Ha ez megvan, composer json és lock létrejött, akkor újra alap helyzetbe állítom a docker környezetet

#### Docker környezet felépítése
```sh
docker compose build
docker compose up -d
```

### Drush telepítése
(Automatikusan telepítésre kerül)
```sh
# belépés a php konténerbe
docker exec -it fabianmaria_php /bin/bash

composer require drush/drush --no-interaction
echo "alias drush='/var/www/html/vendor/bin/drush'" >> ~/.bashrc
. ~/.bashrc
```
### Hosts fájl bejegyzése
- Windows alatt: C:\Windows\System32\drivers\etc\hosts
- Osx, linux: /etc/hosts
```sh
127.0.0.1 dev.fabianmaria.local
```

### Drupal telepítése
```sh
# belépés a php konténerbe
docker exec -it fabianmaria_php /bin/bash

# site telepítése a fejlesztői környezetben
drush site:install standard --db-url=mysql://$MYSQL_USER:$MYSQL_PASSWORD@mysql:3306/$MYSQL_DATABASE --site-name=$LOCAL_HOST_NAME --account-name=admin --account-pass=abc123 --locale=hu -y

drush cset system.site uuid "da84729f-617e-4b9b-9c11-9b2658a505dd" -y
drush cset language.entity.hu uuid "57984991-b07e-4828-bc1c-9f30deed9b45" -y

# tulajdonos, jogosultságok beállítása
chown -R www-data:www-data /var/www/html
chmod -R 700 /var/www/html

# egyedi blokkok helyének betöltése
composer require 'drupal/recreate_block_content:^3.0'
drush en recreate_block_content -y

# Drupal jelenlegi verziójában van egy hiba, ami miatt felül kell írni az alábbi fájl tartalmát
# drupal-dev/config/shortcut.set.default.yml
# https://dev.fabianmaria.local/admin/config/development/configuration/single/export
# Gyorshivatkozás csoport, Alapértelmezés

# konfiguráció importálása
drush cim --source "/var/www/html/config"

# nyelvi fájlok frissítése
drush locale:update
```