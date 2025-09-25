# Dockerfile

# Usamos la imagen oficial de Jenkins como base
FROM jenkins/jenkins:lts-jdk21

# Argumento para el Group ID (GID) del docker socket del host
ARG DOCKER_GID=999

# Cambiamos a usuario root para instalar paquetes
USER root

# ------------------------------------
# --- Instalación de Docker CLI ---
# ------------------------------------
# Actualizamos e instalamos dependencias
RUN apt-get update && apt-get install -y lsb-release gnupg

# Añadimos la llave GPG oficial de Docker
RUN curl -fsSL https://download.docker.com/linux/debian/gpg | gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

# Configuramos el repositorio de Docker
RUN echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/debian \
  $(lsb_release -cs) stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null

# Instalamos el cliente de Docker (CLI)
RUN apt-get update && apt-get install -y docker-ce-cli

# ------------------------------------
# --- Configuración de Permisos ---
# ------------------------------------
# Creamos un grupo 'docker' con el GID del host para que jenkins pueda usar el socket
RUN groupadd -g ${DOCKER_GID} docker && usermod -aG docker jenkins

# ------------------------------------
# --- Instalación de Plugins ---
# ------------------------------------
# Copiamos la lista de plugins
COPY plugins.txt /usr/share/jenkins/ref/plugins.txt

# Ejecutamos el script de instalación
RUN jenkins-plugin-cli -f /usr/share/jenkins/ref/plugins.txt

# Regresamos al usuario jenkins por seguridad
USER jenkins