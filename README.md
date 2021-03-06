# demokratian-docker

Se describen los pasos para realizar una instalación de [Demokratian](http://demokratian.org/) usando [**docker**](https://www.docker.com/) y [**docker-compose**](https://docs.docker.com/compose/). 

Se utilizarán Demokratian para crear la plataforma de votación telemática.

En el ejemplo partiré de un VPS con  **arch=amd64** (Ubuntu 20.04.1 LTS x86_64 en este ejemplo  ) con acceso root. 
## ¿Qué necesitaremos?
- Un usuario sobre el que trabajar (lo suponemos ya creado)
- Tener instalada una versión de [**docker**](https://docker.com) y de [**docker-compose**](https://docs.docker.com/compose/install/). 
- Incluir el usuario (*nombre-usuario*) en el **grupo *docker*** que se creó en la instalación de docker.
- Tener instalada una versión actualizada de [**git**](https://git-scm.com/download/linux). 
- Descargarnos los [fuentes de Demokratian](https://bitbucket.org/csalgadow/demokratian_votaciones/src/master/): Para este ejemplo partiré de la versión 3.1.0
- Elegir la imagen LAMP que dará cobertura a la aplicación Demokratian
- Crear y configurar el fichero *docker-compose.yml* que permita el despliegue de Demokratian.
- Realizar ajustes para el buen funcionamiento (Base de datos y código PHP).

## Instalación de DOCKER
Resumo lo que se indica en [la página oficial de docker](https://docs.docker.com/engine/install/ubuntu/)
```
# Desinstalar antiguas versiones de docker
sudo apt-get remove docker docker-engine docker.io containerd runc
# Prepara paquetes necesarios para funcionamiento de docker
sudo apt-get update
sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common
# Añadir la clave oficial GPG de dock
er
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
#Se agrega el repositorio de docker
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
# Actualizar la información del repositorio e instalar DOCKER
sudo apt update
sudo apt-get install docker-ce docker-ce-cli containerd.io
# Comprobar el estado de DOCKER
sudo systemctl status docker
```
Adicionalmente, para **aprovechar la utilizad docker-compose** (que se apoya en docker) a la hora de desplegar la aplicación necesitaríamos instalarla:
```
 sudo apt install docker-compose
```
## Instalar GIT
[Instalamos git](https://git-scm.com/book/es/v2/Inicio---Sobre-el-Control-de-Versiones-Instalaci%C3%B3n-de-Git) para poder acceer al código de Demokratian; adicionalmente podemos realizar los cambios que consideremos necesarios para personalizar la aplicación y guardarlo en un control del versiones:
```
sudo apt-get install git

```
## Incluir usuario en grupo docker
Necesitamos que el usuario de trabajo tenga acceso de trabajar con docker para no tener que tener privilegios de root:
```
sudo usermod -aG docker nombre-usuario 
```

> A partir de este momento trabajaremos con el usuario ***nombre-usuario***  (que ya tiene permisos para trabajar con **docker**)

## Descargar fuentes Demokratian
Con las herramientas **git** instaladas ya podemos descargar el código fuente. Para ello accedemos al [repositorio de Demokratian](https://bitbucket.org/csalgadow/demokratian_votaciones/src/master/) y lo *clonamos* (pulsando sobre el botón **Clone**) y copiamos el comando de clonación y nos lo llevamos a nuestro servidor. Previamente habremos creado un directorio sobre el que trabajar (**demokratian-docker**). 
```
mkdir demokratian-docker
cd demokratian-docker
# Comando de clonación obtenido en el repositorio de Demokratian
git clone https://bitbucket.org/csalgadow/demokratian_votaciones.git   
```
## Imagen para Demokratian
Como se comenta en el fichero [README.md](https://bitbucket.org/csalgadow/demokratian_votaciones/src/master/README.md) del repositorio de Demokratian, para el buen funcionamiento de Demokratian se ***Necesita un servidor apache con php7.2 y una base de datos mysql***. Es decir, un *servidor LAMP* que cumpla esos requisistos sería suficiente.
De entre la imágenes de docker que ofrecen un servidor lamp completo; ejecutamos:
```
docker search lamp
NAME                        DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED
mattrayner/lamp             A simple LAMP docker image running the prere…   229                                     [OK]
linode/lamp                 LAMP on Ubuntu 14.04.1 LTS Container            178
...
```
Nos quedamos con la primera [**mattrayner/lamp**](https://hub.docker.com/r/mattrayner/lamp) que está muy recomendada, aunque para conocer el funcionamiento de la imagen y utilizarla iremos a la página de [github](https://github.com/mattrayner/docker-lamp). Esta imagen, adicionalmente, instala la utilidad **phpmyadmin** que nos servirá como cliente con el que gestionar la base de datos MySQL desde la web. 

Para este repositorio [se ofrecen tres versiones](https://github.com/mattrayner/docker-lamp/blob/master/README.md): 

Component | `latest-1404` | `latest-1604` | `latest-1804`
---|---|---|---
[Apache][apache] | `2.4.7` | `2.4.18` | `2.4.29`
[MySQL][mysql] | `5.5.62` | `5.7.30` | `5.7.30`
[PHP][php] | `7.3.3` | `7.4.6` | `7.4.6`
[phpMyAdmin][phpmyadmin] | `4.8.5` | `5.0.2` | `5.0.2`

Eligiremos el tag correspondiente según nuestros intereses.

## Despliegue de Demokratian en servidor LAMP
En este ejemplo vamos a desplegar Demokratian en el puerto 8001 del servidor host, con lo que **es neceario tener abierto en el firewall dicho puerto**.

Por otra parte, en nuestro caso utilizaremos la imagen:tag **mattrayner/lamp:latest-1804**:

A continuación se muestran las dos opciones de despliegue:
- Despligue con **docker**
- Despliegue con **docker-compose**

### Despligue con docker
Si queremos desplegar Demokratian con docker hemos de ejecutar:

```
docker run -i -t --name lamp-demokratian -p "8001:80" -v ${PWD}/demokratian_votaciones:/app -v ${PWD}/mysql:/var/lib/mysql -d mattrayner/lamp:latest-1804

```
En el comando docker realizamos el mapeo entre puerto en el host -8001- y puerto en el container -80-.

### Despliegue con docker-compose.yml
Dentro del directorio **demokratian-docker** creamos el fichero **docker-compose.yml** y realizaremos los ajustes que consideremos necesarios.

```
version: '3'

services:
  lamp:
    image: mattrayner/lamp:latest-1804
    container_name: lamp-demokratian
    volumes:
       - ./demokratian_votaciones:/app # Fuentes de Demokratian en el host (directorio demokratian_votaciones) y correspondencia de directorio en el container (directorio /app)
       - ./mysql:/var/lib/mysql # Información de MySQL en el host (directorio mysql) y correspondencia de directorio en el container (directorio /var/lib/mysql)
    ports:
      - "8001:80" # Puertos sobre el escucha Apache (puerto en el host -8001- y puerto en el container -80-)
#      - "3306:3306" # Puertos sobre el escucha Mysql (puerto en el host -3306- y en el container -3306-) 
    restart: always

```
A partir de aquí ya podemos desplegar la aplicación. 
```
docker-compose up -d
```

 Ya haya sido con docker, o con docker-compose el despliegue de Demokratian (la primer vez tardará más pues se tiene que descargar la imagen) ya podemos comprobar que la aplicación está funcionando en un contenedor.
```
docker ps -a
CONTAINER ID        IMAGE                         COMMAND             CREATED              STATUS              PORTS                                          NAMES
8ea8b397eac1        mattrayner/lamp:latest-1804   "/run.sh"           About a minute ago   Up About a minute   0.0.0.0:3306->3306/tcp, 0.0.0.0:8001->80/tcp   lamp-demokratian
```
Vemos que el contenedor se llama **lamp-demokratian** y por el Status vemos que está **UP**; es decir ya podríamos acceder a demokratian; en este caso debería funcionar en la URL **http://localhost:8001/** (el port que configuramos en docker-compose.yml)
Podemos comprobarlo en la línea de comandos 
```
curl http://localhost:8001/
```

Si necesatamos acceder a la base de datos podemos aceder a través de la utilidad phpmyadmin. Podremos conocer el usuario y la contraseña de administración si miramos en los logs del contenedor.
```
docker logs lamp-demokratian
...
========================================================================
You can now connect to this MySQL Server with Ii0U9KGcgMLh

    mysql -uadmin -pIi0U9KGcgMLh -h<host> -P<port>

Please remember to change the above password as soon as possible!
MySQL user 'root' has no password but only allows local connections
...

```
En este caso en particular, teniendo en cuenta la información del log, podríamos Administrar MySQL con **phpmyadmin** mediante la URL **http://localhost:8001/phpmyadmin** con usuario (**admin**) y password (**Ii0U9KGcgMLh**).
## Ajustes
### Base de Datos
Antes de poder instalar y utilizar demokratian necesitaremos:
- Crear una base de datos dentro de Mysql (a través de la utilidad phpmyadmin - **http://localhost:8001/phpmyadmin**)
- Crear un usuario de base de datos
- Dar permisos al usuario sobre la base de datos

Otro ajuste a tener en cuenta está realacionado con la Base de Datos. **Para evitar que aparezcan errores en el alta de entidades** (candidatos, interventores...) debemos [**desactivar la directiva STRICT_TRANS_TABLES**](https://stackoverflow.com/questions/40881773/how-to-turn-on-off-mysql-strict-mode-in-localhost-xampp) del **sql_mode** de la base de datos:
Ejecutamos en la consola de SQL de phpmyadmin:
```
SHOW VARIABLES LIKE 'sql_mode';
+--------------+------------------------------------------+ 
|Variable_name |Value                                     |
+--------------+------------------------------------------+
|sql_mode      |STRICT_TRANS_TABLES,NO_ENGINE_SUBSTITUT...|
+--------------+------------------------------------------+
# Del valor obtenido quitamos, la directiva STRICT_TRANS_TABLES y actualizamos el valor.
set global sql_mode='ONLY_FULL_GROUP_BY,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION';
```
A partir de aquí ya podríamos instalar la aplicación **demokratian** desde la web (**http://localhost:8001**).

> Recordar que cada vez que se cree el contenedor (bien la primera vez, o pq se ha eliminado), es necesario que la directiva STRICT_TRANS_TABLE no esté en la variable sql-mode de MySql

## Reconocimientos
Este repositorio es posible gracias al apoyo de [Carlos Salgado](http://carlos-salgado.es/) *alma mater* de [Demokratian](http://demokratian.org/)
