# Jarkom-Modul-2-D20-2023
## Soal 1
Yudhistira akan digunakan sebagai DNS Master, Werkudara sebagai DNS Slave, Arjuna merupakan Load Balancer yang terdiri dari beberapa Web Server yaitu Prabakusuma, Abimanyu, dan Wisanggeni. Buatlah topologi dengan pembagian [sebagai berikut](https://docs.google.com/spreadsheets/d/1OqwQblR_mXurPI4gEGqUe7v0LSr1yJViGVEzpMEm2e8/edit?usp=sharing). Folder topologi dapat diakses pada [drive berikut](https://drive.google.com/drive/folders/1Ij9J1HdIW4yyPEoDqU1kAwTn_iIxg3gk?usp=sharing) 
## Soal 2
Buatlah website utama pada node arjuna dengan akses ke arjuna.yyy.com dengan alias www.arjuna.yyy.com dengan yyy merupakan kode kelompok.
## Soal 3
Dengan cara yang sama seperti soal nomor 2, buatlah website utama dengan akses ke abimanyu.yyy.com dan alias www.abimanyu.yyy.com.
## Soal 4 
Kemudian, karena terdapat beberapa web yang harus di-deploy, buatlah subdomain parikesit.abimanyu.yyy.com yang diatur DNS-nya di Yudhistira dan mengarah ke Abimanyu.
## Soal 5
Buat juga reverse domain untuk domain utama. (Abimanyu saja yang direverse)
## Soal 6
Agar dapat tetap dihubungi ketika DNS Server Yudhistira bermasalah, buat juga Werkudara sebagai DNS Slave untuk domain utama.
## Soal 7
Seperti yang kita tahu karena banyak sekali informasi yang harus diterima, buatlah subdomain khusus untuk perang yaitu baratayuda.abimanyu.yyy.com dengan alias www.baratayuda.abimanyu.yyy.com yang didelegasikan dari Yudhistira ke Werkudara dengan IP menuju ke Abimanyu dalam folder Baratayuda.
## Soal 8
Untuk informasi yang lebih spesifik mengenai Ranjapan Baratayuda, buatlah subdomain melalui Werkudara dengan akses rjp.baratayuda.abimanyu.yyy.com dengan alias www.rjp.baratayuda.abimanyu.yyy.com yang mengarah ke Abimanyu.
## Soal 9
Arjuna merupakan suatu Load Balancer Nginx dengan tiga worker (yang juga menggunakan nginx sebagai webserver) yaitu Prabakusuma, Abimanyu, dan Wisanggeni. Lakukan deployment pada masing-masing worker.
### Arjuna
Install Bind9 dan Nginx di Arjuna:
```
 apt-get update
 apt-get install bind9 nginx
```
### Prabakusuma
Install lalu setup Nginx dan PHP
```
 apt-get update && apt install nginx php php-fpm -y
```
Buat direktori baru di `/var/www` dengan nama `prabakusuma`
```
 mkdir /var/www/prabakusuma
```
masuk direktori `prabakusuma` ltambahkan file `index.php` dari soal yang berisi seperti berikut
``` php
<?php
$hostname = gethostname();
$date = date('Y-m-d H:i:s');
$php_version = phpversion();
$username = get_current_user();



echo "Hello World!<br>";
echo "Saya adalah: $username<br>";
echo "Saat ini berada di: $hostname<br>";
echo "Versi PHP yang saya gunakan: $php_version<br>";
echo "Tanggal saat ini: $date<br>";
?>
```
Buat konfigurasi server block untuk worker di direktori `/etc/nginx/sites-available/` dengan nama file `prabakusuma`
```
nano prabakusuma
```
Lalu isi dengan konfigurasi berikut
```
server {
    listen 80;
    root /var/www/prabakusuma;  
    index index.php index.html index.htm;
    server_name _;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    # pass PHP scripts to FastCGI server
    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php7.2-fpm.sock;  
    }

    location ~ /\.ht {
        deny all;
    }

    error_log /var/log/nginx/prabakusuma_error.log;  
    access_log /var/log/nginx/prabakusuma_access.log; 
}
```
Lalu simpan kemudian buat `symlink`
```
 ln -s /etc/nginx/sites-available/jarkom /etc/nginx/sites-enabled
```
Restart Nginx
```
service nginx restart
```
Uji konfigurasi
```
nginx -t
```

Ulangi langkah diatas pada worker lainnya yaitu `Abimanyu` dan `Wisanggeni`
## Soal 10
Kemudian gunakan algoritma Round Robin untuk Load Balancer pada Arjuna. Gunakan server_name pada soal nomor 1. Untuk melakukan pengecekan akses alamat web tersebut kemudian pastikan worker yang digunakan untuk menangani permintaan akan berganti ganti secara acak. Untuk webserver di masing-masing worker wajib berjalan di port 8001-8003. Contoh
    - Prabakusuma:8001
    - Abimanyu:8002
    - Wisanggeni:8003
    
### Arjuna 
Buat file baru di direktori `/etc/nginx/sites-available` dengan nama `lb-arjuna`
```
http {
    upstream workers {
        server 192.201.3.2:8001;
        server 192.201.3.3:8002;
        server 192.201.3.4:8003;
    }

    server {
        listen 80;
        server_name arjuna.d20.com;
        location / {
            proxy_pass http://workers;
        }
    }
}
```
Simpan kemudian buat `symlink`
```
 ln -s /etc/nginx/sites-available/lb-jarkom /etc/nginx/sites-enabled
```
### Nakula
Lakukan pengujian pada client Nakula dengan menjalankan perintah lynx
```
lynx http://arjuna.d20.com
```
## Soal 11
Selain menggunakan Nginx, lakukan konfigurasi Apache Web Server pada worker Abimanyu dengan web server www.abimanyu.yyy.com. Pertama dibutuhkan web server dengan DocumentRoot pada /var/www/abimanyu.yyy  
### Abimanyu
Install Apache, PHP, OpenSSL
```
apt-get install apache2 -y
service apache2 start
apt-get install php -y
apt-get install libapache2-mod-php7.0 -y
service apache2 
apt-get install ca-certificates openssl -y
```
Buat file konfigurasi baru dengan nama `abimanyu.d20.com.conf` pada direktori `/etc/apache2/sites-available/`, lalu isi dengan konfigurasi berikut
```
<VirtualHost *:80>
    ServerAdmin webmaster@abimanyu.d20.com
    ServerName www.abimanyu.d20.com
    DocumentRoot /var/www/abimanyu.d20

    ErrorLog \${APACHE_LOG_DIR}/error.log
    CustomLog \${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
Aktifkan konfigurasi 
```
a2ensite abimanyu.d20.com.conf
```
Restart apache dengan perintah `service apache2 restart`  
Pindah ke direktori `/var/www` lalu buat direkortori baru dengan nama `abimanyu.d20.com`, lalu tambahkan file `index.php` dari soal yang berisi seperti ini
``` php
<?php
	if($_SERVER['REQUEST_URI'] == '/index.php/home' || $_SERVER['REQUEST_URI'] == '/home' || $_SERVER['REQUEST_URI'] == '/index.php' || $_SERVER['REQUEST_URI'] == '/') readfile('home.html');
	else http_response_code(404);
?>
```
Uji coba menggunakan `lynx abimanyu.d20.com` 
## Soal 12
Setelah itu ubahlah agar url www.abimanyu.yyy.com/index.php/home menjadi www.abimanyu.yyy.com/home.
### Abimanyu
Konfigurasi file `/var/www/abimanyu.d20.com/.htaccess`
```
a2enmod rewrite
service apache2 restart
echo "
RewriteEngine On
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule (.*) /index.php/\$1 [L]
```
Buka file `abimanyu.d20.com.conf` pada direktori `/etc/apache2/sites-available/` lalu tambahkan aturan penulisan ulang URL seperti berikut  
```
<VirtualHost *:80>
    ServerAdmin webmaster@abimanyu.d20.com
    ServerName www.abimanyu.d20.com
    DocumentRoot /var/www/abimanyu.d20

    ErrorLog \${APACHE_LOG_DIR}/error.log
    CustomLog \${APACHE_LOG_DIR}/access.log combined

    <Directory /var/www/abimanyu.d20.com>
    	Options +FollowSymLinks -Multiviews
    	AllowOverride All
</VirtualHost>
```
Restart apache `service apache2 restart`
## Soal 13
Selain itu, pada subdomain www.parikesit.abimanyu.yyy.com, DocumentRoot disimpan pada /var/www/parikesit.abimanyu.yyy 
### Abimanyu
Konfigurasi file /etc/apache2/sites-available/parikesit.abimanyu.d20.com.conf  
```
<VirtualHost *:80>

        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/parikesit.abimanyu.d20.com
        ServerName parikesit.abimanyu.d20.com
        ServerAlias www.parikesit.abimanyu.d20.com

        ErrorLog \${APACHE_LOG_DIR}/error.log
        CustomLog \${APACHE_LOG_DIR}/access.log combined

        <Directory /var/www/abimanyu.d20.com>
                Options +FollowSymLinks -Multiviews
                AllowOverride All
        </Directory>
</VirtualHost>
```
Aktifkan dengan `a2ensite parikesit.abimanyu.d20.com`  
Lalu buat direktori baru dan copy file content  
```
mkdir /var/www/parikesit.abimanyu.d20.com
cp -r /var/www/parikesit.abimanyu/. /var/www/parikesit.abimanyu.d20.com
```
kemudian `service apache2 restart`  
Uji coba pada client  
```
lynx parikesit.abimanyu.d20.com
```
## Soal 14
Pada subdomain tersebut folder /public hanya dapat melakukan directory listing sedangkan pada folder /secret tidak dapat diakses (403 Forbidden).  
### Abimanyu
Konfigurasi file /etc/apache2/sites-available/parikesit.abimanyu.d20.com.conf  
```
<VirtualHost *:80>

        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/parikesit.abimanyu.d20.com
        ServerName parikesit.abimanyu.d20.com
        ServerAlias www.parikesit.abimanyu.d20.com

        <Directory /var/www/parikesit.abimanyu.d20.com/public>
                Options +Indexes
        </Directory>

	<Directory /var/www/parikesit.abimanyu.d20/secret>
	        Options -Indexes  
	        Deny from all 
	</Directory>

        ErrorLog \${APACHE_LOG_DIR}/error.log
        CustomLog \${APACHE_LOG_DIR}/access.log combined

        <Directory /var/www/abimanyu.d20.com>
                Options +FollowSymLinks -Multiviews
                AllowOverride All
        </Directory>
</VirtualHost>
```
Lalu lakukan `service apache2 restart`  dan lakukan pengujian 
## Soal 15
Buatlah kustomisasi halaman error pada folder /error untuk mengganti error kode pada Apache. Error kode yang perlu diganti adalah 404 Not Found dan 403 Forbidden.  
### Abimanyu
Konfigurasi file /etc/apache2/sites-available/parikesit.abimanyu.d20.com.conf  
```
<VirtualHost *:80>

        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/parikesit.abimanyu.d20.com
        ServerName parikesit.abimanyu.d20.com
        ServerAlias www.parikesit.abimanyu.d20.com

	ErrorDocument 404 /error/404.html
	ErrorDocument 403 /error/403.html

        <Directory /var/www/parikesit.abimanyu.d20.com/public>
                Options +Indexes
        </Directory>

	<Directory /var/www/parikesit.abimanyu.d20/secret>
	        Options -Indexes  
	        Deny from all 
	</Directory>

        ErrorLog \${APACHE_LOG_DIR}/error.log
        CustomLog \${APACHE_LOG_DIR}/access.log combined

        <Directory /var/www/abimanyu.d20.com>
                Options +FollowSymLinks -Multiviews
                AllowOverride All
        </Directory>
</VirtualHost>
```
Lalu lakukan `service apache2 restart`  dan lakukan pengujian 
## Soal 16
Buatlah suatu konfigurasi virtual host agar file asset www.parikesit.abimanyu.yyy.com/public/js menjadi www.parikesit.abimanyu.yyy.com/js   
### Abimanyu
Konfigurasi file /etc/apache2/sites-available/parikesit.abimanyu.d20.com.conf  lalu tambahkan `Alias`
```
<VirtualHost *:80>

        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/parikesit.abimanyu.d20.com
        ServerName parikesit.abimanyu.d20.com
        ServerAlias www.parikesit.abimanyu.d20.com

	ErrorDocument 404 /error/404.html
	ErrorDocument 403 /error/403.html

        <Directory /var/www/parikesit.abimanyu.d20.com/public>
                Options +Indexes
        </Directory>

	<Directory /var/www/parikesit.abimanyu.d20/secret>
	        Options -Indexes  
	        Deny from all 
	</Directory>

        Alias \"/js\" \"/var/www/parikesit.abimanyu.d20.com/public/js\"

        ErrorLog \${APACHE_LOG_DIR}/error.log
        CustomLog \${APACHE_LOG_DIR}/access.log combined

        <Directory /var/www/abimanyu.d20.com>
                Options +FollowSymLinks -Multiviews
                AllowOverride All
        </Directory>
</VirtualHost>
```
Lalu lakukan `service apache2 restart`  dan lakukan pengujian 
## Soal 17
Agar aman, buatlah konfigurasi agar www.rjp.baratayuda.abimanyu.yyy.com hanya dapat diakses melalui port 14000 dan 14400.  
### Abimanyu
Konfigurasi file `/etc/apache2/sites-available/rjp.baratayuda.abimanyu.d20.com.conf` dengan menambahkan virtual host baru yang berada pada port 14000 dan 14400  
```
<VirtualHost *:14000>

        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/grjp.baratayuda.abimanyu.d20.com
        ServerName rjp.baratayuda.abimanyu.d20.com
        ServerAlias www.rjp.baratayuda.abimanyu.d20.com

        ErrorLog \${APACHE_LOG_DIR}/error.log
        CustomLog \${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
<VirtualHost *:14400>

        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/rjp.baratayuda.abimanyu.d20.com
        ServerName rjp.baratayuda.abimanyu.d20.com
        ServerAlias www.rjp.baratayuda.abimanyu.d20.com

        ErrorLog \${APACHE_LOG_DIR}/error.log
        CustomLog \${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
Kemudian
```
a2ensite rjp.baratayuda.abimanyu.d20.com
service apache2 restart
mkdir /var/www/rjp.baratayuda.abimanyu.d20.com
cp -r /var/www/rjp.baratayuda.abimanyu/. /var/www/rjp.baratayuda.abimanyu.d20.com/
```
Selanjutnya konfigurasi file `/etc/apache2/ports.conf` 
```
Listen 80
Listen 14000
Listen 14400
<IfModule ssl_module>
        Listen 443
</IfModule>

<IfModule mod_gnutls.c>
        Listen 443
</IfModule>
```
Lalu `service apache2 restart`  dan lakukan pengujian 
## Soal 18
Untuk mengaksesnya buatlah autentikasi username berupa “Wayang” dan password “baratayudayyy” dengan yyy merupakan kode kelompok. Letakkan DocumentRoot pada /var/www/rjp.baratayuda.abimanyu.yyy. 
### Abimanyu
Jalankan command berikut
```
htpasswd -c /etc/apache2/htpasswd baratayuda.d20 Wayang
```
Kemudian konfigurasi file `/etc/apache2/sites-available/rjp.baratayuda.abimanyu.d20.com.conf` 
```
<VirtualHost *:14000>

        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/grjp.baratayuda.abimanyu.d20.com
        ServerName rjp.baratayuda.abimanyu.d20.com
        ServerAlias www.rjp.baratayuda.abimanyu.d20.com

        <Directory \"/var/www/rjp.baratayuda.abimanyu.d20.com\">
                AuthType Basic
                AuthName \"Restricted Content\"
                AuthUserFile /etc/apache2/.htpasswd
                Require valid-user
        </Directory>

        ErrorLog \${APACHE_LOG_DIR}/error.log
        CustomLog \${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
<VirtualHost *:14400>

        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/rjp.baratayuda.abimanyu.d20.com
        ServerName rjp.baratayuda.abimanyu.d20.com
        ServerAlias www.rjp.baratayuda.abimanyu.d20.com

        <Directory \"/var/www/rjp.baratayuda.abimanyu.d20.com\">
                AuthType Basic
                AuthName \"Restricted Content\"
                AuthUserFile /etc/apache2/.htpasswd
                Require valid-user
        </Directory>

        ErrorLog \${APACHE_LOG_DIR}/error.log
        CustomLog \${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
Lalu `service apache2 restart` dan lakukan pengujian  
## Soal 19
Buatlah agar setiap kali mengakses IP dari Abimanyu akan secara otomatis dialihkan ke www.abimanyu.yyy.com (alias)  
### Abimanyu
Konfigurasi file `/etc/apache2/sites-available/000-default.conf`
```
<VirtualHost *:80>

        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/html

        RewriteEngine On
        RewriteCond %{HTTP_HOST} !^abimanyu.d20.com$
        RewriteRule /.* http://abimanyu.d20.com/ [R]

        ErrorLog \${APACHE_LOG_DIR}/error.log
        CustomLog \${APACHE_LOG_DIR}/access.log combined

</VirtualHost>
```
Lalu `service apache2 restart` kemudian uji `lynx 192.201.3.3`
## Soal 20
Karena website www.parikesit.abimanyu.yyy.com semakin banyak pengunjung dan banyak gambar gambar random, maka ubahlah request gambar yang memiliki substring “abimanyu” akan diarahkan menuju abimanyu.png. 
### Abimanyu
konfigurasi file `/var/www/parikesit.abimanyu.d20.com/.htaccess` 
```
echo "
RewriteEngine On
RewriteCond %{REQUEST_URI} ^/public/images/(.*)abimanyu(.*)
RewriteCond %{REQUEST_URI} !/public/images/abimanyu.png
RewriteRule /.* http://parikesit.abimanyu.d20.com/public/images/abimanyu.png [L]
"
```
Lalu konfigurasi file `/etc/apache2/sites-available/parikesit.abimanyu.d20.com.conf`
```
<VirtualHost *:80>

        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/parikesit.abimanyu.d20.com
        ServerName parikesit.abimanyu.d20.com
        ServerAlias www.parikesit.abimanyu.d20.com

	ErrorDocument 404 /error/404.html
	ErrorDocument 403 /error/403.html

        <Directory /var/www/parikesit.abimanyu.d20.com/public>
                Options +Indexes
        </Directory>

	<Directory /var/www/parikesit.abimanyu.d20/secret>
	        Options -Indexes  
	        Deny from all 
	</Directory>

        Alias \"/js\" \"/var/www/parikesit.abimanyu.d20.com/public/js\"

        <Directory /var/www/parikesit.abimanyu.d20.com>
                Options +FollowSymLinks -Multiviews
                AllowOverride All
        </Directory>

        ErrorLog \${APACHE_LOG_DIR}/error.log
        CustomLog \${APACHE_LOG_DIR}/access.log combined

        <Directory /var/www/abimanyu.d20.com>
                Options +FollowSymLinks -Multiviews
                AllowOverride All
        </Directory>
</VirtualHost>
```
Kemudian `service apache2 restart` dan lakukan pengujian
