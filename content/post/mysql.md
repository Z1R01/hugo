 ---
title: "Mysql Architecture"
date: 2021-01-06T19:57:23+03:30
draft: true
toc: false
image: 'mysql_arch.jpg'
tags:
  - Database
categories: [Database]
dir : "rtl"
---
معماری MySQL چگونگی ارتباط مؤلفه های مختلف با یکدیگر را توصیف می کند. معماری MySQL اساساً یک سیستم کلاینت سروری است و در حالت کلی می توان معماری mysql را به دو بخش تقسیم کرد که شامل:

۱− معماری منطقی: به نحوه برقراری ارتباط بین کلاینت و سرور mysql می پردازد.

۲− معماری فیزیکی: به نحوه ذخیره سازی داده ها بر روی دیسک های سخت می پردازد.

قبل از پرداختن به معماری mysql نیاز است که اجزای تشکیل دهنده آن را بهتر بشناسیم و آن ها را بررسی کنیم تا در ادامه راحت تر بتوانیم کارمان را پیش ببریم.
لذا اجزای پایگاه داده mysql به قرار زیر است:

### فایل های پیکربندی:

#### auto.cnf:
 این فایل شامل uuid سرور می باشد.
#### my.cnf:
 مهمترین فایل mysql برای پیکربندی فایل ها می باشد.
توجه داشته باشید پس از اعمال تغییرات در این فایل ها نیاز است که سرویس mysql را با دستور مقابل راه اندازی مجدد کنیم.
```
systemctl restart mysql
```
در ادامه به بررسی فایل هایی می گردیم که پس از ایجاد تغییرات و ساخت جداول و دیتابیس ایجاد می شود.
لذا برای بررسی این فایل ها باید به مسیر **/var/lib/mysql** برویم در آنجا مشاهده خواهید کرد که فایل ها و دیتابیس های ایجاد شده موجود است. که درباره هرکدام از آنها توضیح خواهیم داد:

#### فایل های frm. : 

FRM یک قالب برای فرمت داده ها است که MySQL استفاده می کند. FRM مخفف FoRMat است و از فایل های  FRM برای تعریف قالب جداول استفاده می شود. MySQL یک پایگاه داده رابطه ای متقابل است بدین معنی که بازای هر جدولی که در mysql تعریف می شود یک فایل با پسوند FRM. ساخته خواهد شد.

#### فایل های par. :

اگر برای دیتابیس خودتان عملیات پارتیشن بندی را انجام دهید سرویس mysql فایلی با پسوند par. ایجاد می کند که مشخص می کند برای جداول به چه صورت پارتیشن بندی صورت گرفته است. دیتابیس این فایل هارا فقط در حالت خواندنی قرار می دهد و قابلیت تغییر در فایل های par. وجود ندارد.

#### فایل های myd. :

فایل هایی با پسوند myd شامل کلیه داده هایی می باشد که در دیتابیس شما ذخیره شده است و مختصر عبارت mysql data می باشد. این فایل ها بصورت باینری می باشند و تمام رکورد های جداول mysql در آن ذخیره شده است.


#### فایل های ibd.
در این فایل ها جداول دیتابیس برای موتور جست و جو InnoDB وجود دارد و اگر جدولی در دیتابیس شما وجود داشته باشد که از موتور InnoDB استفاده می کند mysql فایلی با پسوند ibd. برای آن جدول ایجاد می کند.
متغیری تحت عنوان innodb_file_per_table برای موتور InnoDB وجود دارد که اگر فعال باشد بازای هر tablespaces یک فایل ibd ایجاد می کند که مجموع این فایل ها یک دیتابیس واحد را تشکیل می دهد.
 این فایل ها قابلیت باز شدن و  کانورت شدن را ندارند و درصورت بروز مشکل فقط شما می توانید دیتابیس خود را به نحوی طراحی کنید که این فایل ها را بشناسد.
موقع بک آپ گیری  از این فایل ها باید توجه داشت که پسوند ibd. به ibz. تغییر می کند.

#### فایل های myi. :

