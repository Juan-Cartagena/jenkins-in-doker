Claro, aquí tienes el archivo `README.md` con las instrucciones que solicitaste.

-----

# Configuración y Migración de Jenkins con Docker

Este documento detalla el proceso para crear, configurar y migrar un servidor Jenkins utilizando contenedores de Docker. El proceso se divide en cuatro fases principales:

1.  **Construcción y Ejecución Inicial:** Crear la imagen y el contenedor de Jenkins.
2.  **Configuración Inicial:** Obtener la contraseña y configurar los plugins.
3.  **Backup (Máquina Origen):** Crear una imagen con la configuración y respaldar el volumen de datos.
4.  **Restauración (Máquina Destino):** Cargar la imagen y restaurar los datos en un nuevo servidor.

-----

## 1\. Construcción y Ejecución Inicial ⚙️

Estos pasos se realizan en la máquina original para levantar el servidor Jenkins por primera vez.

1.  **Construir la imagen de Docker.**
    Esto crea una imagen personalizada llamada `jenkins:dind` a partir del `Dockerfile` en el directorio actual.

    ```bash
    docker build -t jenkins:dind .
    ```

2.  **Crear un volumen de Docker.**
    El volumen `jenkins_home` persistirá los datos de Jenkins (configuraciones, jobs, etc.) fuera del contenedor.

    ```bash
    docker volume create jenkins_home
    ```

3.  **Ejecutar el contenedor de Jenkins.**
    Se inicia el servidor Jenkins, mapeando los puertos y montando el volumen que creamos.

    ```bash
    docker run -d -p 8080:8080 -p 50000:50000 --name jenkins-server -v jenkins_home:/var/jenkins_home jenkins:dind
    ```

-----

## 2\. Configuración Inicial de Jenkins 🔑

Una vez que el contenedor está en ejecución, necesitas completar el asistente de instalación.

1.  **Obtener la contraseña inicial de administrador.**
    Ejecuta el siguiente comando para ver los logs del contenedor. La contraseña aparecerá en la salida.
    ```bash
    docker logs jenkins-server
    ```
    Busca una salida similar a esta y copia la contraseña:
    ```log
    *************************************************************
    *************************************************************
    *************************************************************

    Jenkins initial setup is required. An admin user has been created and a password generated.
    Please use the following password to proceed to installation:

    86665f841e2f4297ab47758561671c82  <-- Esta es la contraseña

    This may also be found at: /var/jenkins_home/secrets/initialAdminPassword

    *************************************************************
    *************************************************************
    *************************************************************
    ```
    ![Ejemplo](/docker-in-docker/images/configuracion-inicial-jenkins.png)
2.  **Completar la instalación.**
      * Abre tu navegador y ve a `http://localhost:8080`.
      * Pega la contraseña obtenida.
      * Selecciona **"Instalar plugins sugeridos"** y espera a que el proceso termine.
      * Crea tu usuario administrador.

-----

## 3\. Backup para la Migración (Máquina Origen) 📦

Después de configurar Jenkins, sigue estos pasos para empaquetar tanto la configuración (imagen) como los datos (volumen) para la migración.

1.  **Detener el servidor Jenkins.**

    ```bash
    docker stop jenkins-server
    ```

2.  **Crear una nueva imagen a partir del contenedor configurado.**
    Esto "congela" el estado del contenedor con los plugins instalados en una nueva imagen llamada `jenkins:final`.

    ```bash
    docker commit jenkins-server jenkins:final
    ```

3.  **Guardar la nueva imagen en un archivo `.tar`.**
    Este archivo contendrá la imagen `jenkins:final` para poder moverla a otra máquina.

    ```bash
    docker save -o jenkins.tar jenkins:final
    ```

4.  **Respaldar el volumen de datos.**
    Este comando comprime todo el contenido del volumen `jenkins_home` en un archivo `jenkins-volumen.tar` en tu directorio actual.

      * **Para Windows (CMD):**
        ```cmd
        docker run --rm -v jenkins_home:/data -v %cd%:/backup alpine tar cvf /backup/jenkins-volumen.tar -C /data .
        ```
      * **Para Linux, macOS o PowerShell:**
        ```bash
        docker run --rm -v jenkins_home:/data -v $(pwd):/backup alpine tar cvf /backup/jenkins-volumen.tar -C /data .
        ```

Al finalizar, tendrás dos archivos para mover a la nueva máquina: `jenkins.tar` y `jenkins-volumen.tar`.

-----

## 4\. Restauración en la Máquina Destino 🚚

En la nueva máquina, sigue estos pasos para desplegar el servidor Jenkins migrado. Asegúrate de haber copiado los archivos `jenkins.tar` y `jenkins-volumen.tar` a esta máquina.

1.  **Cargar la imagen de Docker desde el archivo `.tar`.**

    ```bash
    docker load -i jenkins.tar
    ```

2.  **Crear un nuevo volumen.**

    ```bash
    docker volume create jenkins_home_nuevo
    ```

3.  **Restaurar los datos en el nuevo volumen.**
    Este comando descomprime el respaldo `jenkins-volumen.tar` dentro del nuevo volumen `jenkins_home_nuevo`.

      * **Para Windows (CMD):**
        ```cmd
        docker run --rm -v jenkins_home_nuevo:/data -v %cd%:/backup alpine tar xvf /backup/jenkins-volumen.tar -C /data
        ```
      * **Para Linux, macOS o PowerShell:**
        ```bash
        docker run --rm -v jenkins_home_nuevo:/data -v $(pwd):/backup alpine tar xvf /backup/jenkins-volumen.tar -C /data
        ```

4.  **Ejecutar el nuevo contenedor de Jenkins.**
    Este comando inicia el servidor Jenkins utilizando la imagen y el volumen restaurados.

    ```bash
    docker run -d -p 8080:8080 -p 50000:50000 --name jenkins-final-server -v jenkins_home_nuevo:/var/jenkins_home jenkins:final
    ```

¡Listo\! Tu servidor Jenkins migrado debería estar funcionando en `http://localhost:8080` con toda su configuración y datos anteriores.