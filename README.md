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

drush cset system.site uuid "824769d3-1279-42ce-9bcb-6b813354c03c" -y
drush cset language.entity.hu uuid "2a63e705-5e72-41e3-82b9-a828f904c077" -y
drush cset shortcut.set.default uuid "5e7d1d6a-f20c-4daa-8524-beaf1a578270" -y


# tulajdonos, jogosultságok beállítása
chown -R www-data:www-data /var/www/html
chmod -R 700 /var/www/html

# egyedi blokkok helyének betöltése
composer require 'drupal/recreate_block_content'
drush en recreate_block_content -y

# konfiguráció importálása
# drush cim --source "/var/www/html/config"

# nyelvi fájlok frissítése
drush locale:update
```