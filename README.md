# Jenkins GitHub CI/CD con Apache

Este proyecto configura un pipeline básico de integración y despliegue continuo (CI/CD) utilizando Jenkins y GitHub para desplegar automáticamente cambios en un servidor Apache mediante SSH. Es ideal para aprender a integrar Jenkins con un repositorio GitHub y realizar despliegues en un servidor web.

## Descripción

Este proyecto automatiza el proceso de despliegue de archivos desde GitHub a un servidor Apache:
1. Detecta cambios en un repositorio de GitHub.
2. Usa Jenkins para conectarse a un servidor remoto (Apache) y aplicar automáticamente los cambios mediante SSH.

## Requisitos

- Dos máquinas virtuales o servidores (VM1 con Jenkins, y VM2 con Apache).
- Acceso a una cuenta de GitHub.
- Jenkins y Git instalados en VM1.
- Servidor Apache y OpenSSH instalados en VM2.

## Configuración del Proyecto

### 1. Preparar el Entorno

#### Configuración en VM1 (Jenkins)
1. **Instalar Jenkins**:
   - Instala Jenkins y asegúrate de que está ejecutándose (`sudo systemctl start jenkins`).
   - Accede a Jenkins en `http://localhost:8080` (o mediante la URL de Ngrok si estás exponiendo Jenkins al exterior).
   - Crea un usuario administrador y completa la configuración inicial.

2. **Instalar Git**:
   - Instala Git en VM1 con `sudo apt install git`.

#### Configuración en VM2 (Apache)
1. **Instalar Apache y OpenSSH**:
   - Instala Apache: `sudo apt install apache2`.
   - Instala OpenSSH: `sudo apt install openssh-server`.

### 2. Configurar la Autenticación SSH

1. **Generar una Clave SSH en VM1**:
   - Ejecuta `ssh-keygen -t rsa -b 4096 -C "your_email@example.com"` y sigue las instrucciones para crear una clave SSH.
   - Usa `ssh-copy-id user@VM2_IP` para copiar la clave pública a VM2, permitiendo conexiones SSH automáticas.

2. **Verificar la Conexión SSH**:
   - Desde VM1, prueba la conexión a VM2: `ssh user@VM2_IP`.
   - Confirma que puedes conectarte sin necesidad de ingresar una contraseña.

### 3. Configurar Jenkins para Conexión SSH con VM2

1. **Agregar Credenciales en Jenkins**:
   - En Jenkins, navega a "Manage Jenkins" -> "Manage Credentials".
   - Selecciona el almacén de credenciales adecuado (generalmente "global") y agrega una nueva credencial SSH con la clave privada generada.

2. **Configurar un Agente en Jenkins**:
   - Navega a "Manage Jenkins" -> "Manage Nodes and Clouds" -> "New Node".
   - Crea un nuevo nodo para VM2 y usa las credenciales SSH configuradas.
   - Define el directorio remoto en VM2 donde Jenkins ejecutará los trabajos, por ejemplo, `/home/user/jenkins-agent`.

### 4. Configurar GitHub y Webhooks

1. **Crear un Repositorio en GitHub**:
   - Crea un nuevo repositorio en GitHub que contendrá los archivos a desplegar (por ejemplo, `index.html`).

2. **Agregar el Webhook en GitHub**:
   - Navega a la configuración del repositorio y selecciona "Webhooks".
   - Haz clic en "Add webhook" y usa la URL pública de Jenkins con la ruta `/github-webhook/` (por ejemplo, `http://<ngrok-url>/github-webhook/`).
   - Configura el evento para `push` y guarda el webhook.

### 5. Crear el Pipeline en Jenkins

1. **Crear un Jenkinsfile en el Repositorio de GitHub**:
   - En el repositorio de GitHub, crea un archivo llamado `Jenkinsfile` con el siguiente contenido de ejemplo:

   ```groovy
   pipeline {
       agent any
       stages {
           stage('Clonar Repositorio') {
               steps {
                   git 'https://github.com/tu-usuario/tu-repositorio.git'
               }
           }
           stage('Desplegar en Apache') {
               steps {
                   sshagent(['nombre-de-credencial']) {
                       sh 'scp -o StrictHostKeyChecking=no index.html user@VM2_IP:/var/www/html/'
                   }
               }
           }
       }
   }

- Reemplaza nombre-de-credencial con el ID de la credencial SSH configurada en Jenkins, user con el usuario de VM2, y VM2_IP con la IP de VM2.
2. **Configurar el Proyecto en Jenkins**:
- En Jenkins, crea un nuevo proyecto "Pipeline".
- Selecciona la opción para definir el pipeline mediante un Jenkinsfile.
- Apunta a tu repositorio de GitHub para que Jenkins use el Jenkinsfile allí almacenado.

### 6. Ejecutar y Verificar el Despliegue

1. **Realizar un Cambio en GitHub**:
- Realiza un commit y push de una modificación en index.html en el repositorio de GitHub.
- Esto activará el webhook, iniciando el pipeline en Jenkins.

2. **Verificar el Despliegue en Apache**:
- Una vez que el pipeline finalice, verifica el despliegue visitando la IP de VM2 en tu navegador (http://VM2_IP/index.html).
