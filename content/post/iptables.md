 ---
title: " بررسی فایروال Iptables"
date: 2021-04-06T19:57:23+03:30
draft: true
toc: false
image: 'iptables.jpg'
tags:
  - Linux
categories: [Linux]
dir : "rtl"
---
IPtables یک فایروال قدرتمند رایگان برای سیستم عامل لینوکس است که امکان اعمال قوانین و محدودیت های خاص برای تبادل اطلاعات در شبکه را برای مدیر سرور فراهم می کند، فایروال IPtables در عین سادگی بسیار توانمند است و اجازه کنترل ترافیک شبکه در لایه چهارم شبکه (transport) و همچنین لایه های بالاتر و پایین تر را به ما می دهد.

معمولا iptables در زمینه های مختلفی مثل کنترل پکت های ورودی و خروجی سیستم، کنترل و ایجاد دسترسی ها و محدودیت و… استفاده می شود. لازم به ذکر می باشد که فایروال IPtables محدود به پروتکل IPv4 بوده و نوع های دیگر این فایروال برای استفاده غیر از IPv4 به اسم های زیر می باشند:

IP6tables: استفاده برای پروتکل IPv6

arptables: استفاده برای پروتکل ARP

برای نصب این فایروال می توانید از دستور زیر استفاده کنید اول repository سیستم را اپدیت می کنیم و سپس اقدام به نصب آن می کنیم.

```bash
sudo apt update
sudo apt install iptables
```

syntax دستوری این فایروال به این صورت می باشد:
```bash
iptables --table TABLE -A/-C/-D... CHAIN rule --jump Target


iptables -A <chain> -i <interface> -p <protocol {udp/tcp} > -s <source> --dports <port> -j <target>
```

### طریقه کار iptables
وقتی به لینوکس ما کانکشنی برقرار شد یا اینکه کانکشن خروجی دیگری از سیستم عامل لینوکس به محیط بیرون بخواهد برقرار شود این کانکشن با لیست دستورات iptables بررسی شده و با توجه به پالیسی نوشته شده در فایروال اجازه ورود یا خروج پاکت راخواهد داد و همچنین لازم به ذکر است کلی دستورات به صورت خطی بررسی شده اگر کانکشن با یکی از آنها مطابقت داشت آن دستور در مورد آن کانکشن برقرار خواهد شد اگر به هیچ کدام از دستورات نوشته شده مطابقت نداشت از دستورات پیشفرض نوشته شده در iptable طبعیت خواهد کرد

![iptables](/images/iptables2.png)

### Iptables tables and chains

هسته لینوکس با استفاده از امکانات Netfilter به فیلتر کردن بسته، می پردازد و اجازه میدهد بعضی از بسته ها با استفاده از پسورد وارد شود و دیگر بسته اجازه وارد شدن نداشته باشند
که هسته آن از لینوکس درست شده که در آن پنج جدول از پیش طراحی شده که شامل:

**Filter:**

 این جدول به طور پیش فرض برای مدیریت بسته های شبکه است

**NAT:**

 برای تغییر بسته برای یک اتصال جدید کاربرد دارد مثلا تبدیل ای پی اینترنت به ای پی شبکه داخلی و برعکس مورد استفاده قرار میگیرد

**Mangle:**

 برای تغییر و مارک دار کردن انواع خاصی از بسته با توجه به پالیسی نوشته شده به کار میرود

**Raw:**

 عمدتا برای پیکربندی معافیت در طول ردیابی اتصال در ترکیب با هدف NOTRACK است

**Security:**

 برای کنترل دسترسی اجباری قوانین شبکه مورد استفاده قرار میگیرد

### Filter table

Input chain کلیه بسته های ورودی به سیستم عامل

Output chain کلیه بسته های خروجی از سیستم عامل

Forward chain کلیه بسته هایی که از طریق این سیستم عامل بخواهد فوروارد شود

### NAT table

Pre-routing بسته های شبکه را زمانی که آنها می رسند تغییر می دهد و معمولا برای destination network مورد استفاده قرار میگیرد

Output تغییر میدهد بسته های تولید شده در شبکه داخلی قبل از اینکه خارج شود

Post-routing تغییر می دهد بسته ها را قبل از اینکه از شبکه خارج شوند

### Mangle table

Input بسته های شبکه را برای میزبان تغییر می دهد (مارک دار می کند)

Output بسته های تولید شده از شبکه را قبل از فرستادن تغییر می دهد (مارک دار می کند)

Forward تغییر میدهد (مارک دار می کند) پکت های شبکه که می خواهد روت شود به نقطه دیگر

Pre-routing تغییر میدهد (مارک دار می کند) پکت های شبکه را قبل از اینکه روت شوند

Post-routing تغییر میدهد (مارک دار می کند) پکت های شبکه را بعد از اینکه روت شوند

### Raw table

