# examendocker
## Examen RECUPERACION M11-SAD UF4. Docker (alta disponibilidad)
Crear una imagen de Docker con un servidor de correo POP uw-imap y ejecutarla en una máquina de AWS. Comprobar el funcionamiento de la máquina.

Repositorio de las imágenes docker:  
https://hub.docker.com/r/agalilea/m11extraaitor

Ubicación del proyecto en GITHub:  
https://github.com/ClRDAN/examendocker2

Comando para arrancar el docker:
docker run --rm --name popserver --hostname popserver --net popnet -p 110:110 -p 995:995 -d agalilea/m11extraaitor:latest

uw-imap es un servidor tipo xinetd, al instalarlo se crean los archivos de configuración pero tenemos que modificarlos para activar el servicio. Para ello creamos los archivos ipop3 y pop3s, y configuramos la imagen para que sobreescriba estos archivos en /etc/xinetd.d/ al arrancar. También es necesario modificar PAM para permitir que los usuarios de correo se logueen, para ello automatizamos la copia del archivo pamimap en /etc/pam.d/imap al arrancar la imagen.  

Si queremos poder enviar mensajes tipo MIME, debemos crear certificados para cada usuario. Primero creamos una pareja de claves con  
`openssl genrsa -des3 -out perekey.pem 2048`  
Luego generamos una petición de certificado con:  
`openssl req -new -key perekey -out perecsr.pem`  
y finalmente generamos el certificado, para ello necesitamos un archivo de configuración "ca.conf" con las siguientes líneas:  
`basicConstraints = critical,CA:true
extendedKeyUsage = serverAuth,emailProtection`  
y ejecutamos el siguiente comando:  
`openssl x509 -CA cacrt.pem -CAkey cakey.pem -req -in perecsr.pem -days 365 -sha1 -extfile ca.conf -CAcreateserial \
 -out perecrt.pem`
No lo he visto por ningún lado, pero entiendo que estos certificados deben copiarse en /etc/pki/tls/certs en el servidor, y las claves privadas en /etc/pki/tls/private en los clientes.
Para arrancar el container en la máquina de AWS:  
`docker run --rm --name popserver --hostname popserver --network popnet -p 110:110 -p 995:995 -d agalilea/m11aitor:latest`
Previamente debemos haber creado la red popnet con   
`docker network create popnet`
Para que los clientes puedan acceder al servidor hace falta hacer dos cosas:
* redirigir los puertos 110 y 995 (pop3 y pop3s) entre el contenedor y la máquina en la que corre, por eso en el comando de arranque indicamos -p 110:110 y -p 995:995.
* abrir el firewall para que acepte conexiones a esos dos puertos. En AWS esto se hace en el menú "Security Groups", seleccionamos el "security group" al que pertenece nuestra máquina virtual y en el botón de "actions" elegimos "Edit inbound rules". Hay reglas predefinidas para POP3 y POP3s que abren el puerto correspondiente, las seleccionamos y nos aseguramos de que aceptan conexiones de todas partes poniendo como origen 0.0.0.0/0. Guardamos y salimos.

Ahora ya deberíamos tener un servidor de correo funcionando, para comprobar que todo funciona hay que hacer dos pruebas, recibir correo por pop3 y por pop3s:  
* POP3: desde la máquina de clase accedemos por telnet al servidor usando los comandos:  
  ```
   telnet 35.177.67.48 110
    >USER pere
    >PASS pere
    >STAT
    >NOOP
    >LIST       #Lista los correos almacenados en el servidor
    >RETR 1     #recupera el primer mensaje
    >QUIT
  ```
* POP3S: pendiente. Por algún motivo el puerto no se abre, aunque se supone que no hay que hacer nada para que se abra, porque el uw-imap es "plug and play".      Al instalar el programa se crean automáticamente dos certificados para conexión TLS en /etc/pki/tls/certs con el nombre y el formato necesarios (acabados en .pem y con la PK y el certificado almacenados en texto plano en su interior). Deben tener permisos 600. Si se prefiere se puede crear manualmente unos certificados nuevos. 
