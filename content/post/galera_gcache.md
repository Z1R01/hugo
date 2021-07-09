 ---
title: "Galera Gcache چیست؟"
date: 2021-02-05T19:57:23+03:30
draft: true
toc: false
image: 'Galera.jpg'
tags:
  - Database
categories: [Database]
dir : "rtl"
---

در این مقاله قرار است عمیق تر به ساختار galera cluster بپردازیم. قبل از اینکه بخواهیم با مفهومی تحت عنوان gcache آشنا شویم. نیازمند این هستیم که با چند اصطلاح و روندکاری در galera آشنا شویم.


در هنگام کلاسترینگ دیتابیس بین تمام node ها galera  برای انتقال دیتای خود از دو روش زیر استفاده می کند.

```
1- SST: State Snapshot Transfer

2- IST: Incremental State Transfer
```

## SST:

در این روش گلرا برای sync کردن و برقرای replication دیتا بین node اهدا کننده یا مستر به node دریافت کننده یک کپی از تمام دیتا موجود خود می گیرد سپس عملیات انتقال را انجام می دهد این فرایند در پس زمینه به طور خودکار انجام می شود و روند کاری همانند پشتیان گیری بین slave و master است.
این روش در بعضی اوقات می تواند خطرناک باشد زیرا هنگام انتقال node اهداکننده را قفل می کند و فشار زیادی هنگام بک آپ گیری و اتصال به node ها انجام می شود. همچنین اگر دیتا شما حجم سنگینی داشته باشد و بخواهید انتقال را بر رروی wan انجام دهید استفاده از این روش توصیه نمی شود.

![tree1](/images/gcach.png)

خب اما برای جلوگیری از این موضوع باید چه کنیم؟ پس با ما همراه باشید.

### IST:

برای بهبود این وضعیت از روش IST و gcache استفاده می کنیم. IST روشی است که برای انتقال دیتا به node joiner فقط مقادیر نوشتاری گمشده بین دو node را ارسال میکند.

 برای اینکار فایلی تحت عنوان gcache وجود دارد که یک کپی از تمام نوشتار های موجود را درخود ذخیره می کند.
IST بسیار سریعتر از SST می باشد زیرا هیچگونه تغییرات performanc در node اهدا کننده ایجاد نمی شود و هنگام انتقال می توان بر روی گره rw را انجام داد.
برای پیاده سازی به روش IST نیازمند این هستیم که انتخاب درست و آگاهانه از مقدار سایز gcache داشته باشیم.

### بررسی gache_size :

گلرا به صورت پیش فرض از مقدار 128 مگابایت برای ذخیره در gcache استفاده میکند و این وظیفه یک DBA است که با توجه به دارایی و کارکرد دیتابیس خود مقدار صحیح برای gcache انتخاب کند.
برای اینکار می توانیم با زدن قطعه کد زیر در کنسول mysql به انتخاب آگاهانه تر دست یابیم:

```
mysql> set @start := (select sum(VARIABLE_VALUE/1024/1024) from information_schema.global_status where VARIABLE_NAME like 'WSREP%bytes'); do sleep(60); set @end := (select sum(VARIABLE_VALUE/1024/1024) from information_schema.global_status where VARIABLE_NAME like 'WSREP%bytes'); set @gcache := (select SUBSTRING_INDEX(SUBSTRING_INDEX(@@GLOBAL.wsrep_provider_options,'gcache.size = ',-1), 'M', 1)); select round((@end - @start),2) as `MB/min`, round((@end - @start),2) * 60 as `MB/hour`, @gcache as `gcache Size(MB)`, round(@gcache/round((@end - @start),2),2) as `Time to full(minutes)`;
 
+--------+---------+-----------------+-----------------------+
| MB/min | MB/hour | gcache Size(MB) | Time to full(minutes) |
+--------+---------+-----------------+-----------------------+
|   7.95 |  477.00 |  128        	|             	16.10 |
+--------+---------+-----------------+-----------------------+

```
همانطور که مشاهده می کنید یک node گلرا می تواند تقریبا 16 دقیقه بدون درخواست SST برای join خاموش و از دسترس خارج باشد. اگر این زمان بسیار کوتاهی است و فضا کافی بر روی node های خود دارید می توانید آنرا تغییر دهید.

