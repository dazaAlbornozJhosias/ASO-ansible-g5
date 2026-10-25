TASK 3 - Deploy LAMP/Samba con Ansible (openSUSE Leap 16.0)

a) GitHub
   https://github.com/dazaAlbornozJhosias/ASO-ansible-g5
   (repositorio compartido con gcussi@yahoo.com)

b) Pasos para ejecutar el proyecto

Requisitos
 - Maquina Master con ansible-core instalado (sudo zypper install ansible-core).
 - Maquina destino openSUSE Leap 16.0 con sshd activo, un usuario con sudo
   y conectividad de red con el Master (ping).

1. Entrar a la carpeta del proyecto:
     cd ASO-ansible-g5

2. Crear el inventario (sin usuario ni contrasena visibles en el archivo):
     cp inventory/hosts.example inventory/hosts.ini
     nano inventory/hosts.ini      # poner IP del destino y usuario

3. Configurar acceso por llave SSH (una sola vez):
     ssh-keygen -t ed25519
     ssh-copy-id USUARIO@IP_DESTINO
     ssh USUARIO@IP_DESTINO        # debe entrar sin pedir contrasena

4. Crear el archivo de secretos y cifrarlo con Ansible Vault:
     cp group_vars/destinos/vault.yml.example group_vars/destinos/vault.yml
     nano group_vars/destinos/vault.yml     # poner contrasenas reales
     ansible-vault encrypt group_vars/destinos/vault.yml

5. Probar conexion y sintaxis:
     ansible destinos -m ping
     ansible-playbook site.yml --syntax-check --ask-vault-pass

6. Ejecutar todo el despliegue:
     ansible-playbook site.yml --ask-vault-pass --ask-become-pass

   Ejecutar solo un rol (tags): apache, mariadb, phpmyadmin, samba, lamp
     ansible-playbook site.yml --tags samba --ask-vault-pass --ask-become-pass

7. Verificar en el destino:
     systemctl is-active apache2 mariadb smb nmb
     curl -I http://localhost/phpMyAdmin/
     sudo testparm -s
     sudo smbclient //localhost/Compartido -U labuser -c 'ls'

8. Volver a ejecutar el paso 6: no debe haber cambios (idempotencia).

Notas
 - Si zypper falla por bloqueo de PackageKit, el playbook ya lo detiene y
   enmascara; si persiste: sudo systemctl mask --now packagekit
 - Mejoras de rendimiento: pipelining y ControlPersist en ansible.cfg,
   gather_facts desactivado y modulo de paquetes fijado a zypper.
