# docker-mysql
## 🐳 1. Crear la estructura de archivos
En tu VPS, crear directorio:
bash
```bash
mkdir ~/mysql-docker && cd ~/mysql-docker
```
## 📁 2. Crear docker-compose.yml

📄 docker-compose.yml
yaml


```
version: '3.8'

services:
  mysql:
    image: mysql:8.0
    container_name: mysql-produccion
    environment:
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
      MYSQL_DATABASE: ${MYSQL_DATABASE}
      MYSQL_USER: ${MYSQL_USER}
      MYSQL_PASSWORD: ${MYSQL_PASSWORD}
    ports:
      - "3306:3306"
    volumes:
      - mysql_data:/var/lib/mysql
      - ./config:/etc/mysql/conf.d
    restart: unless-stopped
    command: 
      - --bind-address=0.0.0.0
      - --max_connections=1000

volumes:
  mysql_data:

``` 
##  🔐 3. Crear archivo de variables de entorno
📄 .env
env
## Contraseñas - CAMBIA ESTOS VALORES
```
MYSQL_ROOT_PASSWORD=TuPasswordRootSuperSeguro123!
MYSQL_DATABASE=mi_aplicacion
MYSQL_USER=usuario_app
MYSQL_PASSWORD=PasswordUsuarioApp456!
```
⚠️ IMPORTANTE: Cambia las contraseñas por valores seguros.

## ⚙️ 4. (Opcional) Configuración personalizada de MySQL
Crear directorio de configuración:
bash
```bash
mkdir config
```
📄 config/custom.cnf
ini
```
[mysqld]
bind-address = 0.0.0.0
max_connections = 1000
innodb_buffer_pool_size = 256M
query_cache_size = 64M
slow_query_log = 1

[client]
default-character-set = utf8mb4

[mysql]
default-character-set = utf8mb4
```
## 🚀 5. Desplegar MySQL
Ejecutar en el VPS:
bash
### Navegar al directorio
```bash
cd ~/mysql-docker
```

### Levantar el contenedor
docker-compose up -d

### Verificar que esté funcionando
```bash
docker-compose ps
```
Ver logs para confirmar:
bash
```bash
docker-compose logs -f mysql
```
## 🔧 6. Comandos útiles para gestión
Conectar a MySQL:
bash
### Desde el VPS
```bash
docker exec -it mysql-produccion mysql -u root -p
```

### Desde tu máquina local (si expusiste el puerto)
```bash
mysql -h TU_VPS_IP -P 3306 -u root -p
```
Gestionar el contenedor:
bash
### Ver estado
```bash
docker-compose ps
```

### Detener MySQL
```bash
docker-compose down
```

### Reiniciar
```bash
docker-compose restart
```

### Ver logs
```bash
docker-compose logs -f mysql
```
Backup de datos:
bash
### Backup manual
```bash
docker exec mysql-produccion mysqldump -u root -p mi_aplicacion > backup.sql
```

### Ver información del volumen
```bash
docker volume ls
docker volume inspect mysql-docker_mysql_data
```
# 🌐 7. Configurar firewall (si es necesario)
bash
## Permitir conexiones MySQL
```bash
sudo ufw allow 3306/tcp
sudo ufw reload
```
# 📊 8. Verificar que todo funciona
Test de conexión desde tu máquina local:
bash
```bash
mysql -h TU_VPS_IP -P 3306 -u root -p
```
* Dentro de MySQL:
sql
```
SHOW DATABASES;
USE mi_aplicacion;
SHOW TABLES;
SELECT User, Host FROM mysql.user;
```

🗂️ Estructura final del proyecto:
text

```
~/mysql-docker/
├── docker-compose.yml
├── .env
├── config/
│   └── custom.cnf
└── (mysql_data volume creado automáticamente)
```

 ⚡ Comando rápido (si quieres probar rápido):
Si quieres una versión mínima sin archivos de configuración:

bash
## Crear directorio y archivo rápido
```bash
mkdir ~/mysql-simple && cd ~/mysql-simple
```
```bash
cat > docker-compose.yml << EOF
version: '3.8'
services:
  mysql:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: password123
      MYSQL_DATABASE: testdb
    ports:
      - "3306:3306"
    volumes:
      - mysql_data:/var/lib/mysql
    restart: unless-stopped
volumes:
  mysql_data:
EOF

docker-compose up -d
```
¡Y listo! Tendrás MySQL funcionando en tu VPS con persistencia de datos mediante volúmenes Docker. Los datos sobrevivirán a reinicios y actualizaciones del contenedor.

¿Necesitas que ajuste alguna parte de la configuración?

# acceder exteriormente.
Para acceder exteriormente a MySQL en tu VPS, necesitas configurar varios aspectos. Te explico paso a paso:

## 🔧 1. Verificar que MySQL acepta conexiones externas
En tu docker-compose.yml actual:
yaml
```bash
services:
  mysql:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
    ports:
      - "3306:3306"  # ← Esto EXPONE el puerto
    volumes:
      - mysql_data:/var/lib/mysql
    restart: unless-stopped
    command: 
      - --bind-address=0.0.0.0  # ← Esto PERMITE conexiones externas
```
## 🌐 2. Configurar el Firewall del VPS
Abrir el puerto 3306:
bash
### Permitir conexiones MySQL desde cualquier IP
```bash
sudo ufw allow 3306/tcp
```

### O solo desde tu IP específica (MÁS SEGURO)
```bash
sudo ufw allow from TU_IP_PERSONAL to any port 3306
```

### Recargar el firewall
```bash
sudo ufw reload
```