Output اجرا پاکت های که در شبکه داخلی تولید شده اند قبل از اینکه خارج شوند

Pre-routing اجرای بسته های ورودی قبل از اینکه خارج شوند

### Security table

Input اجرا روی پاکت های که به شبکه وارد می شوند

Output اجرا روی پاکت های که از شبکه خارج می شوند

Forward اجرا روی پاکت های که از شبکه می خواهند روت شوند به جای دیگری

### Iptables command قوانین و دستورهای موجود در iptables

A- برای افزودن دستور به انتهای یک زنجیره از جدول قوانین استفاده می شود.

D- برای حذف دستور از مکان خاصی از زنجیره جدول قوانین استفاده می شود.

N- برای ایجاد زنجیر جدید مورد استفاده قرار می¬گیرد که میتوان به آن یک اسم اختصاصی بدهیم

E- برای تغییر اسم زیر شاخه ساخته شده به کار میرود

X- برای پاک کردن زیر شاخه تولید شده به کار می رود

F- برای پاک کردن قوانین مورد استفاده قرار می¬گیرد.

I- برای وارد کردن دستوری خاص به یک سطر از یک زیر شاخه

L- لیست کلیه دستورات یک زیر شاخه را لیست خواهد کرد

P- برای تغییر پالیسی دیفالت استفاده می شود

R- جایگزین کردن دستوری بهجای دستور قبلی در یک زیر شاخه

Z- برای صفر کردن کلیه کانتر های داخل فایروال

### Iptables parameter options

p- برای مشخص کردن نوع پروتکل مورد استفاده قرار می¬گیرد.

s- یا source– برای مشخص کردن شماره ip مبدا مورد استفاده قرار می¬گیرد.

d- یا destination– برای مشخص کردن شماره ip مقصد مورد استفاده قرار می¬گیرد.

i- یا in-interface– برای مشخص کردن کارت شبکه ورودی مورد استفاده قرار می¬گیرد.

o- یا out-interface– برای مشخص کردن کارت شبکه خروجی مورد استفاده قرار می¬گیرد.

j- مشخص کردن نحوه برخورد با بسته مورد استفاده قرار میگیرد. این سویچ به معنی jump بوده و به موارد DROP ، LOG ، ACCEPT و REJECT اشاره می کند.

m- برای مشخص کردن ماژول مورد استفاده قرار می گیرد.

c- این دستور شمارنده های داخل فایروال را فعال می کند که در طول وارد کردن یا اجرا کردن یا جایگزین کردن می باشد
[!] این دستور در ابتدای پارامتر های فوق دستور را به صورت منفی اجرا خواهد کرد

### Saving rules

```bash
iptables-save > /etc/iptables.rules
```
 برای پشتیبان گیری تنظیمات مورد استفاده قرار می گیرد.
 
```bash
post-down iptables-save > /etc/iptables.rules
```
برای پشتیبان گیری تنظیمات قبل از هر خاموش شدن سیستم مورد استفاده قرار می گیرد.

```bash
iptables-restore < /etc/iptables.rules
```
برای بازیابی تنظیمات مورد استفاده قرار می گیرد.

### مهم ترین و پرکاربردترین دستورات و تنظیمات iptables