```
wsrep_provider_options=”gcache.size=<value>” 
```
برای مثال اگر مقدار را به 1 گیگابایت تغییر دهیم  تا 2 ساعت می تواند node در حالت downtime باشد تا از روش IST هنگام rejoin شدن استفاده کند.

![tree1](/images/gcach2.png)

مطابق شکل همانطور که مشاهده میفرمایید اگر در هنگام اتصال node اهدا کننده به node متصل شونده مقادیر writeset داخل gcache سایز موجود باشد آنگاه از روش IST برای انتقال ارسال می کند و اگر gcache مقادیر موجود نباشد آنگاه از روش SST استفاده می کند

### GCache دارای سه نوع فضای ذخیره سازی است:

**Permanent In-Memory Store**

 در اینجا مجموعه فایل های نوشتاری در  حافظه پیش فرضی که برای سیستم عامل اختصاص می یابد ذخیره می شود. این روش در سیستم هایی که حافظه RAM اضافی دارند مفید است. 
به طور پیش فرض این روش غیر فعال است.
 
**Permanent Ring-Buffer**

 گلرا از این روش بصورت پیش فرض برای ذخیره سازی استفاده می کند در این روش مقدار حافظه را از قبل تعیین می کنید و اندازه آن 128 مگابایت است. 

```
wsrep_provider_options="gcache.size=number”
```
**On-Demand Page Store**

 در اینجا مجموعه فایل های نوشتاری به فایلهای صفحه  به صفحه تقسیم می شود. به طور پیش فرض ، اندازه آن 128 مگابایت است و تعداد صفحه هارا می توان تعیین کرد` که بسته به نیاز می توانیم مقدار آن را افزایش دهیم. Galera Cluster پرونده های صفحه را هنگام استفاده حذف می کند ، اما می توانید برای نگهداری از اندازه کلی پرونده های صفحه محدودیت تعیین کنید.

```
wsrep_provider_options="gcache.recover=yes" wsrep_provider_options="gcache.page_size=1G"
```

### انتخاب درست gcache size :

برای اینکار ابتدا نیاز است که مقادیر ورودی در برحسب بایت را در هر دقیقه بر روی سرویس mysql خود محاسبه کنیم. بدین منظور دستورات زیر را در mysql وارد می کنیم.
```
wsrep_replicated_bytes
```
مجموع writeset های ارسالی را بر حسب بایت نشان می دهد.
```
wsrep_received_bytes
```
مجموع writeset های دریافتی را بر حسب بایت نشان می دهد.

```

mysql> show global status like 'wsrep_received_bytes'; 
show global status like 'wsrep_replicated_bytes'; 
select sleep(60); 
show global status like 'wsrep_received_bytes'; 
show global status like 'wsrep_replicated_bytes';

+----------------------+----------+
| Variable_name        | Value    |
+----------------------+----------+
| wsrep_received_bytes | 83976571 |
+----------------------+----------+
+------------------------+-------+
| Variable_name          | Value |
+------------------------+-------+
| wsrep_replicated_bytes | 0     |
+------------------------+-------+

[...]

+----------------------+----------+
| Variable_name        | Value    |
+----------------------+----------+
| wsrep_received_bytes | 90576957 |
+----------------------+----------+
+------------------------+-------+
| Variable_name          | Value |
+------------------------+-------+
| wsrep_replicated_bytes | 800   |
+------------------------+-------+

```
مطابق مثال بالا برای محاسبه تعداد بایت ها در هر دقیقه داریم:

```
(second wsrep_received_bytes – first wsrep_received_bytes) + (second wsrep_replicated_bytes – first wsrep_replicated_bytes)

(90576957 – 83976571) + (800 – 0) = 6601186 bytes or 6 MB per minute.

```
و برای محاسبه تعداد بایت ها در هر ساعت داریم :

```
6MB * 60 minutes = 360 MB per hour of writesets received by the cluster.
```
بنابر این طبق مثال بالا اگر می خواهید یک ساعت از گره را نگهداری کنید (یا زمان خاموش) ، باید gcache را به360 مگ افزایش دهید و اگر زمان بیشتری می خواهید ، فقط باید  آن را بزرگتر کنید که برای اینکار قطعه کد زیر را در فایل تنظیمات galera اضافه می کنیم.

```
wsrep_provider_options="gcache.size=1G"
```
