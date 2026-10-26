
README - PROYECTO ANSIBLE

1. ENLACE AL REPOSITORIO DE GITHUB

Repositorio del proyecto:
https://github.com/dazaAlbornozJhosias/ASO-ansible-g5

2. REQUISITOS PREVIOS

Antes de ejecutar el proyecto, se necesita lo siguiente:

2.1. Máquinas virtuales

Se deben utilizar dos máquinas virtuales con openSUSE Leap 16.0.

VM 01 - Máquina maestra:
- Tiene Ansible instalado en caso de no tenerlo instalado seguir ir al punto 2.4.
- Contiene el código fuente del proyecto.
- Desde esta máquina se ejecutan los playbooks.

VM 02 - Servidor destino:
- Es la máquina donde se instalarán y configurarán los servicios.
- Debe estar encendida y conectada a la misma red que la VM 01.

2.2. Requisitos de red

Ambas máquinas deben poder comunicarse entre sí.

Se necesita conocer la dirección IP de la VM 02. Para consultar la dirección IP, abrir una terminal en la VM 02 y ejecutar:

    hostname -I

Anotar la dirección IPv4 que corresponda a la red utilizada por ambas máquinas.

En la VM 01, comprobar la conectividad con el siguiente comando, sustituyendo la dirección por la IP real de la VM 02:

    ping -c 4 192.168.1.11

Si la conexión funciona, se recibirán respuestas desde el servidor destino.

Si no hay respuesta, comprobar que ambas máquinas estén encendidas, conectadas a la red correcta y que la comunicación no esté bloqueada por el firewall.

Nota: La IP 192.168.1.11 es un ejemplo. Se debe utilizar la dirección real de la VM 02.

2.3. Requisitos de acceso

Se necesita:
- Un usuario con acceso SSH al servidor destino.
- La contraseña de ese usuario.
- La contraseña de sudo o de escalación de privilegios, si es diferente.
- La contraseña de Ansible Vault utilizada para proteger las variables sensibles del proyecto.
  contraseña:2468
2.4. Instalar Ansible y las herramientas necesarias en la VM 01

 Si la VM 01 todavía no tiene Ansible instalado, seguir estos pasos.
 Paso 1. Actualizar la información de los repositorios

 Abrir una terminal en la VM 01 y ejecutar:
 sudo zypper refresh
 Introducir la contraseña del usuario cuando se solicite.
 Paso 2. Instalar Ansible, Python 3, SSH y Git

 Ejecutar:
 sudo zypper install ansible python3 openssh-clients git
 Confirmar la instalación cuando el sistema lo solicite.
 Estos paquetes cumplen las siguientes funciones:
 ansible: permite ejecutar los playbooks.
 python3: proporciona Python 3, necesario para Ansible y para ejecutar sus módulos en el servidor administrado.
 openssh-clients: proporciona el cliente SSH para conectarse al servidor destino.
 git: permite descargar o actualizar el proyecto desde GitHub.
 Si el paquete ansible no está disponible en los repositorios configurados, comprobar si se ofrece el paquete ansible-core:
 sudo zypper search ansible
 Si aparece ansible-core como paquete disponible, se puede instalar con:
 sudo zypper install ansible-core
 Paso 3. Verificar las instalaciones
 Ejecutar estos comandos:
 ansible --version
 python3 --version
 ssh -V
 git --version

3. PREPARACIÓN DEL PROYECTO

3.1. Abrir una terminal en la VM 01.

3.2. Acceder al directorio del proyecto.

Por ejemplo, si el proyecto está en el directorio personal del usuario:

    cd ~/ASO-ansible-g5-main

Si el proyecto se encuentra en otra ubicación, utilizar la ruta correspondiente.

3.3. Verificar que se encuentran los archivos principales.

El proyecto debe contener, entre otros, los siguientes elementos:

- site.yml: playbook principal que organiza la ejecución de los servicios.
- inventory/hosts.ini: archivo que identifica los servidores administrados.
- roles/: directorio que contiene las tareas y configuraciones de cada servicio.
- group_vars/: directorio donde se encuentran variables utilizadas por el proyecto.

