Deploy Laravel + Elasticsearch on Docker( Ubuntu )
==================================================

* ***Actions on the deployment of the project:***

- Making a new project elastic_docker.loc:
																		
	sudo chmod -R 777 /var/www/LARAVEL/Elasticsearch/elastic_docker.loc

	//!!!! .conf
	sudo cp /etc/apache2/sites-available/test.loc.conf /etc/apache2/sites-available/elastic_docker.loc.conf
			
	sudo nano /etc/apache2/sites-available/elastic_docker.loc.conf

	sudo a2ensite elastic_docker.loc.conf

	sudo systemctl restart apache2

	sudo nano /etc/hosts

	cd /var/www/LARAVEL/Elasticsearch/elastic_docker.loc/

---

Deploy on Docker( Ubuntu ).
===========================

For Deploy on Docker you must have the structure folders and files.

SCHEMA:

/var/www/LARAVEL/Elasticsearch/elastic_docker.loc/
												  |web/
												  |    Dockerfile     - file for create an image with php7.2-apache version
												  |.env               - file with paths environment variables
												  |docker-compose.yml - file with managing images services and their settings
												  |databases/         - folder that consist database locally
 												  |elastic_amitavroy/ - folder with a project which appeared after cloning from github

---

* ***Dockerfile***

```
FROM php:7.2-apache

RUN docker-php-ext-install \
     pdo_mysql \
     && a2enmod \
     rewrite
```

* ***.env***

```
#PATHS=

DB_PATH_HOST=./databases

APP_PATH_HOST=./elastic_amitavroy

APP_PATH_CONTAINER=/var/www/html/

APP_NAME=Docker_Laravel_Es
```

* ***docker-compose.yml - Elasticsearch version: `"image: elasticsearch:6.8.13"`***

```
version: '2'

services:
    web:
		container_name: ${APP_NAME}_web
        build: ./web
        environment:
             - APACHE_RUN_USER=#1000
        volumes:
             - ${APP_PATH_HOST}:${APP_PATH_CONTAINER}
        ports:
             - 8080:80
        working_dir: ${APP_PATH_CONTAINER}
             
    db:
		container_name: ${APP_NAME}_db
        image: mariadb
        restart: always
        environment:
           MYSQL_ROOT_PASSWORD: root
        volumes:
            - ${DB_PATH_HOST}:/var/lib/mysql    
			
    adminer:
		container_name: ${APP_NAME}_adminer
        image: adminer
        restart: always
        ports:
           - 6080:8080           
		   
    search:
       container_name: ${APP_NAME}_search
       image: elasticsearch:6.8.13
       ports:
           - 6200:9200 		   		   
		   
    composer:
        image: composer:1.6
        volumes:
            - ${APP_PATH_HOST}:${APP_PATH_CONTAINER}
        working_dir: ${APP_PATH_CONTAINER}    
        command: composer install
```		
	
---

`In the new terminal:`
	
	cd /var/www/LARAVEL/Elasticsearch/elastic_docker.loc

Cloned a project with github's:

	git clone https://github.com/mslobodyanyuk/elastic_amitavroy.git		  

Create an container:

	docker-compose up --build
		
OR ( 
_- IF you DO NOT need to rebuild the image, then use the command WITHOUT the "--build" option. Startup is much faster:_ )
	
	docker-compose up

All services started + the `vendor` folder appeared in the Laravel project.

`In another new terminal:`
	
	cd /var/www/LARAVEL/Elasticsearch/elastic_docker.loc

To enter the service inside the container:

	docker-compose exec web bash

OR
	
	docker exec -it Docker_Laravel_Es_web bash

To see the files of our project (- to check whether it enters our container):
	
	ls -l

If there is no `.env-file` of laravel-project you can also take in Docker structure folder. 
Rename `copy.env` to `.env`

Generate the application key:

	php artisan key:generate

`In the browser in the address bar ->`
	
	127.0.0.1:6080

Then Login and Create database in `adminer` named, for example, `elastic_amitavroy` -> choose encoding `utf8mb4_general_ci`:

	MySQL >> db	 
			->Create database

Configure the `.env` file of the Laravel project:

```
DB_HOST = db
DB_DATABASE = elastic_amitavroy
DB_USERNAME = root
DB_PASSWORD = root
```

Return to the `"container" terminal`:	
	
	php artisan migrate

run Seeders	
	
	php artisan db:seed

