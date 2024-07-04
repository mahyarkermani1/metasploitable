# آنچه گذشت
در قسمت قبل، پورت ۲۱ و پروتکل ftp را باهم بررسی کردیم و انواع روش‌های تست‌نفوذ اون را دیدیم. در این بخش می‌خوایم روی پورت ۲۲ پروتکل ssh از سیستم‌عامل متاسپلویتیبل کار کنیم.

# کمی عمیق‌تر
قبل‌ازاین‌که وارد اسکن کردن بشیم و پورت ۲۱ سرور را نگاه کنیم تا ببینیم چه سرویسی مسئول اجرای ssh است، بیاین کمی درباره پروتکل ftp و ssh صحبت کنیم. در بخش قبلی، ما مثال‌های مختلفی را از پروتکل ftp دیدیم. که باعث میشد دوتا کامپیوتر در شبکه بتونند فایل‌هاشون را به‌همدیگه انتقال بدند. این تمام قابلیت ftp بود که می‌تونه به عنوان یک سرور فعال بشه و کاربران بهش وصل بشن و فایل‌هایی را اونجا ببینند یا به‌اشتراک بزارند. اما ssh مخفف secute shell علاوه‌بر انتقال فایل، قابلیت‌های دیگه‌ای هم مثل دستورات shell/ اجرای دستورات ویندوزی، تونلینگ، پورت‌فورواردینگ و غیره را فراهم می‌کنه. درواقع شما با استفاده ازاین پروتکل، می‌تونید از راه دور به یک سیستم‌عامل متصل بشید و از سرویس‌ها، و قابلیت‌های اون سیستم‌عامل هم استفاده کنید. همچنین تمرکز ssh بر روی رمزنگاری و ارتباطات راه‌دور به‌صورت ایمن است.
بیاین برای درک بیشتر، یک مثال از ssh را هم باهم ببینیم. من با دستور زیر به پورت ۲۲ سرور خودم متصل میشم:

`ssh -p 22 192.168.56.102`

> [!WARNING]
> سیستم‌عامل متاسپلویتیبل ۱ به دلیل قدیمی‌بودن از الگوریتم‌های قدیمی‌تری برای اتصال به SSH پشتیبانی می‌کنه که ممکنه در ابزار SSH شما پشتیبانی نشه. بنابرین اگه هنگام اتصال چنین خطایی را دریافت کردید: `Unable to negotiate with 192.168.56.102 port 22: no matching host key type found. Their offer: ssh-rsa,ssh-dss` باید در فایل `~/.ssh/config` (اگه این فایل وجود نداشت، یکی بسازید) رشته عبارت
> ```
> Host 192.168.56.102
>  HostKeyAlgorithms +ssh-rsa,ssh-dss
>```
> را قرار بدید تا ابزار ssh شما از الگوریتم‌های ssh-rsa و ssh-dss پشتیبانی کنه. سپس ازشما تاییدیه‌ای میخواد مبنی‌بر اضافه‌کردن کلیدعمومی سرور؛ و بعداز اون مجددا از شما نام‌کاربری و رمزعبور معتبر هم می‌خواد. دراینجا یوزرنیم پسوردهای پیشفرض برای متاسپلویتیبل ۱ همیشه msfadmin:msfadmin است. بنابرین ما باردیگه با نام‌کاربری msfadmin به سرور متصل میشیم:

`ssh -p 22 msfadmin@192.168.56.102`

سپس از ما رمزعبور می‌خواد، که اون را هم msfadmin وارد می‌کنیم. و این خروجی ما:

```
➜  ~ ssh -p 22 msfadmin@192.168.56.102
msfadmin@192.168.56.102's password: 
Linux metasploitable 2.6.24-16-server #1 SMP Thu Apr 10 13:58:00 UTC 2008 i686

The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

To access official Ubuntu documentation, please visit:
http://help.ubuntu.com/
No mail.
Last login: Thu Jul  4 00:07:43 2024 from 192.168.56.1
msfadmin@metasploitable:~$ 

```

همانطور که می‌بینید، درحال حاضر ما با موفقیت به سرور ssh خودمون وصل شدیم. اما دراینجا چه دستوراتی را می‌تونیم اجرا کنیم؟ در بخش قبلی که به سرور ftp وصل شدیم، تنها مجاز بودیم تا دستورات زیر را اجرا کنیم:
```
ftp> ?
Commands may be abbreviated.  Commands are:

!               edit            lpage           nlist           rcvbuf          struct
$               epsv            lpwd            nmap            recv            sunique
account         epsv4           ls              ntrans          reget           system
append          epsv6           macdef          open            remopts         tenex
ascii           exit            mdelete         page            rename          throttle
bell            features        mdir            passive         reset           trace
binary          fget            mget            pdir            restart         type
bye             form            mkdir           pls             rhelp           umask
case            ftp             mls             pmlsd           rmdir           unset
cd              gate            mlsd            preserve        rstatus         usage
cdup            get             mlst            progress        runique         user
chmod           glob            mode            prompt          send            verbose
close           hash            modtime         proxy           sendport        xferbuf
cr              help            more            put             set             ?
debug           idle            mput            pwd             site
delete          image           mreget          quit            size
dir             lcd             msend           quote           sndbuf
disconnect      less            newer           rate            status
```