No modificar ni eliminar archivos del proyecto sin conocer su función.

4. CONFIGURACIÓN DEL INVENTARIO

El inventario indica a Ansible a qué servidor debe conectarse.

Abrir el archivo:

    nano inventory/hosts.ini

Verificar que la dirección IP corresponda a la VM 02.

El inventario utilizado en este proyecto tiene la siguiente estructura:

    [destinos]
    lamp-server ansible_host=192.168.1.11

    [destinos:vars]
    ansible_become=true

Si la dirección IP del servidor destino es diferente, reemplazar 192.168.1.11 por la IP correcta.

Para guardar los cambios en nano:
1. Presionar Ctrl + O.
2. Presionar Enter.
3. Presionar Ctrl + X.

5. VERIFICAR LA CONEXIÓN SSH

Antes de ejecutar el playbook, comprobar que la VM 01 pueda conectarse al servidor destino.

Ejecutar:

    ssh elmer@192.168.1.11 //ejemplo

Sustituir la IP si corresponde.

La primera vez puede aparecer una pregunta para confirmar la identidad del servidor. Verificar que se trata de la VM destino antes de aceptar.

Después, introducir la contraseña del usuario remoto.

Si la conexión funciona, se abrirá una terminal del servidor destino. Para salir y regresar a la VM 01, ejecutar:

    exit

Si la conexión falla, revisar la IP, el usuario, la contraseña, el estado del servicio SSH y la conectividad de red.

6. COMPROBAR LA SINTAXIS DEL PLAYBOOK

Antes de realizar cambios en el servidor, comprobar que Ansible puede interpretar el playbook.

Desde el directorio principal del proyecto, ejecutar:

    ansible-playbook -i inventory/hosts.ini site.yml --syntax-check

Si la sintaxis es correcta, se mostrará:

    playbook: site.yml

Esta comprobación valida la sintaxis del playbook, pero no garantiza por sí sola que todas las tareas puedan ejecutarse correctamente.

7. EJECUTAR EL PROYECTO ANSIBLE

Desde la VM 01, dentro del directorio del proyecto, ejecutar:

    ansible-playbook -i inventory/hosts.ini site.yml -u elmer --ask-pass --ask-become-pass --ask-vault-pass

El comando utiliza los siguientes parámetros:

- -i inventory/hosts.ini: indica el archivo de inventario.
- site.yml: identifica el playbook principal.
- -u elmer: indica el usuario utilizado para conectarse al servidor destino.
- --ask-pass: solicita la contraseña SSH.
- --ask-become-pass: solicita la contraseña para obtener privilegios administrativos.
- --ask-vault-pass: solicita la contraseña para descifrar las variables protegidas con Ansible Vault.

Durante la ejecución aparecerán solicitudes de contraseña:

SSH password:
Introducir la contraseña del usuario remoto.

BECOME password:
Introducir la contraseña de sudo si se solicita.

Vault password:
Introducir la contraseña de Ansible Vault del proyecto:2468.

Las contraseñas no se muestran mientras se escriben. Esto es normal.

Esperar hasta que termine la ejecución. No cerrar la terminal mientras el playbook esté trabajando.

8. INTERPRETAR EL RESULTADO

Al finalizar, Ansible muestra un resumen llamado PLAY RECAP.

Un ejemplo de resultado correcto es:

    PLAY RECAP
    lamp-server : ok=44 changed=5 unreachable=0 failed=0 skipped=5

Significado de los valores:

- ok: tareas ejecutadas correctamente o cuyo estado ya era correcto.
- changed: tareas que realizaron cambios en el servidor.
- unreachable: equipos a los que Ansible no pudo conectarse.
- failed: tareas que terminaron con errores.
- skipped: tareas que se omitieron porque no correspondían a las condiciones configuradas.

Para esta práctica, el resultado obtenido fue:

    ok=44
    changed=5
    unreachable=0
    failed=0
    skipped=5

El valor failed=0 indica que ninguna tarea falló durante esa ejecución. El valor unreachable=0 indica que no hubo errores de conexión reportados por Ansible.
