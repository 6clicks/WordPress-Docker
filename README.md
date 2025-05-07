# 🚀 Docker WordPress Template

Un template Docker Compose pour lancer **autant d’instances WordPress** que tu veux, sans collision de ports, de volumes ni de bases de données !

---

## 📂 Structure du projet

```
mon-wp-template/
├── docker-compose.yml
├── .env
└── data/               # Contiendra automatiquement les dossiers par projet
    ├── mon_wp1/
    │   ├── db/         # Données MySQL
    │   └── wordpress/  # Fichiers WordPress (code + uploads)
    └── mon_wp2/        # Exemple pour deuxième instance
        ├── db/
        └── wordpress/
```

---

## 🔧 Prérequis

* Docker ≥ 20.10
* Docker Compose ≥ 1.29
* (Optionnel) Docker Desktop pour Windows/Mac
* Connaissances basiques de ligne de commande

---

## 📝 Fichier de configuration : `.env`

Place un fichier `.env` à la racine (même dossier que `docker-compose.yml`).
Voici un exemple à copier/adapter pour chaque projet :

```dotenv
# Nom du projet (use tes propres tags pour Docker Compose)
COMPOSE_PROJECT_NAME=mon_wp1

# Dossier de stockage (relatif au dossier du compose)
PROJECT_DIR=./data/mon_wp1

# Ports exposés
WP_PORT=8010
PMA_PORT=8012

# Identifiants BD
DB_NAME=wp1_db
DB_USER=wp1_user
DB_PASSWORD=SuperSecret123

# (Optionnel) Versions images
WP_IMAGE=automattic/wordpress-xdebug
DB_IMAGE=mysql:8
```

> 💡 **Astuce** : Pour lancer sans dupliquer de dossier, tu peux aussi
>
> ```bash
> COMPOSE_PROJECT_NAME=mon_wp2 PROJECT_DIR=./data/mon_wp2 WP_PORT=8020 PMA_PORT=8022 \
> DB_NAME=wp2_db DB_USER=wp2_user DB_PASSWORD=UltraSecret \
> docker-compose up -d
> ```

---

## 🐳 `docker-compose.yml`

Le cœur du template :

```yaml
version: '3.9'

services:
  mysql:
    image: ${DB_IMAGE}
    volumes:
      - db_data:/var/lib/mysql
    restart: always
    environment:
      MYSQL_ALLOW_EMPTY_PASSWORD: 'yes'
      MYSQL_DATABASE: ${DB_NAME}
      MYSQL_USER: ${DB_USER}
      MYSQL_PASSWORD: ${DB_PASSWORD}

  wordpress:
    image: ${WP_IMAGE}
    depends_on:
      - mysql
    ports:
      - "${WP_PORT}:80"
    volumes:
      - wordpress_data:/var/www/html
    restart: always
    environment:
      WORDPRESS_DB_HOST: mysql
      WORDPRESS_DB_USER: ${DB_USER}
      WORDPRESS_DB_PASSWORD: ${DB_PASSWORD}
      WORDPRESS_DB_NAME: ${DB_NAME}

  phpmyadmin:
    image: phpmyadmin/phpmyadmin:latest
    depends_on:
      - mysql
    ports:
      - "${PMA_PORT}:80"
    restart: always
    environment:
      PMA_HOST: mysql
      PMA_PORT: 3306
      PMA_ARBITRARY: '1'

volumes:
  db_data: {}
  wordpress_data: {}
```

> Ici, Docker Compose **préfixe** les volumes (`db_data` → `mon_wp1_db_data`) grâce à `COMPOSE_PROJECT_NAME`.

---

## 🚀 Lancer une instance

1. Copier le dossier `mon-wp-template/`
2. Créer/adapter ton `.env`
3. Dans ton terminal :

   ```bash
   docker-compose up -d
   ```
4. Accède à WordPress sur `http://localhost:WP_PORT` (ex. [http://localhost:8010](http://localhost:8010)) et phpMyAdmin sur `http://localhost:PMA_PORT`.

---

## 🛑 Arrêt et nettoyage

* **Stop**

  ```bash
  docker-compose down
  ```
* **Supprimer volumes + données**

  ```bash
  docker-compose down -v
  ```

---

## ⚙️ Customisation avancée

* **Bind mount vers disque réseau (Windows U:)**

  * Autoriser le partage de `U:` dans Docker Desktop > Resources > File Sharing
  * Exemple dans `docker-compose.yml` :

    ```yaml
    wordpress:
      volumes:
        - "U:/mes_wp/mon_wp1:/var/www/html"
    ```
* **Ajouter Xdebug, Redis, un reverse-proxy…**

  * Surcharge `WP_IMAGE` dans `.env` ou étends le service
  * Exemple :

    ```yaml
    wordpress:
      image: my-custom-wp:latest
      build: ./docker/wp-custom
      # …
    ```
* **Réseau personnalisé**

  ```yaml
  networks:
    wp-net:
      driver: bridge

  services:
    mysql:
      networks: [wp-net]
    wordpress:
      networks: [wp-net]
    phpmyadmin:
      networks: [wp-net]
  ```

---

## ❓ FAQ rapide

* **Pourquoi `db_data` ?**
  Pour **persister** ta BDD au-delà du conteneur. Sans volume, tu perds tout à chaque `down`/`up`.
* **Comment lancer 5 WP en même temps ?**
  Duplique simplement ton dossier, ajuste `.env` (ports + répertoire), puis `docker-compose up -d` dans chacun.
* **Performance lente sur réseau ?**
  Les bind mounts CIFS/SMB sont plus lents : ok en dev, à proscrire en prod ou tests lourds.

---

## 📖 Licence & Crédits

MIT © \[Ton Entreprise] – Template inspiré par [automattic/wordpress-docker](https://github.com/automattic/wordpress-docker) et ta dose de café ☕

---

> Prêt à conteneuriser ? Let’s Docker! 🐳✨