که شامل دستوراتی محدود و صرفا برای انتقال فایل‌ها مناسب بود ( مثل تغییر دایرکتویر، خوندن فایل‌ها، دانلود/آپلود ). اما زمانی که با پروتکل ssh وصل شدیم، تمام دستوراتی که در سرور وجود داره، و تمام ابزارهایی که اونجا هست را می‌تونیم استفاده کنیم. مثلا می‌تونیم نرم‌افزار پایتون را در سرور اجرا و استفاده کنیم: یک آدرس را پینگ بگیریم: یک فایل را آپلود و دانلود کنیم:

```
# اجرا کردن ابزار پایتون در سرور
msfadmin@metasploitable:~$ python
Python 2.5.2 (r252:60911, Jan 20 2010, 21:48:48) 
[GCC 4.2.4 (Ubuntu 4.2.4-1ubuntu3)] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>> 

# پینگ کردن کالی‌لینوکس خودمون از سرور
msfadmin@metasploitable:~$ ping 192.168.56.1 -c 3
PING 192.168.56.1 (192.168.56.1) 56(84) bytes of data.
64 bytes from 192.168.56.1: icmp_seq=1 ttl=64 time=0.611 ms
64 bytes from 192.168.56.1: icmp_seq=2 ttl=64 time=0.614 ms
64 bytes from 192.168.56.1: icmp_seq=3 ttl=64 time=0.745 ms

--- 192.168.56.1 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 1998ms
rtt min/avg/max/mdev = 0.611/0.656/0.745/0.069 ms

# آپلود کردن یک فایل از سیستم‌عامل خودمون به سرور
scp /home/mahyar/test.txt msfadmin@192.168.56.102:/home/msfadmin
msfadmin@192.168.56.102's password: 
test.txt           100%    6     8.2KB/s   00:00  

# دانلود کردن یک فایل از سرور به سیستم‌عامل خودمون
scp msfadmin@192.168.56.102:/home/msfadmin/test.txt /home/mahyar/
msfadmin@192.168.56.102's password: 
test.txt           100%    6     8.2KB/s   00:00 

## برای آپلود و دانلود کردن فایل‌ها، می‌تونید از ابزار scp در لینوکس خودتون استفاده کنید. دقت کنید که این ابزار باید روی سیستم‌عامل خودتون اجرا بشه تا بتونید آپلود/دانلود را از سرور انجام بدید.
```

درمثال بالا دیدیم که می‌تونیم دستورات شل را هم در سرور با استفاده از ssh اجرا کنیم، که میشه نتیجه گرفت با استفاده از این پروتکل، می‌تونیم کل سیستم‌عامل را هم کنترل کنیم ( برخلاف ftp که تنها برای انتقال فایل‌ها با دستورات محدود عمل می‌کرد ).


# اسکن پورت ۲۲
حالا که درک خوبی پیدا کردیم از پروتکل ssh و کاربردهای اون. ما الان روی متاسپلویتیبل یا سرور خودمون، روی پورت ۲۲، یک پروتکل ssh داشتیم و الان می‌خوایم ببینیم چه سرویسی روی پورت ۲۲ داره پروتکل ssh را اجرا می‌کنه و بررسی کنیم که آیا اون سرویس و نسخه آسیب‌پذیری شناخته‌شده‌ای داره که ما بتونیم ازش استفاده کنیم یا نه.
در قدم اول، پورت ۲۲ سرور را برای دیدن سرویس و نسخه اون اسکن می‌کنیم:

`sudo nmap -p 22 -sV 192.168.56.102`

و این خروجی دستور بالا:

```
➜  ~ sudo nmap -p 22 -sV 192.168.56.102
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-07-04 08:09 +0330
Nmap scan report for 192.168.56.102
Host is up (0.00080s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
MAC Address: 08:00:27:B8:DA:1E (Oracle VirtualBox virtual NIC)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 1.22 seconds

```

بنابرین روی پورت ۲۲ سرویس OpenSSH با نسخه 4.7p1 اجرا میشه. بیاین یک کوچولو درباره این سرویس هم بدونیم. openssh یا open secure shell یک ابزار متن‌باز است که امکان استفاده از پروتکل ssh را به صورت رمزنگاری‌شده و ایمن به‌ما میده و طیف دستورات زیادی را برامون فراهم کرده تا بتونیم با استفاده از اون‌ها، با سرور تعامل داشته باشیم. درمثالی که قبل‌تر باهم دیدیم، این اتصالی که به سرور ssh خودمون داشتیم و ابزارهایی که از راه‌دور ( از روی سیستم‌عامل خودمون ) در سرور اجرا کردیم، همش کار openssh بود که این رابط کاربری را برامون فراهم کرده.
حالا بیاین این نسخه از سرویس را از نظر آسیب‌پذیری‌های موجود هم بررسی کنیم. می‌تونیم درقدم اول از searchsploit استفاده کنیم تا ببینیم آسیب‌پذیری شناخته‌شده‌ای برای این نسخه در دیتابیس exploitdb وجود داره یا نه!

`searchsploit OpenSSH 4.7p1`

> [!NOTE]
> ادامه دارد . . .
