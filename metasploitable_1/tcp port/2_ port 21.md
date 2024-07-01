# آنچه گذشت
این‌ها پورت‌های باز tcp بودند که روی ماشین مجازی داشتیم:
<details>
<summary>Tcp port ports</summary>
</br>
  
  ```
Starting Nmap 7.94 ( https://nmap.org ) at 2024-04-14 18:58 +0330
Nmap scan report for 192.168.56.102
Host is up (0.000092s latency).
Not shown: 65506 closed tcp ports (reset)
PORT      STATE SERVICE
21/tcp    open  ftp
22/tcp    open  ssh
23/tcp    open  telnet
25/tcp    open  smtp
53/tcp    open  domain
80/tcp    open  http
111/tcp   open  rpcbind
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
512/tcp   open  exec
513/tcp   open  login
514/tcp   open  shell
1099/tcp  open  rmiregistry
1524/tcp  open  ingreslock
2049/tcp  open  nfs
2121/tcp  open  ccproxy-ftp
3306/tcp  open  mysql
3632/tcp  open  distccd
5432/tcp  open  postgresql
5900/tcp  open  vnc
6000/tcp  open  X11
6667/tcp  open  irc
6697/tcp  open  ircs-u
8009/tcp  open  ajp13
8180/tcp  open  unknown
8787/tcp  open  msgsrvr
33847/tcp open  unknown
35181/tcp open  unknown
35811/tcp open  unknown
41689/tcp open  unknown
MAC Address: 08:00:27:F6:23:56 (Oracle VirtualBox virtual NIC)

Nmap done: 1 IP address (1 host up) scanned in 15.48 seconds

```
</details>


توی خروجی، تعداد پورت‌های بسته tcp را داشتیم: `Not shown: 65506 closed tcp ports`

شماره پورت/پروتکلش/وضعیتش و سرویسی که روی اون پورت داره اجرا میشه.

# نگاهی عمیق‌تر به پورت‌ها
اولین پورت بازی که nmpa پیدا کرده، پورت tcp ۲۱ هست که سرویس ftp روی اون اجرا میشه. اول از همه بیاین کمی بیشتر درباره این سرویس بدونیم تا بفهمیم با چی طرفیم.

پروتکل ftp یا file transfer protocol یک پروتکل tcp/ip که برای انتقال فایل بین دو دستگاه در شبکه به‌کار میره. برای درک بهتر اون می‌تونیم از یک ابزار ساده به نام ftp روی کالی لینوکس استفاده کنیم. برای اتصال به سرویس ftp روی پورت ۲۱ کاربر از دستور زیر استفاده می‌کنیم:

`ftp 192.168.56.101 -p 21`


بعد از وارد کردن دستور، این خروجی را داریم:

```
Connected to 192.168.56.102.
220 (vsFTPd 2.3.4)
Name (192.168.56.101:mahyar):
```
عبارت Connected to 192.168.56.102 یعنی روی پورت ۲۱ سرویس ftp وجود داشت و ما تونستیم بهش وصل بشیم.
اینجا هم 220 (vsFTPd 2.3.4) کد ۲۲۰ به معنی اتصال موفقیت‌آمیز هست ( سرویس و نسخه استفاده شده از پروتکل ftp را هم اینجا بهمون نشون داده `vsFTPd 2.3.4` که خودش یک ضعف امنیتی است که بعدا درموردش صحبت می‌کنیم ).
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
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-05-29 17:58 +0330
Nmap scan report for 192.168.56.102
Host is up (0.00068s latency).

PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 2.3.4
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to 192.168.56.1
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      vsFTPd 2.3.4 - secure, fast, stable
|_End of status
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
MAC Address: 08:00:27:F6:23:56 (Oracle VirtualBox virtual NIC)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running: Linux 2.6.X
OS CPE: cpe:/o:linux:linux_kernel:2.6
OS details: Linux 2.6.9 - 2.6.33
Network Distance: 1 hop
Service Info: OS: Unix

TRACEROUTE
HOP RTT     ADDRESS
1   0.68 ms 192.168.56.102

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 2.56 seconds
```

</details>


اطلاعات مهمی مثل نرم‌افزار و نسخه‌‌ای از اون که داره پروتکل ftp را اجرا می‌کنه: `vsftpd 2.3.4`
Anonymous login هم مجاز هست ( که بعدا در موردش صحبت می‌کنیم )
مک آدرس سرور: `08:00:27:F6:23:56 (Oracle VirtualBox virtual NIC)` که مشخص کرده تارگت ما روی VirtualBox اجرا میشه
نسخه کرنل لینوکس سرور بین 2.6.9 تا 2.6.33 هست.

یکم قبل‌تر دیدین که ما می‌تونیم به سروری که داره سرویس ftp اجرا می‌کنه وصل بشیم، فایل‌هاش رو ببینیم، دانلود و آپلود کنیم و از طریق پورت ۲۱ و سرویس ftp به سرور خودمون توی شبکه و یا از راه دور دسترسی داشته باشیم. اما سوال اینجاست که سرویس‌های ftp که روی سرور اجرا میشند، اغلب پیکربندی‌های سفت و سختی دارند. دسترسی‌های محدود، آیپی‌آدرس‌های خاص، و یا در مثال ما، نام کاربری و رمزعبور تایید شده‌ای نیاز داره. حالا که ما به این مقادیر ( مثلا یوزرنیم پسورد ) دسترسی نداریم، باید چیکار کنیم؟ خب به صورت کلی ۳ تا راه برای حل این مشکل، و یا دقیق‌ترش، بررسی امنیت سرویس ftp روی سرور و این‌که آیا ما به عنوان مهاجم می‌تونیم بهش وصل بشیم یا نه، وجود داره.
