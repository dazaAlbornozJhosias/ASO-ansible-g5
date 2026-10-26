TASKS 3, 4 y 5 - Deploy LAMP/Samba/FTP + PostgreSQL con Ansible (openSUSE Leap 16.0)

a) GitHub
   https://github.com/dazaAlbornozJhosias/ASO-ansible-g5
   Rama con el codigo entregado: repuesto-JD
   https://github.com/dazaAlbornozJhosias/ASO-ansible-g5/tree/repuesto-JD
   (repositorio compartido con gcussi@yahoo.com)

b) Pasos para ejecutar el proyecto

Contenido
 - Task 3: roles apache_php, mariadb, phpmyadmin, samba y vsftpd.
 - Task 4: scripts de Ansible y plantillas (smb.conf.j2, vsftpd.conf.j2,
   phpMyAdmin.conf.j2, ansible.cnf.j2) versionados en GitHub.
 - Task 5: rol postgresql (instalacion, php8-pgsql, contrasena de postgres,
   usuario y base nuevos, conexion remota cifrada con SSL).

Requisitos
 - Maquina Master con ansible-core y git
   (sudo zypper install ansible-core git).
 - Maquina destino openSUSE Leap 16.0 con sshd activo, un usuario con sudo
   y conectividad de red con el Master (ping).
 - En openSUSE, sudo pide la contrasena de root (no la del usuario): esa es
   la que se escribe en "BECOME password".

1. Obtener el proyecto y entrar a la carpeta:
     git clone -b repuesto-JD https://github.com/dazaAlbornozJhosias/ASO-ansible-g5.git
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
   Variables: vault_samba_password, vault_mariadb_root_password,
   vault_mariadb_app_password, vault_ftp_password (usuario ftpuser),
   vault_postgres_password (usuario postgres) y vault_pg_user_password
   (usuario pguser, base pgdb).

6. Probar conexion y sintaxis:
     ansible destinos -m ping --ask-become-pass --ask-vault-pass
     ansible-playbook site.yml --syntax-check --ask-vault-pass

7. Ejecutar todo el despliegue:
     ansible-playbook site.yml --ask-become-pass --ask-vault-pass

   Ejecutar solo un rol (tags): apache, mariadb, phpmyadmin, samba, vsftpd,
   postgresql, lamp
     ansible-playbook site.yml --tags vsftpd --ask-become-pass --ask-vault-pass

8. Verificar en el destino:
     systemctl is-active apache2 mariadb smb nmb vsftpd postgresql
     curl -I http://localhost/phpMyAdmin/
     sudo testparm -s
     smbclient //localhost/Compartido -U labuser -c 'ls'
     sudo firewall-cmd --list-services
   Desde un navegador: http://IP_DESTINO/phpMyAdmin/
   (usuario appuser, contrasena definida en el vault).

   Verificar FTP desde el Master (usuario ftpuser, contrasena del vault):
     echo prueba > /tmp/p.txt
     curl -T /tmp/p.txt --user ftpuser ftp://IP_DESTINO/
     curl --user ftpuser ftp://IP_DESTINO/        # debe listar p.txt

   Verificar PostgreSQL desde el Master (requiere el cliente:
   sudo zypper install postgresql):
     psql "host=IP_DESTINO dbname=pgdb user=pguser sslmode=require" -c "SELECT version();"
     psql "host=IP_DESTINO dbname=pgdb user=pguser sslmode=disable" -c "SELECT 1;"
     psql "host=IP_DESTINO dbname=postgres user=postgres sslmode=require" -c "SELECT ssl, version FROM pg_stat_ssl WHERE pid = pg_backend_pid();"
   La primera debe mostrar la version, la segunda debe ser rechazada por
   pg_hba.conf (sin cifrado) y la tercera debe mostrar ssl = t.

9. Volver a ejecutar el paso 7: no debe haber cambios (idempotencia,
   el resumen final debe mostrar changed=0).

Notas
 - Todos los comandos se ejecutan dentro de la carpeta del proyecto, porque
   ansible.cfg y group_vars se leen desde ahi.
 - Si zypper falla por bloqueo de PackageKit, el playbook ya lo detiene y
   enmascara; si persiste: sudo systemctl mask --now packagekit
 - Si las maquinas usan DHCP, la IP del destino puede cambiar: actualizar
   ansible_host en inventory/hosts.ini.
 - La red permitida para PostgreSQL (192.168.0.0/24) y los nombres de
   usuario y base estan en roles/postgresql/defaults/main.yml; ajustarlos
   si la red del destino es otra.
 - Mejoras de rendimiento: pipelining y ControlPersist en ansible.cfg,
   gather_facts desactivado y tareas sin reinicios innecesarios (handlers).
