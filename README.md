# Reverse proxy docker et nginx

## Ce qu'il faut :

- installation de docker
- installation de docker-compose

## Configuration du serveur

- Aller dans le dossier `/opt/`
- Y créé deux dossiers `mkdir sites && mkdir proxy`
- Dans le dossier proxy y cloner le projet
- Puis déplacer le dossier `data` et le `docker-compose.yml` avec la commande `mv data ../data && mv docker-compose.yml ../docker-compose.yml`
- Lancer les containeurs docker `docker-compose up -d`
- Dans le dossier site y mettre ces projets et les sites à mettre en ligne

## Mettre un site en ligne 

### Pour laravel && Symfony

a) Mettre le projet en place

- clone le projet dans le dossier `/opt/sites`
- Mettre en place le `.env`, `cp .env.prod.exemple .env`
- Mettre en place le `docker-compose.yml`, `cp docker-compose.prod.yml docker-compose.yml`
- Lancer les containeurs docker `docker-compose.yml`

b) Configurer le proxy

Pour configurer la configuration nginx il faut aller dans le dossier `/proxy/data/nginx/conf.d`

- Copier Coller le `exemple.conf`, `cp exemple.conf mon_site.conf`

````nginx configuration pro

upstream name_my_projet {
        server container_name_nginx:80;
}

server {
        listen 80;
        listen [::]:80;

        server_name url_site.fr;

        location /.well-known/acme-challenge/ {
                root /var/www/certbot;
        }
	
	location / {
		return 301 https://$host$request_uri;
	}
}

#server {
#        listen 443 ssl;
#        listen [::]:443 ssl;
#
#        server_name url_site.fr;
#
#        location / {
#                proxy_set_header        Host $host:$server_port;
#                proxy_set_header        X-Real-IP $remote_addr;
#                proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
#                proxy_set_header        X-Forwarded-Proto $scheme;
#                proxy_redirect          off;
#                proxy_pass              http://name_my_projet;
#        }
#
#        ssl_certificate /etc/letsencrypt/live/url_site.fr/fullchain.pem;
#        ssl_certificate_key /etc/letsencrypt/live/url_site.fr/privkey.pem;
#}
````

- Relancer les containers du proxy `docker-compose restart`

Comme nous pouvons remarquer nous avons la partie ssl en commentaire dans notre configration nginx, c'est normal, si cela n'est pas commenter nous pouvons pas up les container du proxy 

- Création du certificat SSL 

Il nous faut aller dans le container de `certbot`, `docker-compose exec certbot ash`, danns le container nous allons lancer la commande suivante 

````shell
certbot certonly
````

Dans un premier temps il va nous demande notre mode de fonctionnement et nous allons choisir la deuxième possibilité, donc entré `2`.

Ensuite il va nous demande notre url de notre site `my_url.fr`

Et va nous demande ou on veux mettre le certificat et on va le mettre dans `/var/www/certbot`

- Maintenant vous pouvez décommenter la partie SSL de votre configuration nginx
- Egalement vous pouvez redémarrer les container du proxy `docker-compose restart`

Votre site est bien en `HTTPS` maintenant !!