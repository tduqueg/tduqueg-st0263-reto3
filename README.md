# Proyecto de Configuración de WordPress con Balanceo y Datos Distribuidos en AWS

En este proyecto, se configurará un entorno de alta disponibilidad con balanceo de carga y almacenamiento distribuido en AWS utilizando cinco máquinas virtuales (VMs) con los siguientes roles:

- NFS: 18.208.103.163
- APP1: 52.6.98.238
- NGINX: 54.81.156.51
- DB: 3.227.195.79
- APP2: 54.237.135.137

## Paso 1: Configuración del Servidor de Base de Datos (DB)

1. Conéctate a la VM designada para la base de datos.

2. Actualiza e instala Docker:

   ```bash
   sudo apt update && sudo apt upgrade -y
   sudo apt install docker.io -y
   sudo systemctl enable docker && sudo systemctl start docker
   ```

Ejecuta un contenedor con MySQL:

```bash
docker run --name mysql-wp -e MYSQL_ROOT_PASSWORD=mi_contraseña -e MYSQL_DATABASE=wordpress -e MYSQL_USER=wp_user -e MYSQL_PASSWORD=wp_password -p 3306:3306 -d mysql:latest
```

## Paso 2: Configuración del Servidor NFS (FileServer)

Conéctate a la VM destinada para NFS.

Instala y configura el servidor NFS:

```bash
sudo apt-get update
sudo apt-get install nfs-kernel-server -y
```

Crea y configura un directorio para compartir:

```bash
sudo mkdir -p /var/nfs/general
sudo chown nobody:nogroup /var/nfs/general
Edita /etc/exports y agrega el siguiente contenido:
```

```bash
/var/nfs/general *(rw,sync,no_subtree_check)
```

Reinicia el servicio NFS:

```bash
sudo systemctl restart nfs-kernel-server
```

## Paso 3: Configuración de WordPress con Docker (2 VMs)

Conéctate a las VMs destinadas para WordPress.

Actualiza, instala Docker y monta el directorio NFS:

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install docker.io nfs-common -y
sudo mkdir -p /mnt/nfs/general
sudo mount ip_fileserver:/var/nfs/general /mnt/nfs/general
```

Ejecuta un contenedor WordPress en cada VM:

```bash
docker run -e WORDPRESS_DB_HOST=ip_dbserver -e WORDPRESS_DB_USER=wp_user -e WORDPRESS_DB_PASSWORD=wp_password -e WORDPRESS_DB_NAME=wordpress -v /mnt/nfs/general:/var/www/html/wp-content -p 80:80 -d wordpress:latest
```

## Paso 4: Configuración del Balanceador de Carga con Nginx

Conéctate a la VM destinada para Nginx.

Instala Nginx:

```bash
sudo apt update && sudo apt install nginx -y
```

Configura Nginx:

Edita /etc/nginx/sites-available/default y configura el balanceador:

```nginx
upstream wordpress_backend {
    server ip_wordpress1:80;
    server ip_wordpress2:80;
}

server {
    listen 80;

    location / {
        proxy_pass http://wordpress_backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

Reinicia Nginx:

```bash
sudo systemctl restart nginx
```

¡Has completado la configuración de tu entorno de alta disponibilidad de WordPress en AWS!
