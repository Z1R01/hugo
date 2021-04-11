 ---
title: "HAproxy Detect Attack"
date: 2021-02-06T19:57:23+03:30
draft: true
toc: false
image: 'HAProxy.png'
tags:
  - haproxy
dir : "rtl"
---

HAproxy ابزاری قدرتمند است که در زمینه load balancing استفاده می شود. اما اگر بتوانید به درستی آن را کانفیگ کنید می توانید بسیاری از حملاتی که در بستر وب به سرور های شما می شود را با یک کانفیگ صحیح شناسایی کنید و سرورهای خودرا امن نگه دارید.

در این مستند قرار است درباره دستورات و تنظیمات درون HAproxy صحبت کنیم که به شما کمک شایانی برای حفظ امنیت سرورهای خود می کند.

دقت داشته باشید اگر آشنایی با این ابزار ندارید ابتدا به بررسی HAproxy بپردازید سپس ادامه مقاله را مطالعه نمایید.
برای جلوگیری از حملات DDOS در HAproxy می توانیم از آپشن های زیر بهره ببریم که در ادامه هرکدام از آنهارا بررسی میکنیم:

**tcp-request , http-request :**

 پرکاربردترین آپشن جهت برقراری امنیت HA استفاده از این آپشن به همراه پارامترهای آن می باشد که با چند مثال آنهارا بررسی می کنیم.
```
frontend mywebsite
    bind *:80
    http-request track-sc0 src table per_ip_rates
```
مطابق دستور بالا هر کلاینتی که به سرور ما متصل شود ip آن دنبال شده و داخل جدولی به **per_ip_rates** ذخیره می شود.
```
listen Secure_Web_Cluster
bind *:80 transparent
mode http
timeout http-request 5s #Slowloris protection
tcp-request connection accept if { src -f /etc/haproxy/whitelist.lst }
tcp-request content reject if { src -f /etc/haproxy/blacklist.lst }
```
مطابق دستور بالا کلاینت ها را به دو دسته سفید و سیاه تقسیم می کنیم و ip هایی که مجاز به اتصال می باشند را در یک لیست تعریف کرده و سپس مجوز دسترسی را در HAproxy برای آنها تعریف میکنیم. به همین ترتیب برای ip هایی که مشکوک هستند نیز می توانیم این کار را انجام دهیم.
```
tcp-request connection reject if { src_conn_rate(Abuse) ge 10 }
tcp-request connection reject if { src_conn_cur(Abuse) ge 10 }
tcp-request connection track-sc1 src table Abuse
```
اگر کانفیگ بالا را لحاظ نماییم به ip کلاینت کاری ندارد و اگر تعداد درخواست هایش بیش از 10 بار بود و ۱۰ اتصال بیشتر همزمان به سرور داشتیم را reject میکند.
```
use_backend bk_web_static if { path_end .jpg .png .gif .css .js }
default_backend bk_web
```
گاهی اوقات شما لازم دارید که فایل های استاتیک و از پیش تعریف شده شما مستقیما به یک سرور در بک اند متصل شوند در غیر اینصورت به بک اند دیگری که تعریف کرده اید متصل شوند. با قرار دادن کانفیگ بالا این کار محیاست.

تا بدینجا با کانفیگ های ساده در بهبود امنیت haproxy آشنا شدیم لذا اگر بخواهیم به صورت تخصصی به موضوع بپردازیم و تهدیدات را با الگو خاصی بلاک کنیم دانستن موارد زیر نیز ضروری می باشد.

بلاک کردن درخواست هایی که از پروتکل HTTP_1.0 استفاده می کنند.
```
http-request deny if HTTP_1.0
```
 
بلاک کردن درخواست هایی که از طریق مرورگر به سرور متصل نشده اند همچون ابزار curl:

```
http-request deny if { req.hdr(user-agent) -i -m sub curl }
```
 
گاهی مهاجم به نحوی درخواست ها را ارسال می کند که user agent ندارد که با دستور زیر می توانید جلوی اینکار را نیز بگیرید:
```
http-request deny unless { req.hdr(user-agent) -m found }
```
 
برای جلوگیری از لیستی از agent های مشکوک می توانیم از دستور زیر بهره ببریم:

```
http-request deny if { req.hdr(user-agent) -i -m sub -f /etc/haproxy/badagents.acl }
```

برای جلوگیری از لیستی از ip های مشکوک می توانیم از دستور زیر بهره ببریم:
```
http-request deny if { src -f /etc/hapee-1.8/blacklist.acl }
```
از متداول ترین آپشن های دیگر می توانیم به **timeout** و **Maxconn** اشاره کنیم.که با کاهش زمان آنها در بهبود امنیت سرور کمک بسزایی انجام دهیم.