اگر از موتور ذخیره ساز MyISAM در جداول خود استفاده کرده باشید حتما چنین فایلی را مشاهده خواهید کرد. این فایل ها شامل index هایی می باشد که موتور ذخیره ساز MyISAM برای بررسی آنها از این فایل استفاده می کند. فایل هایی با فرمت myi با SQLite و microsoft SQL نیز قابل باز شدن هستند.


### انواع فایل های لاگ در mysql:

#### Error log:
 بطور پیش فرض در mysql این قابلیت فعال است و اتفاقاتی از قبیل از کار افتادن دستگاه و هر نوع خرابی و رویداد های غیر منتظره را در سه سطح note,warning,error تعریف میکند.
```
Variable : log_error

log-error = mysqld.log 
```
#### General log:
 بطور پیش فرض این دستور لاگ غیرفعال است و ما با فعال کردن آن می توانیم از کویری هایی که کاربر سمت سرور ارسال می کند لاگ بگیریم و برای خطایابی سمت کلاینت بسیار کاربردی است.
```
Variables : general_log & general_log_file
```
#### slow query log:
  بطور پیش فرض این دستور لاگ غیرفعال است و ما با فعال کردن آن می توانیم از مدت زمان طول کشیدن و آستانه اجرای دستورات در sql بصورت ثانیه ای باخبر شویم.
```
Variables : slow_query_log & slow_query_log_file
```
#### binary log:
 با نام binlog هم شناخته می شود و شامل رویدادهایی می شود که باعث تغییرات در داده ها و ایجاد جدول ها می شود. این قابلیت بصورت پیش فرض غیرفعال است و اگر از mysql replication بر روی سرور های خود استفاده می کنید حتما این قابلیت را فعال کنید.
برای مشاهده لاگ های باینری binlog ما می توانیم از ابزار mysqlbinlog استفاده کنیم که فایل های باینری را در قالب متن برای ما نشان می دهد.
```
Variables : log_bin
```
نکته : برای فعالسازی هرکدام از لاگ های mysql نیاز است که نام variable را در فایل my.cnf تعریف کنیم. برای مثال برای فعالسازی binlog در mysql مراحل زیر را انجام می دهیم:

![mysql](/images/mysql.png)

برای بررسی اینکه آیا binlog بر روی mysql فعال است دستور بالا را وارد می کنیم. مشاهده می کنید که binlog فعال نیست.
لذا به مسیر my.cnf رفته و مطابق شکل کد های زیر را وارد می کنیم:

![mysql](/images/mysql2.png)

در انتها با یکبار ریستارت کردن mysql قطعه کد زیر را مطابق شکل وارد می کنیم:

![mysql](/images/mysql3.png)

مشاهده خواهید کرد که logbin بر روی سرور mysql فعال شده است و اگر به مسیر /var/lib/mysql بروید تغییرات جدید اعمال شده است.

### معماری Mysql :

![mysql](/images/mysql4.png)

#### لایه اپلیکیشن:

این لایه بالاترین لایه در معماری MySQL است. این لایه شامل برخی از سرویس هایی است که برای اکثر برنامه های کلاینت - سروری مشترک است ، برخی از این سرویس ها شامل:

###### Connection Handling :