### Verificar que el puerto esté abierto
```bash
sudo ufw status
```
## 🔍 3. Obtener la IP de tu VPS
bash
### Ver la IP pública de tu VPS
```bash
curl ifconfig.me
```
### O ver las interfaces de red
```bash
ip addr show
```
## 🔌 4. Conectar desde tu máquina local
Desde línea de comandos:
bash
### Sintaxis: mysql -h IP_VPS -P PUERTO -u USUARIO -p
```bash
mysql -h TU_VPS_IP -P 3306 -u root -p
```

### Ejemplo:
```bash
mysql -h 192.168.1.100 -P 3306 -u root -p
```
### Desde herramientas gráficas:
MySQL Workbench

DBeaver

HeidiSQL

phpMyAdmin

Configuración:

text
```
Host: TU_VPS_IP
Port: 3306
User: root
Password: TuPasswordRootSuperSeguro123!
```
## 🛡️ 5. Configuración de Seguridad (IMPORTANTE)
Crear usuario específico para acceso externo:
sql
 * Conectarse al MySQL del VPS
```bash
docker exec -it mysql-produccion mysql -u root -p
```

 * Crear usuario para acceso externo

```
CREATE USER 'usuario_externo'@'%' IDENTIFIED BY 'PasswordExternoSeguro123!';
GRANT SELECT, INSERT, UPDATE, DELETE ON mi_aplicacion.* TO 'usuario_externo'@'%';
FLUSH PRIVILEGES;
```
* Ver usuarios creados
```
SELECT User, Host FROM mysql.user;
```
O crear usuario solo para tu IP:
sql

 * Más seguro: solo desde tu IP específica
 ```
CREATE USER 'usuario_seguro'@'TU_IP_PERSONAL' IDENTIFIED BY 'PasswordSeguro456!';
GRANT ALL PRIVILEGES ON mi_aplicacion.* TO 'usuario_seguro'@'TU_IP_PERSONAL';
FLUSH PRIVILEGES;
```
## 🧪 6. Probar la conexión externa
Test básico de conectividad:
bash
### Verificar que el puerto responde
```bash
telnet TU_VPS_IP 3306
```

### O con netcat
```bash
nc -zv TU_VPS_IP 3306
```
Conectar con el nuevo usuario:
bash
###  Con usuario específico para externo
```bash
mysql -h TU_VPS_IP -P 3306 -u usuario_externo -p
```

# Una vez conectado, verificar permisos
```bash

SHOW DATABASES;
USE mi_aplicacion;
SHOW TABLES;
```
## ⚠️ 7. Solución de problemas comunes
Error: "Host is not allowed to connect"
sql
* Verificar que el usuario tenga acceso desde '%'
```bash

SELECT User, Host FROM mysql.user WHERE User = 'usuario_externo';
```

 * Si no existe, crear de nuevo
 ```

CREATE USER 'usuario_externo'@'%' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON *.* TO 'usuario_externo'@'%';
FLUSH PRIVILEGES;
```

Error: "Can't connect to MySQL server"
bash
## Verificar que el contenedor está corriendo
```bash

docker-compose ps
```

## Ver logs de MySQL
```bash

docker-compose logs mysql
```


## Verificar que el puerto está escuchando
```bash

netstat -tulpn | grep 3306
```

Error: Firewall bloqueando
bash
## Verificar reglas UFW
```bash

sudo ufw status
```

## Verificar iptables
```bash

sudo iptables -L
```

# 🔄 8. Configuración automática con init script
Crear script de inicialización:
bash
```bash

mkdir -p scripts
```

📄 scripts/init-users.sql
sql
 * Crear usuario para acceso externo

 ```
CREATE USER IF NOT EXISTS 'externo'@'%' IDENTIFIED BY '${MYSQL_EXTERNAL_PASSWORD}';
GRANT SELECT, INSERT, UPDATE, DELETE ON ${MYSQL_DATABASE}.* TO 'externo'@'%';
```

 Crear usuario administrativo
```

CREATE USER IF NOT EXISTS 'admin'@'%' IDENTIFIED BY '${MYSQL_ADMIN_PASSWORD}';
GRANT ALL PRIVILEGES ON ${MYSQL_DATABASE}.* TO 'admin'@'%';

FLUSH PRIVILEGES;
```

 * Actualizar docker-compose.yml:
yaml
```

services:
  mysql:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
      MYSQL_DATABASE: ${MYSQL_DATABASE}
    ports:
      - "3306:3306"
    volumes:
      - mysql_data:/var/lib/mysql
      - ./scripts/init-users.sql:/docker-entrypoint-initdb.d/init-users.sql
    restart: unless-stopped
    command: 
      - --bind-address=0.0.0.0
```

 * Actualizar .env:
env
```

MYSQL_ROOT_PASSWORD=TuPasswordRootSuperSeguro123!
MYSQL_DATABASE=mi_aplicacion
MYSQL_EXTERNAL_PASSWORD=PasswordExterno456!
MYSQL_ADMIN_PASSWORD=PasswordAdmin789!
```

# 🚀 9. Comandos finales de despliegue
bash
## En tu VPS
```bash

cd ~/mysql-docker
```

## Reiniciar con la nueva configuración
```bash

docker-compose down
docker-compose up -d
```


## Verificar que todo funcione
```bash

docker-compose logs -f mysql
```


## Probar conexión externa desde tu local
```bash

mysql -h TU_VPS_IP -P 3306 -u externo -p
```

# 📱 10. URLs de acceso

MySQL Directo: mysql -h TU_VPS_IP -P 3306 -u root -p

Con usuario externo: mysql -h TU_VPS_IP -P 3306 -u externo -p

Para aplicaciones: Usar externo con permisos limitados

¡Listo! Ahora puedes acceder a tu MySQL desde cualquier lugar usando la IP de tu VPS y el puerto 3306.

