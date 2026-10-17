# 🌐 NimbusDocs — Portal Documental Seguro Multi-Marca
En este README.md haré una documentación de cada carpeta explicando detalladamente lo que hay dentro de ellas y con que finalidad lo he hecho. He intentado crear una infraestructura ordenada, organizada y sencilla de entender.

Consta de una carpeta principal **servidor-web-apache2** donde dentro encontramos una serie de carpetas. El esqueleto de mi trabajo ahora mismo es así:

## Esquema de la infraestructura

- servidor-web-apache2
    - configuracion-virtualhosts
        - 000-nimbuspordefecto.conf
        - nimbuscoches.conf
        - nimbusmotos.conf
    - html
        - nimbuscoches
            - index.html
        - nimbusmotos
            - index.html
    - ssl
        - nimbus
            - nimbuscoches.crt
            - nimbuscoches.key
    - nginx
        - nimbuscoches.conf
        - nimbusmotos.conf
        - pordefecto.conf
- claves.txt
- docker-compose.yml

## 1. ESTRUCTURA DE CONFIGURACIÓN DE APACHE.

- **configuracion-virtualhosts**. Esta carpeta la he creado para tener toda la configuración de virtualhosts de manera ordenada. Dentro de esta carpeta se encuentra toda la configuración de los virtualhosts. En total he configurado 3 archivos .conf.

    - 000-nimbuspordefecto.conf. Este es el archivo de configuración del virtualhost por defecto. Le he puesto que conmience por esa serie de numeros ("000") ya que nos asegura que Apache lo cargue en primer lugar, es decir, la función de este virtualhost será interceptar cualquier petición cuyo dominio no concida con ningún sitio configurado. Gestionar un comportamiento controlado, como denegar el acceso, error 403 Forbidden.

    - nimbuscoches.conf. Archivo que define el virtualhost para el sitio "nimbuscoches". En él se establece el Servername, Documentroot que es la ruta física del contenido y los archivos de registro específicos para este sitio.

    - nimbusmotos.conf. Archivo que define el virtualhost para el sitio "nimbusmotos". Es practicamente la misma configuración que el archivo anterior pero con una gran diferencia que es el bloque "<Directory>" con la instrucción AllowOverride All para permitir que Apache precese el archivo .htaccess de nimbusmotos. También he configurado el bloque <Files> con la instrucción Require all denied, que literalmente dice que si alguien intenta acceder a un archivo que se llame exactamente claves.txt le deniege el acceso.

## 2. CONTENIDO DE LOS SITIOS.

- **html**. Esta carpeta es también necesaria para tener todos los contenidos de una vez y facilitar el volumen en docker-compose.yml. Es decir, se montará como volumen en el contenedor para servir los contenidos de los sitios.

    - nimbuscoches/index.html. Documento HTML que actúa como la página principal de bienvenida del sitio de coches.

    - nimbusmotos/index.html. Documento HTML de bienvenido a nimbusmotos.

## 3. ESTRUCTURA DE CONFIGURACIÓN DE NGINX

- nginx. Es la
    - nimbuscoches.conf
    - nimbusmotos.conf
    - pordefecto.conf

- claves.txt. Archivo de texto que contiene información confidencial como son las claves de los usuarios y su acceso está bloqueado gracias a las reglas de .htaccess.