هنگامی که یک کلاینت به سرور متصل می شود ، کلاینت گره اتصال خود را از سرور می گیرد و کلیه دستورات  مربوط به آن کلاینت در همان گره مشخص شده اجرا می شود. این گره توسط سرور ذخیره می شود ، بنابراین نیازی به ایجاد و از بین بردن هر اتصال جدید نیست.
###### Authentication :
هرگاه کلاینت به یک سرور MySQL متصل شود ، سرور احراز هویت را انجام می دهد. که این احراز هویت بر اساس نام کاربری ، میزبان و رمز عبور کاربر انجام می شود.
###### Security :
پس از اتصال کلاینت به سرور MySQL ، سرور بررسی می کند که آیا این کلاینت خاص دارای امتیاز صدور پرس و جوهای خاص در برابر سرور MySQL است یا نه. با دستور show privileges در محیط mysql می توانید از سطح دسترسی ها آگاه شوید.
###### لایه سرور mysql :
این لایه شامل کلیه عملکردهای منطقی در سیستم مدیریت پایگاه داده رابطه ای MySQL می باشد. مغز سرور MySQL در این لایه ساکن است. لایه منطقی MySQL به اجزای فرعی مختلفی تقسیم می شود که در زیر آورده شده است:
###### MySQL services and utilities
 یکی از دلایل اصلی محبوبیت MySQL خدمات گسترده ای است که ارائه می دهد. این لایه خدمات و برنامه های کاربردی را برای مدیریت و نگهداری سیستم MySQL فراهم می کند که برخی از آنها در زیر ذکر شده است:
 ```
Backup & recovery
Security
Replication
Cluster
Partitioning
Workbench
```
SQL Interface
یک زبان پرس و جو است که برای  استفاده از  کویری ها به سرور MySQL استفاده می شود. این ابزار برای تعامل بین کاربر MySQL و سرور است.

 برخی از مؤلفه های رابط SQL شامل:
```
Data Manipulation Language (DML)
Data Definition Language (DDL)
Stored Procedures
Views
Triggers
```

###### SQL Parser
برای اینکه کویری های کاربر برای mysql قابل فهم باشد نیاز به یک sql parser داریم که پرس و جو کاربر را می شکند (درخت تجزیه) و آن ها را کام‍پایل می کند که ساختار داخلی آن به صورت زیر می باشد:

 در مرحله اول ، هنگام تجزیه گزاره های منظم ، تجزیه و تحلیل واژگانی(ایجاد کلمات) اجرا می شود. سپس تجزیه و تحلیل نحو (ایجاد 'جملات') اجرا می شود. سپس تجزیه و تحلیل معنایی (اطمینان از اینکه جمله ها معنی دارد) اجرا و در انتها تولید کد (برای کامپایلرها)  صورت می گیرد.

Optimizer
پس از ایجاد درخت تجزیه داخلی ، MySQL انواع تکنیک های بهینه سازی را اعمال می کند. این تکنیک ها ممکن است شامل  بازنویسی پرس و جو ، ترتیب اسکن جداول و انتخاب شاخص های مناسب برای استفاده باشد. در واقع می توانید از سرور بخواهید که جنبه های مختلف بهینه سازی را توضیح دهد.

```
EXPLAIN SELECT * FROM test;
```

Caches & buffers
حافظه کش MySQL مجموعه کامل نتایج را برای عبارات مختلف ذخیره می کند. حتی قبل از تجزیه و تحلیل پرس و جو ، سرور MySQL حافظه کش پرس و جو را بررسی می کند. اگر هر کلاینت یک پرس و جو را که مشابهش در حافظه کش موجود باشد صادر کند ، سرور به سادگی از تجزیه ، بهینه سازی و حتی اجرای آن صرفنظر می کند و فقط خروجی را از حافظه کش نمایش می دهد.

##### لایه موتور ذخیره سازی:
ویژگی موتور ذخیره سازی قابل تعویض ، MySQL را به عنوان گزینه ای بی نظیر و برتر برای اکثر توسعه دهندگان تبدیل می کند که object های mysql را در سطوح فیزیکی [ذخیره سازی] و منطقی [اجرای] تعریف می کند. موتور ذخیره سازی مسئول اجرای دستورات واکشی داده ها است.

SQL 
MySQL به ما این امکان را می دهد که موتورهای ذخیره سازی متنوعی را برای شرایط مختلف متناسب با نیازمان انتخاب کنیم. لیست موتورهای ذخیره سازی پشتیبانی شده در زیر ذکر شده است.
```
MyISAM
InnoDB
Federated
Blackhole
CSV
Memory
Archive
```

 شما می توانید از موتورهای مختلف ذخیره سازی در جدول های گوناگون استفاده کرد. یک دیتابیس می تواند شامل جداول با موتورهای ذخیره سازی چندگانه باشد.
 ```
mysql>SHOW ENGINES \G;
```

این فرمان تمام موتورهای ذخیره سازی پشتیبانی شده توسط سرور شما را فهرست می کند.

