Instalación y Migración de Jenkins con Docker
Esta guía describe los pasos para construir una imagen de Jenkins personalizada con Docker, realizar la configuración inicial, y luego empaquetar tanto la imagen como el volumen de datos para migrarla a otra máquina.

1. Construcción y Ejecución Inicial
Primero, construimos la imagen de Docker a partir de un Dockerfile en el directorio actual y creamos un volumen para la persistencia de los datos de Jenkins.

Construir la imagen:

docker build -t jenkins:dind .

Crear el volumen:

docker volume create jenkins_home

Ejecutar el contenedor:

docker run -d -p 8080:8080 -p 50000:50000 --name jenkins-server -v jenkins_home:/var/jenkins_home jenkins:dind

2. Configuración Inicial de Jenkins
Una vez que el contenedor esté en ejecución, necesitas la contraseña inicial del administrador para acceder a la interfaz web en http://localhost:8080.

Obtener la contraseña inicial:
Revisa los logs del contenedor para encontrar la contraseña generada automáticamente.

docker logs jenkins-server

Busca un resultado similar a este en la consola:

[LF]> Jenkins initial setup is required. An admin user has been created and a password generated.
[LF]> Please use the following password to proceed to installation:
[LF]>
[LF]> 86665f841e2f4297ab47758561671c82    <-- Esta es la contraseña
[LF]>
[LF]> This may also be found at: /var/jenkins_home/secrets/initialAdminPassword

Instalar Plugins:
Accede a la interfaz web y, cuando se te solicite, procede con la instalación de los plugins sugeridos. Completa el resto de la configuración inicial.

3. Empaquetar para Migración (Configuración Manual)
Después de configurar Jenkins, detenemos el contenedor y creamos una nueva imagen a partir de su estado actual. También exportamos la imagen y el volumen de datos.

Detener el contenedor:

docker stop jenkins-server

Crear una nueva imagen a partir del contenedor detenido:

docker commit jenkins-server jenkins:final

Guardar la imagen en un archivo .tar:

docker save -o jenkins.tar jenkins:final

Hacer un backup del volumen (ejecutar en CMD en Windows):
Este comando crea un archivo jenkins-volumen.tar en el directorio actual con el contenido del volumen jenkins_home.

docker run --rm -v jenkins_home:/data -v %cd%:/backup alpine tar cvf /backup/jenkins-volumen.tar -C /data .

Nota: Para Linux o macOS, reemplaza %cd% por $(pwd).

4. Restaurar en otra Máquina
En la máquina de destino, carga la imagen de Docker, crea un nuevo volumen y restaura los datos del backup.

Cargar la imagen desde el archivo .tar:

docker load -i jenkins.tar

Crear un nuevo volumen:

docker volume create jenkins_home_nuevo

Restaurar los datos del volumen (ejecutar en CMD en Windows):
(Este paso se omite en tus instrucciones originales, pero sería necesario para restaurar los datos. Si el objetivo es solo usar la nueva imagen con un volumen vacío, ignora este paso).

Ejecutar el nuevo contenedor con la imagen y el volumen restaurados:

docker run -d -p 8080:8080 -p 50000:50000 --name jenkins-final-server -v jenkins_home_nuevo:/var/jenkins_home jenkins:final

Ahora deberías tener una instancia de Jenkins idéntica a la original corriendo en la nueva máquina.