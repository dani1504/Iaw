# **🌐 NimbusDocs — Portal Documental Seguro Multi-Marca**

## **ÍNDICE**

* **<big>[Esquema de la infraestructura](#esquema-de-la-infraestructura)**</big>
    * **<big>[2. CONTENIDO DE LOS SITIOS.](#2-contenido-de-los-sitios)**</big>
    * **<big>[3. ESTRUCTURA DE CONFIGURACIÓN DE NGINX](#3-estructura-de-configuración-de-nginx)**</big>
    * **<big>[4. OTROS ARCHIVOS](#4-otros-archivos)**</big>
    * **<big>[I. Diseño y Requisitos](#i-diseño-y-requisitos)**</big>
    * **<big>[II. Fase 1 — Backend Apache Multi-Marca](#ii-fase-1--backend-apache-multi-marca)**</big>
    * **<big>[III. Fase 2 — Terminación TLS en Nginx](#iii-fase-2--terminación-tls-en-nginx)**</big>
    * **<big>[IV. Fase 2 — Seguridad y Límite de Peticiones](#iv-fase-2--seguridad-y-límite-de-peticiones)**</big>
    * **<big>[V. Fase 3 — Diagnóstico Provocado](#v-fase-3--diagnóstico-provocado)**</big>

---

## **Introducción**

En este README.md haré una documentación de cada carpeta explicando detalladamente lo que hay dentro de ellas y con que finalidad lo he hecho. He intentado crear una infraestructura ordenada, organizada y sencilla de entender.

Consta de una carpeta principal **servidor-web-apache2** donde dentro encontramos una serie de carpetas. El esqueleto de mi trabajo ahora mismo es así:

# **Esquema de la infraestructura**

- <big>**servidor-web-apache2**</big>
    - <big>**configuracion-virtualhost**</big>
        - <big>**000-nimbuspordefecto.conf**</big>
        - <big>**nimbuscoches.conf**</big>
        - <big>**nimbusmotos.conf**</big>
    - <big>**html**</big>
        - <big>**nimbuscoches**</big>
            - <big>**index.html**</big>
        - <big>**nimbusmotos**</big>
            - <big>**index.html**</big>
            - <big>**claves.txt**</big>
            - <big>**.htaccess**</big>
    - <big>**nginx**</big>
        - <big>**nimbuscoches.conf**</big>
        - <big>**nimbusmotos.conf**</big>
        - <big>**pordefecto.conf**</big>
        - <big>**limite.conf**</big>
    - <big>**ssl**</big>
        - <big>**nimbuscoches.crt**</big>
        - <big>**nimbuscoches.key**</big>
- <big>**docker-compose.yml**</big>
- <big>**README.md**</big>

## **1. ESTRUCTURA DE CONFIGURACIÓN DE APACHE.**

- **configuracion-virtualhosts**. Esta carpeta la he creado para tener toda la configuración de virtualhosts de manera ordenada. Dentro de esta carpeta se encuentra toda la configuración de los virtualhosts. En total he configurado 3 archivos .conf.

    - `000-nimbuspordefecto.conf`. Este es el archivo de configuración del virtualhost por defecto. Posteriormente explicaré el funcionamiento de este archivo y porque motivo lo he creado.

    - `nimbuscoches.conf`. Archivo que define el virtualhost para el sitio "nimbuscoches". En él se establece el Servername, Documentroot que es la ruta física del contenido y los archivos de registro específicos para este sitio.

    - `nimbusmotos.conf`. Archivo que define el virtualhost para el sitio "nimbusmotos". Es practicamente la misma configuración que el archivo anterior pero con algunas diferencias que serán explicadas en este README.md.

## **2. CONTENIDO DE LOS SITIOS.**

- `html`. Esta carpeta es también necesaria para tener todos los contenidos de una vez y facilitar el volumen en docker-compose.yml. Es decir, se montará como volumen en el contenedor para servir los contenidos de los sitios.

    - `nimbuscoches/index.html`. Documento HTML que actúa como la página principal de bienvenida del sitio de coches.

    - `nimbusmotos/index.html`. Documento HTML de bienvenido a nimbusmotos.
        - `claves.txt`. Archivo de texto que contiene información confidencial como son las claves de los usuarios y su acceso está bloqueado gracias a las reglas de .htaccess.
        - `.htaccess`. Reglas de serguridad para restringir el acceso a claves.txt.

## **3. ESTRUCTURA DE CONFIGURACIÓN DE NGINX**

- `nginx`. Es donde se alojan todas las configuraciones de Nginx como proxy inverso HTTPS.
    - `nimbuscoches.conf`. Servidor HTTPS y proxy para nimbuscoches.com
    - `nimbusmotos.conf`. Servidor HTTPS y proxy para nimbusmotos.com
    - `pordefecto.conf`. Servidor por defecto de Nginx para peticiones no reconocidas
    - `limite.conf`. Configuración de la zona de memoria y tasa del rate limiting.

## **4. OTROS ARCHIVOS**

- `docker-compose.yml`. Es el archivo de configuración en formato YAML que utiliza Docker Compose para definir, orquestar y ejecutar los contenedores.

## **I. Diseño y Requisitos**

Es necesario documentar las siguientes decisiones. Los dos dominios ficticios que he elegido son:
- `nimbuscoches.com`
- `nimbusmotos.com`

La razón de por qué he elegido estos nombres para el dominio es sencilla. Pues al estar haciendo la práctica con estos nombres me facilitan distinguir los dos servicios rápidamente, además de que es una forma de tener la arquitectura más ordenada, facilitando la identificación y navegación para el usuario. 

El MPM (Multi-Processing Module) de Apache elegido es `event`. Un MPM es el componente que determina como Apache gestiona las conexiones y los procesos que manejan las solicitudes de los usuarios, existen tres: prefork, worker y event. No he tenido que configurar nada ya que la imagen que he usado viene con una configuración base ya optimizada. Event es el más eficiente para manejar la concurrencia de Apache. Lo que más me ha sorprendido de este MPM es que a diferencia de los otros dos que mantienen hilos ocupados esperando a que las conexiones inactivas se queden libres. Event lo que hace es delegar esas conexiones inactivas (keep-alive) a hilos dedicados, liberando los hilos de trabajo reales para que puedan procesar peticiones activas de nuevos usuarios, mejorando el rendimiento general del servidor.

## **II. Fase 1 — Backend Apache Multi-Marca**

### - **Demostrado con curl, usando la cabecera Host explícita, que cada dominio devuelve el contenido correcto.**
Hemos usado el comando curl con el dominio del sitio. 

*Salida:*
---
```bash
curl -H "Host: nimbusmotos.com" http://localhost/
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Nimbusdocs Página motos</title>
</head>
<body>
      
    <h1>Bienvenidos a Nimbusmotos!</h1>
    <p>En esta página encontraras toda la información de coches que necesites</p>
    
</body>
</html>
```
---
*Salida:*
---
```bash
curl -H "Host: nimbuscoches.com" http://localhost/
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Nimbusdocs Pagina coches</title>
</head>
<body>
    
    <h1>Bienvenidos a Nimbuscoches!</h1>
    <p>En esta página encontraras toda la información de coches que necesites</p>
    

</body>
</html>
```
---
### - **La marca "interna" (la que decidáis que no debe ser pública) tiene que proteger un fichero sensible simulado — con nombre propio, no wp-config.php — y desactivar el listado de directorios en toda su ruta.**

He elegido un fichero sensible claves.txt que dentro contiene información que no quiero que nadie lea. Lo que he configurado para hacer esto ha sido la configuración del virtualhost de uno de mis sitios, en este caso `nimbusmotos.com`. La configuración consiste en colocar el bloque `<Directory>` con la instrucción `AllowOverride All` para permitir que Apache precese el archivo `.htaccess` de nimbusmotos. Y configurar también el .htaccess con bloque `<Files>` configurando la instrucción `Require all denied`, que literalmente dice que si alguien intenta acceder a un archivo que se llame exactamente claves.txt le deniege el acceso.

### - **Un dominio que no coincida con ningún Virtualhost debe caer en un comportamiento controlado por vosotros (no el primero al azar) — explicado en el README porqué pasa eso.**

Para controlar qué ocurre cuando un dominio no coincide con ningún sitio, he creado el archivo de configuración `000-nimbuspordefecto.conf`. He nombrado el archivo comenzando por "000-" para que Apache lo cargue en primer lugar (ya que lee los archivos en orden alfabético). Al ser el primero en cargarse, Apache lo asigna internamente como su VirtualHost por defecto.

#### **¿Por qué pasa esto?**
Cuando el servidor recibe una petición cuya cabecera *Host* no coincide con ninguno de los sitios que tenemos configurados que en este caso son (`nimbusmotos.com` o `nimbuscoches.com`), Apache necesita saber qué hacer con ella. Si no tuviéramos el virtualhost por defecto, Apache serviría simplemente el primer sitio web que encontrara configurado, mostrando contenido de una de las marcas a un dominio no reconocido. Su función principal es denegar el acceso y devolver un código de error **403 Forbidden**, protegiendo nuestra infraestructura.

## **III. Fase 2 — Terminación TLS en Nginx**

### - **Generar vuestro propio certificado de laboratorio (curva ECDSA, igual que en UD1) con el CN de una de vuestras dos marcas.**

El certificado se encuentra dentro de una carpeta que he creado /ssl y el comando que he utilizado es el que viene a continuación:

---
```bash
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout nimbuscoches.key -out nimbuscoches.crt
```
---

### - **Configurar el bloque server de Nginx con ssl_protocols restringido a TLS 1.2 y 1.3 (nada de versiones obsoletas) y un cipher suite razonado en el README.**

---
```bash
server {
    listen 443 ssl;
    server_name nimbuscoches.com www.nimbuscoches.com;

    ssl_certificate /etc/nginx/ssl/nimbuscoches.crt;
    ssl_certificate_key /etc/nginx/ssl/nimbuscoches.key;

    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers 'ECDHE-ECDSA-AES256-GCM-SHA384';
    ssl_prefer_server_ciphers on;
}
```
---
Esta es la configuración del bloque server de uno de mis sitios (`nimbuscoches.com`). Veamos la configuración desglosada para entenderlo mejor:

- `server { ... }`. Es el bloque que contiene todas las reglas de procesamiento para un sitio web dentro del servidor.
- `listen 443 ssl;`. Ponemos esto para indicarle a nginx que debe escuchar peticiones en el puerto 443 y que debe aplicar la capa SSL/TLS a todo el tráfico que entre por él.
- `server_name nimbuscoches.com www.nimbuscoches.com;`. Para establecer los nombres de dominio a los que responde este bloque de configuración.
- `ssl_certificate /etc/nginx/ssl/nimbuscoches.crt;`. Ruta absoluta dentro del contenedor donde se encuentra el certificado público.
- `ssl_certificate_key /etc/nginx/ssl/nimbuscoches.key;`. Ruta donde se guarda la clave privada asociada al certificado, sirve para descifrar las peticiones que envían los clientes.
- `ssl_protocols TLSv1.2 TLSv1.3;`. La restricción de protocolos.
- `ssl_prefer_server_ciphers on;`. Esto lo que hace es forzar al servidor a imponer su lista de algoritmos sobre la lista de cliente. Con esto evitamos que un cliente antiguo o malicioso intente negociar la conexión utilizando un cifrado débil si el servidor dispone de uno más seguro.

#### **¿Por qué ese cipher suite?**

La elección de este cipher suite es por la necesidad de implementar seguridad en nuestra práctica. He escogido esta combinación específica por cuatro razones:

- `ECDHE` (Intercambio de claves): Es el método matemático para que el cliente y el servidor se pongan de acuerdo en una clave secreta a través de Internet sin que nadie pueda deducirla.

- `RSA` (Autenticación): Es el tipo de firma de nuestro certificado, demuestra al usuario que yo poseeo la clave privada correcta.

- `AES256-GCM` (Cifrado de datos): El algoritmo es muy rápido ya que empaqueta y encripta el contenido web real usando la clave acordada.

- `SHA384` (Sello de integridad): Es una función hash usada para la integridad, es decir, si alguien intenta cambiar un solo bit del mensaje por el camino, el SHA384 no coincidirá y el navegador rechazará la conexión por seguridad.

### - **Redirigir automáticamente todo el tráfico HTTP (puerto 80) a HTTPS con un código de estado correcto — justificado en el README por qué ese código y no otro.**

---
```bash
server {
    listen 80;
    server_name nimbuscoches.com www.nimbuscoches.com;
    return 301 https://$host$request_uri;
}
```
---
Este bloque es el que he configurado en el server block de mi sitio `nimbuscoches.com`, literalmente lo que estamos haciendo es redirigir automáticamente todo el tráfico HTTP a HTTPS. 

Las razones de porque he usado la redirección **301** (Moved Permanently) son las siguientes:
1. Indica que la versión HTTP ha sido reemplazada definitivamente. Además de que los navegadores la memorizan en caché y transforman futuras peticiones a `https://` de forma local, evitando enviar tráfico en claro y reduciendo el riesgo de ataques **Man-in-the-Middle**.

2. Las otras opciones era usar **302** / **307** pero no he usado ninguna de estas porque al ser redirecciones temporales el navegador no las guarda en caché como lo hace la redirección 301. Haría al cliente conectarse siempre por el puerto 80, dejando la conexión expuesta cada vez que inicie sesión.

### - **Nginx debe reenviar la petición a Apache preservando la cabecera Host original — sin eso, Apache no puede resolver a qué marca corresponde la petición.**

---
```bash
proxy_set_header Host $host;
```
---

Al añadir `proxy_set_header Host $host;`, Nginx conserva el dominio original solicitado por el usuario (`nimbusmotos.com`o `nimbuscoches.com`) y se lo traslada a Apache. Con esta información, Apache identifica correctamente la marca solicitada y sirve el sitio web con un código 200 OK.

### - **Verificar con openssl s_client el protocolo y cifrado negociados, y adjuntar la salida real (no una captura de pantalla del guión) en el README.**

- En `nimbuscoches.com`:

---
```bash
echo | openssl s_client -connect localhost:443 -servername nimbuscoches.com -tls1_3 -CAfile ./ssl/nimbuscoches.crt
```
```bash
CONNECTED(00000003)
depth=0 CN = nimbuscoches
verify return:1
---
Certificate chain
 0 s:CN = nimbuscoches
   i:CN = nimbuscoches
   a:PKEY: id-ecPublicKey, 256 (bit); sigalg: ecdsa-with-SHA256
   v:NotBefore: Oct 16 09:57:37 2026 GMT; NotAfter: Oct 16 09:57:37 2027 GMT
---
Server certificate
-----BEGIN CERTIFICATE-----
MIIBgjCCASmgAwIBAgIUZFlIBFto6L4QZSLfLKWWpLgzGKwwCgYIKoZIzj0EAwIw
FzEVMBMGA1UEAwwMbmltYnVzY29jaGVzMB4XDTI2MTAxNjA5NTczN1oXDTI3MTAx
NjA5NTczN1owFzEVMBMGA1UEAwwMbmltYnVzY29jaGVzMFkwEwYHKoZIzj0CAQYI
KoZIzj0DAQcDQgAEE5xgYYU1FhkOJz9L20p0fgIu4Zbjt8jzSalgNBR5RiSTPbsB
K40PSdTEPcTkiqVby/cYdvUNuRGtoCjFMRQVZKNTMFEwHQYDVR0OBBYEFIfgYuZn
ZHA4k40bgkyyCr9ap8hrMB8GA1UdIwQYMBaAFIfgYuZnZHA4k40bgkyyCr9ap8hr
MA8GA1UdEwEB/wQFMAMBAf8wCgYIKoZIzj0EAwIDRwAwRAIgVk/LnWNmFEKi4Lpq
HXNRsXyU6TOMGZRpO8Yn0a+OXKACICceoCebWNjvBhe5/MO6WqIcXjy5A8FNY07q
bdJ27Zhv
-----END CERTIFICATE-----
subject=CN = nimbuscoches
issuer=CN = nimbuscoches
---
No client certificate CA names sent
Peer signing digest: SHA256
Peer signature type: ECDSA
Server Temp Key: X25519, 253 bits
---
SSL handshake has read 788 bytes and written 330 bytes
Verification: OK
---
New, TLSv1.3, Cipher is TLS_AES_256_GCM_SHA384
Server public key is 256 bit
Secure Renegotiation IS NOT supported
Compression: NONE
Expansion: NONE
No ALPN negotiated
Early data was not sent
Verify return code: 0 (ok)
---
DONE
```
---

- En `nimbusmotos.com`:

---
```bash
echo | openssl s_client -connect localhost:443 -servername nimbusmotos.com -tls1_3 -CAfile ./ssl/nimbuscoches.crt
```
```bash
CONNECTED(00000003)
depth=0 CN = nimbuscoches
verify return:1
---
Certificate chain
 0 s:CN = nimbuscoches
   i:CN = nimbuscoches
   a:PKEY: id-ecPublicKey, 256 (bit); sigalg: ecdsa-with-SHA256
   v:NotBefore: Oct 16 09:57:37 2026 GMT; NotAfter: Oct 16 09:57:37 2027 GMT
---
Server certificate
-----BEGIN CERTIFICATE-----
MIIBgjCCASmgAwIBAgIUZFlIBFto6L4QZSLfLKWWpLgzGKwwCgYIKoZIzj0EAwIw
FzEVMBMGA1UEAwwMbmltYnVzY29jaGVzMB4XDTI2MTAxNjA5NTczN1oXDTI3MTAx
NjA5NTczN1owFzEVMBMGA1UEAwwMbmltYnVzY29jaGVzMFkwEwYHKoZIzj0CAQYI
KoZIzj0DAQcDQgAEE5xgYYU1FhkOJz9L20p0fgIu4Zbjt8jzSalgNBR5RiSTPbsB
K40PSdTEPcTkiqVby/cYdvUNuRGtoCjFMRQVZKNTMFEwHQYDVR0OBBYEFIfgYuZn
ZHA4k40bgkyyCr9ap8hrMB8GA1UdIwQYMBaAFIfgYuZnZHA4k40bgkyyCr9ap8hr
MA8GA1UdEwEB/wQFMAMBAf8wCgYIKoZIzj0EAwIDRwAwRAIgVk/LnWNmFEKi4Lpq
HXNRsXyU6TOMGZRpO8Yn0a+OXKACICceoCebWNjvBhe5/MO6WqIcXjy5A8FNY07q
bdJ27Zhv
-----END CERTIFICATE-----
subject=CN = nimbuscoches
issuer=CN = nimbuscoches
---
No client certificate CA names sent
Peer signing digest: SHA256
Peer signature type: ECDSA
Server Temp Key: X25519, 253 bits
---
SSL handshake has read 787 bytes and written 329 bytes
Verification: OK
---
New, TLSv1.3, Cipher is TLS_AES_256_GCM_SHA384
Server public key is 256 bit
Secure Renegotiation IS NOT supported
Compression: NONE
Expansion: NONE
No ALPN negotiated
Early data was not sent
Verify return code: 0 (ok)
---
DONE
```
---

## **IV. Fase 2 — Seguridad y Límite de Peticiones**

### **Cabeceras de seguridad mínimas: Strict-Transport-Security, X-Content-Type-Options y una tercera que elijáis y justifiquéis vosotros (repasad UD1, tabla de cabeceras).**

- `Strict-Transport-Security`. El navegador recuerda durante `max-age` segundos(en este caso he puesto 1 año en segundos)que ese dominio solo va por HTTPS, rechazando cualquier intento de degradación a HTTP no cifrado.
- `X-Content-Type-Options "nosniff"`. Prohíbe que los navegadores intenten adivinar el tipo MIME de los archivos (MIME sniffing), forzándolos a respetar estrictamente el `Content-Type` enviado por el servidor para prevenir la ejecución de scripts maliciosos camuflados.
- `X-Frame-Options "SAMEORIGIN"`: Previene ataques de Clickjacking. Sin esta cabecera, un atacante podría incrustar el sitio web legítimo dentro de un marco transparente (`<iframe>`) en una página maliciosa de terceros, engañando al usuario para que interactúe con botones o formularios invisibles de la web original. Con `SAMEORIGIN`, se instruye al navegador para que solo permita incrustar la página en marcos si el sitio contenedor pertenece exactamente al mismo dominio de origen, bloqueando cualquier intento de renderizado desde dominios externos.

### **Demostrad con una ráfaga de peticiones repetidas (un bucle for con curl, o ab desde un contenedor cliente) que el límite calculado realmente bloquea, adjuntando la salida.**

El burcle for que he utilizado para demostrar el limit_req_zone es el siguiente:

---
```bash
for i in {1..20}; do curl -s -o /dev/null -w "Petición $i: HTTPS %{http_code}\n" -k -H "Host: nimbusmotos.com" https://localhost/; done
```
---

Salida:

---
```bash
Petición 1: HTTPS 200
Petición 2: HTTPS 200
Petición 3: HTTPS 200
Petición 4: HTTPS 200
Petición 5: HTTPS 200
Petición 6: HTTPS 200
Petición 7: HTTPS 503
Petición 8: HTTPS 503
Petición 9: HTTPS 503
Petición 10: HTTPS 503
Petición 11: HTTPS 503
Petición 12: HTTPS 503
Petición 13: HTTPS 503
Petición 14: HTTPS 503
Petición 15: HTTPS 503
Petición 16: HTTPS 503
Petición 17: HTTPS 503
Petición 18: HTTPS 503
Petición 19: HTTPS 503
Petición 20: HTTPS 503
```
---

---
```bash
limit_req_zone $binary_remote_addr zone=limit_nimbus:10m rate=3r/s;
```
---

- `rate=3r/s`: Cubre exactamente las 3 peticiones por segundo que genera un usuario legítimo leyendo documentación de forma continua.

- `burst=5 nodelay`: Cuando un usuario legítimo recarga varias pestañas a la vez, genera un pico de 6 peticiones simultáneas. Nginx procesa 1 petición dentro de la tasa base y aloja 5 en la ráfaga (burst=5). Al añadir nodelay, las 6 peticiones se responden de forma instantánea con un código HTTP 200 OK, sin introducir latencia ni penalizar al usuario legítimo.

Ante un script automatizado que lanza 20 peticiones por segundo, Nginx procesa las primeras 6 peticiones permitidas y rechaza inmediatamente las 14 sobrantes devolviendo un código `HTTP 503 Service Temporarily Unavailable`, conteniendo el ataque sin sobrecargar el servidor backend Apache.

Esta sería la configuración para añadir al server block de cada sitio que tenemos:


---
```bash
    location / {
        limit_req zone=limit_nimbus burst=5 nodelay;
        proxy_pass http://apache:80;
        proxy_set_header Host $host;
    }
```
---

## **V. Fase 3 — Diagnóstico Provocado**

Para esta parte de la práctica, he provocado dos fallos de configuración a propósito para entender cómo se comunican las distintas piezas (`Apache` y `Nginx`) y cómo se aplican los permisos. No son simples errores de teclear mal una ruta, sino problemas de concepto.

- **1 fallo relacionado con la parte Apache (Fase 1: Virtual Host, protección de recursos).**
    - El primer fallo consiste en que la protección del archivo sensible falla,  aunque tengo todo el código del bloqueo bien puesto, el servidor se lo salta.

- **1 fallo relacionado con la parte Nginx (Fase 2: TLS, comunicación con Apache, o rate limiting).**
    - El segundo fallo tiene que ver con cómo Nginx hace de intermediario, el tráfico llega y se cifra bien, pero al pasarlo hacia atrás la web se rompe.



| **Síntoma observado** | **Hipótesis inicial** | **Comando de diagnóstico** | **Causa** | **Solución** | 
| --- | --- | --- | --- | --- |
| Intento entrar al archivo `claves.txt` desde terminal y me deja leerlo perfectamente (me devuelve un 200 OK), cuando en realidad debería darme un `error 403 Forbidden` porque le puse un archivo .htaccess configurado para bloquearlo. | Al principio pensé que había escrito mal el código dentro del .htaccess o que lo había creado en la carpeta equivocada y no en la raíz del VirtualHost. | Tiré un `curl -I -k -H "Host: nimbusmotos.com" https://localhost/claves.txt` y vi que devolvía un 200, entré al contenedor `docker compose exec apache bash` y comprobé el archivo de configuración de los virtualhost que yo he configurado manualmente con: `cat /etc/apache2/sites-enabled/nimbusmotos.conf`. | El fallo era que en el archivo de mi VirtualHost, dentro de la etiqueta `<Directory>`, no tenía puesto `AllowOverride All` entonces lo que hace Apache es bloquearlo por defecto. Apache pasará del archivo .htaccess y no aplicará las reglas de seguridad.| Editar el archivo de configuración de mi virtualhost, cambiar esa directiva a `AllowOverride All` para autorizar la lectura de los .htaccess, y reiniciamos el contenedor de Apache. |
| Al entrar a `nimbuscoches.com` o `nimbusmotos.com` (con `curl -k -H "Host: nimbuscoches.com" https://localhost/`), en vez de cargarme la web correcta de cada marca, me salta el VirtualHost por defecto de Apache (un error 403), como si ninguna de mis dos webs existiera. | Creía que había escrito mal los nombres en el ServerName de los archivos .conf de Apache, o que Apache no estaba leyendo la carpeta de los VirtualHosts en el volumen de Docker. | Miré qué le estaba llegando exactamente a Apache leyendo sus logs en tiempo real con: `docker compose logs apache -f` y ahí me di cuenta de que las peticiones no llegaban con el dominio. | El problema era que Nginx estaba actuando como proxy, pero no le estaba "chivando" a Apache qué dominio pedía el usuario. Al hacer el `proxy_pass` Nginx reenviaba el tráfico a secas, así que Apache recibía la petición como si el destino fuera la IP interna de Docker o el nombre apache:80. Al no coincidir con `nimbusmotos.com`, Apache cargaba el sitio por defecto. | Añadir la directiva `proxy_set_header Host $host;` en el bloque `location /` de Nginx, le pasa intacto el nombre del dominio a Apache, y este ya sabe qué VirtualHost debe cargar. |
---