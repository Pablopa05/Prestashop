# Prestashop
1. Configuración de la instancia EC2 en AWS
Se lanzó una instancia EC2 basada en Ubuntu Server.
Se asignó una dirección IP elástica para garantizar que la instancia mantenga la misma dirección IP pública en caso de reinicios.
Se configuraron los grupos de seguridad para permitir el tráfico HTTP (puerto 80), HTTPS (puerto 443), y SSH (puerto 22).
2. Configuración de DNS
Se utilizó el servicio NO-IP para asociar el nombre de dominio personalizado (miprestashops.zapto.org) con la dirección IP elástica de la instancia EC2.
Verificado con el comando nslookup para confirmar que el dominio apunta correctamente a la instancia.
3. Instalación de la pila LAMP
Se diseñó y ejecutó un script Bash (install_lamp.sh) para automatizar la instalación de Apache, MySQL y PHP junto con los módulos necesarios:
Apache para servir la aplicación web.
MySQL para la base de datos de PrestaShop.
PHP y sus extensiones para compatibilidad con PrestaShop.
4. Configuración de Certbot y HTTPS
Se instaló y configuró Certbot para obtener un certificado SSL/TLS gratuito de Let's Encrypt.
El script Bash (setup_letsencrypt_https.sh) automatizó el proceso de instalación de Certbot y la configuración de HTTPS en el dominio.
5. Instalación y configuración de PrestaShop
Se diseñó y ejecutó el script (deploy_prestashop.sh) para descargar, extraer y configurar PrestaShop:
Configuración de permisos para que Apache pueda acceder y ejecutar los archivos de PrestaShop.
Creación de la base de datos y usuario para PrestaShop.
Configuración de un archivo VirtualHost de Apache para servir el sitio desde /var/www/html/prestashop.
6. Verificación de requisitos de PrestaShop
Se instalaron extensiones adicionales de PHP, como intl, para cumplir con los requisitos técnicos de PrestaShop.
Se verificaron los parámetros de PHP y se ajustaron según las recomendaciones dadas por el instalador de PrestaShop.
7. Finalización de la instalación
Se accedió al dominio configurado para completar la instalación a través del navegador.
Los datos de la base de datos creados anteriormente se usaron para conectar PrestaShop con MySQL.
Una vez completada la instalación, se eliminó la carpeta /install para garantizar la seguridad.
Scripts de Bash utilizados
A continuación, se muestran los scripts utilizados para automatizar la instalación y configuración:

scripts/install_lamp.sh
Este script instala la pila LAMP en la instancia.

echo "Instalando pila LAMP..."
sudo apt update && sudo apt upgrade -y
sudo apt install -y apache2 mysql-server php libapache2-mod-php php-mysql php-xml php-curl php-gd php-mbstring unzip
sudo a2enmod rewrite
sudo systemctl restart apache2
scripts/setup_letsencrypt_https.sh
Este script configura HTTPS mediante Certbot.

source .env

echo "Instalando Certbot..."
sudo apt install -y certbot python3-certbot-apache
sudo certbot --apache -d $DOMAIN
sudo systemctl restart apache2
scripts/deploy_prestashop.sh
Este script descarga, extrae e instala PrestaShop.

source .env

# Variables
DB_NAME="prestashop_db"
DB_USER="prestashop_user"
DB_PASS="securepassword"

echo "Creando base de datos para PrestaShop..."
sudo mysql -e "CREATE DATABASE $DB_NAME CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;"
sudo mysql -e "CREATE USER '$DB_USER'@'localhost' IDENTIFIED BY '$DB_PASS';"
sudo mysql -e "GRANT ALL PRIVILEGES ON $DB_NAME.* TO '$DB_USER'@'localhost';"
sudo mysql -e "FLUSH PRIVILEGES;"

echo "Descargando e instalando PrestaShop..."
cd /var/www/html
sudo wget https://www.prestashop.com/download/latest -O prestashop.zip
sudo unzip prestashop.zip -d prestashop
sudo chown -R www-data:www-data /var/www/html/prestashop
sudo chmod -R 755 /var/www/html/prestashop
scripts/.env
Este archivo contiene las variables de entorno utilizadas en los scripts.

# Variables de la base de datos
DB_NAME="prestashop_db"
DB_USER="prestashop_user"
DB_PASS="securepassword"

# Dominio
DOMAIN="miprestashops.zapto.org"
conf/000-default.conf
Archivo de configuración de Apache.

apache
<VirtualHost *:80>
    ServerName miprestashops.zapto.org
    DocumentRoot /var/www/html/prestashop

    <Directory /var/www/html/prestashop>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/prestashop_error.log
    CustomLog ${APACHE_LOG_DIR}/prestashop_access.log combined
</VirtualHost>
Ejecuta los scripts en orden:
Instala LAMP:
scripts/install_lamp.sh
Configura HTTPS:
scripts/setup_letsencrypt_https.sh
Instala PrestaShop:
scripts/deploy_prestashop.sh
