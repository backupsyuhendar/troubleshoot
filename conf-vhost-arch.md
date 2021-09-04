1| Tambahkan port di /etc/httpd/conf/httpd.conf agar terbaca
di awal2 baris: 
Listen 81

Buat configurasi baru di dalam /etc/httpd/conf/extra
Namanya bebas contoh : httpd-myproject.conf
isi :
<VirtualHost *:81>
    DocumentRoot /srv/http/nama_direktori
    ErrorLog "/var/log/httpd/nama_dir-error_log"
    CustomLog "/var/log/httpd/nama_dir-access_log" common
    <Directory "/srv/http/nama_direktori">
        Require all granted
    </Directory> 
</VirtualHost>


Masukkan configurasi di /etc/httpd/conf/httpd.conf
di baris akhir :
Include conf/extra/httpd-myproject.conf

-------------------------------
2| Cara lain ada di wiki arch:
https://wiki.archlinux.org/index.php/Apache_HTTP_Server#Virtual_hosts

-------------------------------
3| Cara lain :
enable vhost di httpd.conf
dgn uncomment :
Include extra/httpd-vhost.conf

lalu buat di httpd-vhost.conf :
<VirtualHost *:81>
    DocumentRoot /srv/http/nama_direktori
    ErrorLog "/var/log/httpd/nama_dir-error_log"
    CustomLog "/var/log/httpd/nama_dir-access_log" common
    <Directory "/srv/http/nama_direktori">
        Require all granted
    </Directory> 
</VirtualHost>