![screenshot of sample]( https://github.com/mslobodyanyuk/elastic_docker/blob/master/public/images/1.png )

---

For Testing the project:

	php artisan tinker
	>>>
	User::reindex();
	User::all();

_- Every time the database is filled, the Faker class randomly generates data. - Insert your substring from the database( - instead of "Vanessa" ) for search:_
	
	User::search('Vanessa');
	
![screenshot of sample]( https://github.com/mslobodyanyuk/elastic_docker/blob/master/public/images/2.png )

OR

`In Browser:`
		
	localhost:8080/public
				
			( - Laravel )	
	
_- Every time the database is filled, the Faker class randomly generates data. - Insert your substring from the database( - instead of "Vanessa" ) for search:_

'routes/web.php':

```php
use App\User;

Route::get('/', function () {
    return view('welcome');
});

Route::get('/users', function(){

//$user = User::all();
	
User::reindex();
$user = User::search('Vanessa');	//id: 10
	
return $user;
	
});
```
	
	localhost:8080/public/users

Error: 

_"ErrorException (E_WARNING) "file_put_contents(/var/www/app/storage/framework/sessions/qRzhr9WVVcwjv87mNyHw6NS9nVYJwDbUkf5Hs7wC): failed to open stream: No such file or directory"_

<https://stackoverflow.com/questions/38888568/laravel-file-put-contents-failed-to-open-stream-permission-denied-for-sessio>

	sudo chmod -R 777 /var/www/LARAVEL/Elasticsearch/elastic_docker.loc
		
	php artisan config:cache	

	localhost:8080/public/users

![screenshot of sample]( https://github.com/mslobodyanyuk/elastic_docker/blob/master/public/images/3.png )

---

Exit container:

`exit`

Stop container:

`Ctrl+C`

---

Useful commands for Ubuntu before using Docker & Elasticsearch.
===============================================================

- UBUNTU - "4+ commands":																							

<https://losst.ru/ochistka-sistemy-ubuntu>

1:

	sudo apt-get autoclean
	
		It is recommended to run this command periodically, cleaning the system of packages that it no longer needs.
2:

	sudo apt-get autoremove

		This command removes the remaining dependencies on packages that have been removed from the system.
3:

	sudo apt-get clean

		Clearing the cache and/or `/var/cache/apt/archives/`.
4:		
	
	sudo /usr/local/bin/remove_old_snaps.sh
	
- IF you create `remove_old_snaps.sh` before, like: 

`remove_old_snaps.sh`:

```
#!/bin/bash
set -eu
LANG=en_US.UTF-8 snap list --all | awk '/disabled/{print $1, $3}' |
while read snapname revision; do
snap remove "$snapname" --revision="$revision"
done
```	
		
`+`	

	sudo apt-get -f install
	
		 Clean up unnecessary packages after software removal, if any.

- DOCKER:
		
<https://habr.com/ru/company/flant/blog/336654/>
		
		Stopping and removing all containers:
		
	docker stop $(docker ps -a -q) && docker rm $(docker ps -a -q)
	
		Removing all images:
		
	docker rmi $(docker images -a -q)

- ALLOCATE MEMORY:

	`free -m`
	
	`sudo /bin/dd if=/dev/zero of=/var/swap.1 bs=1M count=1024`
	
	`sudo /sbin/mkswap /var/swap.1`
	
	`sudo /sbin/swapon /var/swap.1`

Error: 
	
_"proc_open(): fork failed - Cannot allocate memory"_	
	
<https://www.nicesnippets.com/blog/proc-open-fork-failed-cannot-allocate-memory-laravel-ubuntu>

####Useful links:

["Deploy on Docker( Ubuntu )" - Actions in README.MD]( https://github.com/mslobodyanyuk/KeyUA-test )

[The easiest and smallest laravel launch in docker | laravel installation in docker | #10)](https://www.youtube.com/watch?v=TumfGqUf39U&list=PLD5U-C5KK50XMCBkY0U-NLzglcRHzOwAg&index=13&t=164s)

[Docker commands](https://habr.com/ru/company/flant/blog/336654/)

Elasticsearch

<https://hub.docker.com/_/elasticsearch/>
										
UBUNTU

<https://losst.ru/ochistka-sistemy-ubuntu>				

<https://www.nicesnippets.com/blog/proc-open-fork-failed-cannot-allocate-memory-laravel-ubuntu>

Laravel

<https://stackoverflow.com/questions/38888568/laravel-file-put-contents-failed-to-open-stream-permission-denied-for-sessio>