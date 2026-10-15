Aquí es donde tendremos todas las configuraciones.
Consta de una carpeta principal servidor-web-apache2 donde dentro encontramos una serie de carpetas. El esqueleto de mi trabajo ahora mismo es así:

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
            - claves.txt
            - .htaccess
- docker-compose.yml

Haré una documentación de cada carpeta explicando detalladamente lo que hay dentro de ellas y con que finalidad lo he hecho.

1. ESTRUCTURA DE CONFIGURACIÓN DE APACHE.
En la carpeta configuracion-virtualhosts podemos ver que se encuentran tres archivos con extesión .conf.

    - 000-nimbuspordefecto.conf. Este es el archivo de configuración del virtualhost por defecto. Le he puesto que conmience por esa serie de numeros ("000") ya que nos asegura que Apache lo cargue en primer lugar, es decir, la función de este virtualhost será interceptar cualquier petición cuyo dominio no concida con ningún sitio configurado. Gestionar un comportamiento controlado, como denegar el acceso, error 403 Forbidden.

    - nimbuscoches.conf. Archivo que define el virtualhost para el sitio "nimbuscoches". En él se establece el Servername, Documentroot que es la ruta física del contenido y los archivos de registro específicos para este sitio.

    - nimbusmotos.conf. Archivo que define el virtualhost para el sitio "nimbusmotos". Es practicamente la misma configuración que el archivo anterior pero con una gran diferencia que es el bloque "<Directory>" con la instrucción AllowOverride All para permitir que Apache precese el archivo .htaccess de nimbusmotos.

2. DIRECTIVO RAÍZ DE LOS SITIOS.
Es la carpeta principal que se montará como volumen en el contenedor para servir los archivos.

    - nimbuscoches/index.html. Documento HTML que actúa como la página principal de bienvenida del sitio de coches.
    - nimbusmotos/. Carpeta raíz del sitio de motos, la cual incluye los siguientes archivos de configuracion de seguridad.
        - .htaccess. Archivo 
        - claves.txt.
        - index.html. 