پاک کردن تمام تنظیمات جاری
```
Iptables -F
```
ایجاد تنظیمات عمومی و مسدود کردن همه دسترسی ها
```
iptables -P INPUT DROP

iptables -P FORWARD DROP

iptables -P OUTPUT DROP
```
مسدود کردن یک ip خاص
```
iptables -A INPUT -s xxx.xxx.xxx.xxx -j DROP
```
باز کردن پورت SSH برای تمامی ارتباطات ورودی
```
iptables -A INPUT -i eth0 -p tcp --dport 22 -m state --state NEW,ESTABLISHED -j ACCEPT

iptables -A OUTPUT -o eth0 -p tcp --sport 22 -m state --state ESTABLISHED -j ACCEPT
```
باز کردن پورت ssh برای یک ip یا رنج ip خاص
```
iptables -A INPUT -i eth0 -p tcp -s xxx.xxx.xxx.xxx/24 --dport 22 -m state --state NEW,ESTABLISHED -j ACCEPT

iptables -A INPUT -i eth0 -p tcp -s xxx.xxx.xxx.xxx --dport 22 -m state --state NEW,ESTABLISHED -j ACCEPT

iptables -A OUTPUT -o eth0 -p tcp --sport 22 -m state --state ESTABLISHED -j ACCEPT
```
باز کردن پورت http
```
iptables -A INPUT -i eth0 -p tcp --dport 80 -m state --state NEW,ESTABLISHED -j ACCEPT

iptables -A OUTPUT -o eth0 -p tcp --sport 80 -m state --state ESTABLISHED -j ACCEPT
```
باز کردن پورت https
```
iptables -A INPUT -i eth0 -p tcp --dport 443 -m state --state NEW,ESTABLISHED -j ACCEPT

iptables -A OUTPUT -o eth0 -p tcp --sport 443 -m state --state ESTABLISHED -j ACCEPT
```
باز کردن چند پورت بصورت یکجا
```
iptables -A INPUT -i eth0 -p tcp -m multiport --dports 22,80,443 -m state --state NEW,ESTABLISHED -j ACCEPT

iptables -A OUTPUT -o eth0 -p tcp -m multiport --sports 22,80,443 -m state --state ESTABLISHED -j ACCEPT
```
باز کردن پورت برای ارتباط خروجی ssh
```
iptables -A OUTPUT -o eth0 -p tcp -d 192.168.101.0/24 --dport 22 -m state --state NEW,ESTABLISHED -j ACCEPT

iptables -A INPUT -i eth0 -p tcp --sport 22 -m state --state ESTABLISHED -j ACCEPT
```
باز کردن پورت https برای ارتباطات خروجی
```
iptables -A OUTPUT -o eth0 -p tcp --dport 443 -m state --state NEW,ESTABLISHED -j ACCEPT

iptables -A INPUT -i eth0 -p tcp --sport 443 -m state --state ESTABLISHED -j ACCEPT
```
ایجاد امکان ping از داخل به خارج
```
iptables -A OUTPUT -p icmp --icmp-type echo-request -j ACCEPT
```
ایجاد امکان ping از خارج به داخل
```
iptables -A INPUT -p icmp --icmp-type echo-request -j ACCEPT
```
ایجاد امکان دسترسی loopback
```
iptables -A INPUT -i lo -j ACCEPT

iptables -A OUTPUT -o lo -j ACCEPT
```
ایجاد امکان دسترسی به شبکه خارجی eth1 از شبکه داخلی eth0
```
iptables -A FORWARD -i eth0 -o eth1 -j ACCEPT
```
باز کردن دسترسی خروجی پورت dns
```
iptables -A OUTPUT -p udp -o eth0 --dport 53 -j ACCEPT

iptables -A INPUT -p udp -i eth0 --sport 53 -j ACCEPT
```
ذخیره تغییرات iptables
```
service iptables save
```
 فعال کردن IP Forwarding:
```
echo "1" > /proc/sys/net/ipv4/ip_forward
```
### فعال کردن LOG در قوانین فایروال

برای اینکه مدیریت کامل تری به شبکه خود داشته باشیم و بتوانیم منابعی که در حال اسکن کردن سیستم ما هستند بیابیم و یا در هر حال گزارشی از عملکرد صحیح فایر وال داشته باشیم می توانیم به طرق ذیل Log فایل ها را برای موارد دلخواهمان فعال کنیم:

فعال کردن Log برای دیدن بسته های ICMP:
```
iptables -A OUTPUT -p icmp -j LOG --log-prefix "PING:> "

iptables -A INPUT -p icmp -j LOG --log-prefix "PING:> "
```
برای دیدن این log ها به این مسیر بروید:

/var/log/messages

و خط هایی را که با PING
شروع شده اند بررسی نمایید.( البته راه ساده تر آن استفاده از دستور فیلتر کننده grep و مختص کردن به log های فایروال می باشد)

فعال کردن Log برای یک ip خاص:
```
iptables -t  nat POSTROUTING -s 192.168.0.88 -o eth1 -j LOG --log-prefix " "
```
برای جلوگیری از حجیم شدن لاگ فایل می توان از قابلیت سوکت -m در دستور استفاده کرد که به کمک آن می توان تنظیم نمود که برای مثال در هر 5 دقیقه بیش از 7 مورد را در  لاگ ذخیره ننماید:
```
iptables -A INPUT -i eth1 -s 10.0.0.0/8 -m limit –limit 5/m –limit-burst 7 -j LOG –log-prefix “IP_SPOOF A: “
```
نمایش وضعیت فایروال:
```
iptables -L -n -v
```
حذف قوانین و رول های فایروال :

ابتدا به کمک دستورات زیر شماره خط رول را بدست آورید:
```
iptables -L INPUT -n –line-numbers
iptables -L OUTPUT -n –line-numbers
iptables -L OUTPUT -n –line-numbers | less
iptables -L OUTPUT -n –line-numbers | grep 202.54.1.1
```
حال به عنوان مثال برای حذف رول موجود در خط شماره 4 می توانید از دستور زیر استفاده فرمائید:
```
iptables -D INPUT 4
```
و یا از دستور زیر برای حذف قوانین مروبطه به ای پی مورد نظر خود استفاده فرمایید:
```
iptables -D INPUT -s 202.54.1.1 -j DROP
```
اگر بدنبال راهنمای فایروال تنها برای یک دستور خاص هستید نیز سینتکس زیر استفاده فرمائید:
```
iptales -j DROP -h
```