از MySQL 5.5 به بعد ، موتور ذخیره سازی پیش فرض InnoDB است. موتور ذخیره سازی پیش فرض برای MySQL قبل از نسخه 5.5  MyISAM بود. انتخاب موتور مناسب ذخیره سازی یک تصمیم استراتژیک مهم است که بر توسعه و آینده نگری پایگاه داده تان  تأثیر خواهد گذاشت. در ادامه به موتورهای ذخیره سازی MyISAM، InnoDB ، Memory و CSV اشاره خواهیم کرد. 

###### InnoDB
 پرکاربردترین موتور ذخیره سازی با پشتیبانی از تراکنش ها است. این موتور ذخیره ساز سازگار با ACID است. این موتور برای بازیابی خرابی های دیتابیس و کنترل همروند در چند نسخه را پشتیبانی می کند. این تنها موتوری است که یکپارچگی پایگاه داده را با کلیدی خارجی فراهم می کند. اوراکل از موتور InnoDB برای جداولش به جز موارد خاصی استفاده می کند.
###### MyISAM
 موتور ذخیره ساز قدیمی است و یک موتور ذخیره ساز سریع با سرعت بالا است زیرا از تراکنش ها پشتیبانی نمی کند. MyISAM از table-level locking استفاده می کند که بیشتر در ذخیره سازی داده ها در وب استفاده می شود.
###### Memory
 موتور ذخیره ساز است که جداول را در حافظه ایجاد می کند. این سریعترین موتور ذخیره ساز است و از تراکنش ها پشتیبانی نمی کند. موتور ذخیره ساز Memory برای ایجاد جداول موقت با جستجوی سریع ایده آل است که با راه اندازی مجدد پایگاه داده ، داده ها از بین می روند.
###### CSV
 داده ها را در پرونده های CSV ذخیره می کند. این انعطاف پذیری عالی باعث می شود که  داده ها در این قالب به راحتی در برنامه های دیگر ادغام می شوند.
###### Archive
 موتور ذخیره سازی برای درج با سرعت بالا بهینه شده است که داده ها را همزمان با درج فشرده سازی نیز می کند. این موتور از تراکنش ها پشتیبانی نمی کند و ایده آل برای ذخیره و بازیابی مقادیر زیادی از داده های بایگانی شده که ندرت ارجاع شده است می باشد.
###### Blackhole
 موتور ذخیره سازی است که  داده ها را فقط می پذیرد اما ذخیره نمی کند و بازیابی ها همیشه یک مجموعه خالی را برمی گردانند. این قابلیت را می توان در طراحی پایگاه داده توزیع شده استفاده کرد که در آن داده ها بطور خودکار تکثیر می شوند ، اما به صورت محلی ذخیره نمی شوند. این موتور ذخیره سازی می تواند برای انجام تست های عملکرد یا آزمایش های دیگر استفاده شود.
###### Federated
 موتور ذخیره سازی است که  امکان جداسازی سرورهای MySQL را برای ایجاد یک بانک اطلاعاتی منطقی از بسیاری از سرورهای فیزیکی ارائه می دهد. دستوراتی که در سرور محلی اجرا می شود به طور خودکار در جداول از راه دور فدرال اجرا می شود لذا هیچ داده ای روی جداول محلی ذخیره نمی شود که مناسب برای محیط های توزیع شده است.

#### نحوه تغییر دادن موتور های ذخیره ساز برای جداول mysql :
برای اینکار ابتدا برای بررسی اینکه از چه نوع  موتور ذخیره سازی استفاده می کند مطابق شکل دستور زیر را بر روی جدول مورد نظر خودمان وارد می کنیم:

![mysql](/images/mysql5.png)

مطابق شکل می بینید که جدول user از موتور MyISAM استفاده می کند. برای تغییر موتور ذخیره ساز کافی است که دستور زیر را وارد کنیم:
```
mysql> ALTER TABLE user ENGINE=’InnoDB’;
```

مطابق دستور بالا موتور ذخیره ساز از MyISAM به InnoDB تغییر می کند.