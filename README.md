# Docker image with Apache and PHP 5.3.10 running on Ubuntu 12.04
Docker image with Apache optimized to run Drupal 6 websites. Uses PHP 5.3 because Drupal 6 has some issues with newer versions of PHP.

Includes (Composer, Drush, PHP uploadprogress, APC)

Docker-compose starts nginx reverse proxy server from "jwilder/nginx-proxy" image together with letsencrypt companion for https support. 

### Build and usage with docker-compose

below docker-compose.yml should be placed in /home on host system
Docker file and config folder goes to /home/drupal_build

The website will be loaded from /var/www mapped to /home/www on a host system.


#### docker-compose

		version: "3"
		services:
		  nginx-proxy:
		   image: jwilder/nginx-proxy
		   restart: unless-stopped
		   container_name: nginx-proxy
		   labels: 
			 com.github.jrcs.letsencrypt_nginx_proxy_companion.nginx_proxy: "true"
		   ports:
			 - "80:80"
			 - "443:443"
		   volumes:
			 - /var/run/docker.sock:/tmp/docker.sock:ro
			 - /home/nginx_proxy/certs:/etc/nginx/certs
			 - /home/nginx_proxy/cert_challange/vhost.d:/etc/nginx/vhost.d
			 - /home/nginx_proxy/cert_challange/html:/usr/share/nginx/html
			 - /home/nginx_logs/:/var/log/nginx/
			 - /home/nginx_conf/nginx.conf:/etc/nginx/nginx.conf
			 #- /home/nginx_conf/blockips.conf:/etc/nginx/blockips.conf
			 
		  web:
			build: ./drupal_build
			labels:
			  - com.example
			depends_on:
			  - db
			restart: unless-stopped
			container_name: web
			ports:
			 - "8080:80"
			volumes:
			  - /home/www/:/var/www
			  - /home/log:/var/log/supervisor
			environment:
			- VIRTUAL_HOST=example.com, www.example.com
			- LETSENCRYPT_HOST=example.com, www.example.com
			  
		  db:
			image: mysql:5.7
			restart: always
			container_name: db
			command: --max_allowed_packet=104857600
			environment:
			  MYSQL_DATABASE: 'db'
			  # so you don't have to use root, but you can if you like
			  MYSQL_USER: 'db_user'
			  # you can use whatever password you like
			  MYSQL_PASSWORD: 'password'
			  # password for root access
			  MYSQL_ROOT_PASSWORD: 'root_password'
			ports:
			  # <port exposed> : < mysql port running inside container>
			  - '3306:3306'
			# expose:
			  # # opens port 3306 on the container
			  # - '3306'
			  # where our data will be persisted
			volumes:
			  - /home/db_files:/var/lib/mysql
			  
		  letsencypt:
		   image: jrcs/letsencrypt-nginx-proxy-companion
		   restart: unless-stopped
		   container_name: letsencypt
		   depends_on:
			  - "nginx-proxy"
		   volumes:
			 - /var/run/docker.sock:/var/run/docker.sock:ro
			 - /home/nginx_proxy/certs:/etc/nginx/certs
			 - /home/nginx_proxy/cert_challange/vhost.d:/etc/nginx/vhost.d
			 - /home/nginx_proxy/cert_challange/html:/usr/share/nginx/html
