TASK 3 - Deploy LAMP/Samba con Ansible (openSUSE Leap 16.0)

a) GitHub
   https://github.com/dazaAlbornozJhosias/ASO-ansible-g5
   (repositorio compartido con gcussi@yahoo.com)

b) Pasos para ejecutar el proyecto

Requisitos
 - Maquina Master con ansible-core y git
   (sudo zypper install ansible-core git).
 - Maquina destino openSUSE Leap 16.0 con sshd activo, un usuario con sudo
   y conectividad de red con el Master (ping).
 - En openSUSE, sudo pide la contrasena de root (no la del usuario): esa es
   la que se escribe en "BECOME password".

1. Obtener el proyecto y entrar a la carpeta:
     git clone https://github.com/dazaAlbornozJhosias/ASO-ansible-g5.git
     cd ASO-ansible-g5
   (o descomprimir el zip entregado y entrar a la carpeta que se crea)

2. Instalar la coleccion necesaria (community.general):
     ansible-galaxy collection install -r requirements.yml

3. Crear el inventario (sin usuario ni contrasena visibles en el archivo):
     cp inventory/hosts.example inventory/hosts.ini
     nano inventory/hosts.ini      # poner IP del destino y usuario

4. Configurar acceso por llave SSH (una sola vez):
     ssh-keygen -t ed25519
     ssh-copy-id USUARIO@IP_DESTINO
     ssh USUARIO@IP_DESTINO        # debe entrar sin pedir contrasena

5. Crear el archivo de secretos y cifrarlo con Ansible Vault:
     cp group_vars/destinos/vault.yml.example group_vars/destinos/vault.yml
     nano group_vars/destinos/vault.yml     # poner contrasenas reales
     ansible-vault encrypt group_vars/destinos/vault.yml

6. Probar conexion y sintaxis:
     ansible destinos -m ping --ask-become-pass --ask-vault-pass
     ansible-playbook site.yml --syntax-check --ask-vault-pass

7. Ejecutar todo el despliegue:
     ansible-playbook site.yml --ask-become-pass --ask-vault-pass

   Ejecutar solo un rol (tags): apache, mariadb, phpmyadmin, samba, lamp
     ansible-playbook site.yml --tags samba --ask-become-pass --ask-vault-pass

8. Verificar en el destino:
     systemctl is-active apache2 mariadb smb nmb
     curl -I http://localhost/phpMyAdmin/
     sudo testparm -s
     smbclient //localhost/Compartido -U labuser -c 'ls'
     sudo firewall-cmd --list-services
   Desde un navegador: http://IP_DESTINO/phpMyAdmin/
   (usuario appuser, contrasena definida en el vault).

9. Volver a ejecutar el paso 7: no debe haber cambios (idempotencia,
   el resumen final debe mostrar changed=0).

Notas
 - Todos los comandos se ejecutan dentro de la carpeta del proyecto, porque
   ansible.cfg y group_vars se leen desde ahi.
 - Si zypper falla por bloqueo de PackageKit, el playbook ya lo detiene y
   enmascara; si persiste: sudo systemctl mask --now packagekit
 - Si las maquinas usan DHCP, la IP del destino puede cambiar: actualizar
   ansible_host en inventory/hosts.ini.
 - Mejoras de rendimiento: pipelining y ControlPersist en ansible.cfg,
   gather_facts desactivado y tareas sin reinicios innecesarios (handlers).
