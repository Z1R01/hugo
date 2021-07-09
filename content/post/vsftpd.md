---
title: "مراحل راه اندازی FTP سرور"
date: 2021-02-08T19:57:23+03:30
draft: true
toc: false
image: 'vsftpd.jpg'
tags:
  - Linux
categories: [Linux]
dir : "rtl"
---

سرور FTP مخفف (File Transfer Protocol) است كه یک پروتکل شبکه استاندارد بوده و برای انتقال فايل ها به شبکه از راه دور استفاده می شود. برای انتقال امن تر و سریع تر داده ها ، از SCP یا SFTP استفاده کنید. بسیاری از سرورهای منبع باز FTP برای لینوکس در دسترس هستند. محبوب ترین و پرکاربردترین آنها PureFTPd ، ProFTPD و vsftpd است.

در این مقاله ، نحوه نصب و پیکربندی سرور FTP با VSFTPD در Debian را آموزش مي دهيم.  ما همچنین به شما نشان خواهیم داد که چگونه vsftpd را پیکربندی کنید تا کاربران را در فهرست خود محدود کنید و کل انتقال را با SSL / TLS رمزگذاری کنید.

برای نصب بسته vsftpd دستور زیر را وارد می‌کنیم.

```bash
sudo apt-get install vsftpd  
```
### پیکربندی vsftpd
سرور vsftpd را می توان با تغییر فايل vsftpd.conf ، که در فهرست / etc یافت می شود ، پیکربندی کرد.

بسیاری از تنظیمات در فايل پیکربندی به خوبی ثبت شده اند. برای همه گزینه های موجود به صفحه رسمی vsftpd مراجعه کنید.

در بخش های بعدی برخی از تنظیمات مهم مورد نیاز برای پیکربندی نصب امن vsftpd را مرور خواهیم کرد.

با باز کردن فايل پیکربندی vsftpd شروع کنید:
```bash
sudo nano /etc/vsftpd.conf
```
### دسترسی FTP
دستورالعمل های Anonymous_enable و local_enable را پیدا کنید و مطابقت پیکربندی خود را با خطوط زیر بررسی کنید:
```bash
anonymous_enable=NO
local_enable=YES
```
این تضمین می کند که فقط کاربران محلی می توانند به سرور FTP دسترسی پیدا کنند.

### فعال کردن بارگذاری ها
تنظیمات write_enable را لغو کنید تا تغییراتی در سیستم فایل مانند ، آپلود و حذف فايل ها انجام شود.
```bash
write_enable=YES
```
برای جلوگیری از دسترسی کاربران FTP به هر فايلي در خارج از دایرکتوری خود دستور زیر را اضافه می کنیم:

```bash
chroot_local_user=YES
```
به طور پیش فرض برای جلوگیری از آسیب پذیری امنیتی ، هنگامی که chroot فعال است ، vsftpd در صورت نوشتن فهرستی که کاربران قفل شده اند ، از آپلود فايل خودداری می کند.

از یکی از روشهای زیر استفاده کنید تا دانلود در هنگام فعال شدن chroot آپلود شود.

#### روش اول

روش پیشنهادی برای اجازه آپلود ، فعال کردن chroot و پیکربندی دایرکتوری های FTP است.
```bash
user_sub_token=$USER
local_root=/home/$USER/ftp
```
#### روش دوم

گزینه دیگر اضافه کردن بخشنامه زیر در فايل پیکربندی vsftpd است. اگر می خواهید به کاربر دسترسی پیدا کنید به فهرست خانه خود رفته ، از این گزینه استفاده کنید.

```bash
allow_writeable_chroot=YES
```
به‌صورت پیش‌فرض vsftpd برای سرویس‌دهی روی پروتکل IPv6 پیکربندی شده است. پس در صورتی که در شبکه از IPv6 استفاده نمی‌کنید می‌بایست به‌صورت زیر عمل کنید.

```bash
listen=YES  
listen_ipv6=NO
```
### محدود کردن ورود کاربر
برای ورود فقط برخی از کاربران به سرور FTP ، خطوط زیر را در انتهای فايل اضافه کنید:

```bash
userlist_enable=YES
userlist_file=/etc/vsftpd.user_list
userlist_deny=NO
```
وقتی این گزینه فعال شد ، باید صریحاً مشخص کنید که کاربران با اضافه کردن نامهای کاربر به فايل /etc/vsftpd.user_list (یک کاربر در هر خط) می توانند وارد سیستم شوند.

### انتقال با SSL / TLS
برای رمزگذاری انتقال FTP با SSL / TLS ، برای استفاده از آن باید یک گواهی SSL داشته باشید و سرور FTP را پیکربندی کنید.

می توانید از یک گواهی SSL موجود که توسط یک مجوز معتبر گواهی امضا شده استفاده شده یا یک گواهی خود امضا شده استفاده کنید.

اگر دامنه یا زیر دامنه ای دارید که به آدرس IP سرور FTP ، می توانید به راحتی یک مجوز رمزگذاری SSL رایگان ایجاد کنید.

در این آموزش ، ما با استفاده از دستور opensl یک گواهی SSL خود امضا تولید می کنیم.

دستور زیر یک کلید خصوصی ۲۰۴۸ بیتی و گواهی خود امضا شده برای ۱۰ سال ایجاد می کند. کلید خصوصی و گواهینامه در یک فايل ذخیره می شوند:
```bash
sudo opensssl req -x509 -nodes -days 3650 -newkey rsa:2048 -keyout /etc/ssl/private/vsftpd.pem -out /etc/ssl/private/vsftpd.pem
```
پس از ایجاد گواهینامه SSL پرونده پیکربندی vsftpd را باز کنید و دستورالعمل های rsa_cert_file و rsa_private_key_file را پیدا کنید ، مقادیر آنها را در مسیر فایل pam تغییر دهید و دستورالعمل ssl_enable را در YES تنظیم کنید:
```bash
sudo nano /etc/vsftpd.conf
---
rsa_cert_file=/etc/ssl/private/vsftpd.pem
rsa_private_key_file=/etc/ssl/private/vsftpd.pem
ssl_enable=YES
```
اگر در غیر این صورت فرايند پيش رفت ، سرور FTP برای برقراری اتصالات ایمن فقط از TLS استفاده می کند.

پس از انجام ویرایش ، فایل پیکربندی vsftpd (به استثنای نظرات) باید چیزی شبیه به این باشد:

```bash
listen=YES
listen_ipv6=NO
anonymous_enable=NO
local_enable=YES
write_enable=YES
dirmessage_enable=YES
use_localtime=YES
xferlog_enable=YES
connect_from_port_20=YES
chroot_local_user=YES
secure_chroot_dir=/var/run/vsftpd/empty
pam_service_name=vsftpd
rsa_cert_file=/etc/ssl/private/vsftpd.pem
rsa_private_key_file=/etc/ssl/private/vsftpd.pem
ssl_enable=YES
user_sub_token=$USER
local_root=/home/$USER/ftp
userlist_enable=YES
userlist_file=/etc/vsftpd.user_list
userlist_deny=NO
```
فایل را ذخیره کرده و سرویس vsftpd را مجدداً برای تغییر اعمال کنید:

```bash
sudo systemctl restart vsftpd
```
### ایجاد کاربر FTP
برای تست سرور FTP ما یک کاربر جدید ایجاد خواهیم کرد.

اگر قبلاً یک کاربر دارید که می خواهید دسترسی FTP را اعطا کنید ، اولین مرحله را پشت سر بگذارد.

اگر اجازه دهید writeable_chroot = YES را در فايل پیکربندی خود قرار دهید ، از انجام مرحله ۳ صرف نظر كنيد.

۱) کاربر جدیدی بنام FTP_USER ایجاد کنید:
```bash
sudo adduser FTP_USER
```
۲) کاربر را به لیست کاربران مجاز FTP اضافه کنید:

```bash
echo "FTP_USER" | sudo tee -a /etc/vsftpd.user_list
```
۳) پوشه FTP را ایجاد کنید و مجوزهای صحیح را تنظیم کنید:

```bash
sudo mkdir -p /home/FTP_USER/ftp/upload
sudo chmod 550 /home/FTP_USER/ftp
sudo chmod 750 /home/FTP_USER/ftp/upload
sudo chown -R FTP_USER: /home/FTP_USER/ftp
```
همانطور که در بخش قبلی بحث شد کاربر قادر خواهد بود فايل های خود را در فهرست upload/ آپلود کند.

در این مرحله ، سرور FTP شما کاملاً کاربردی است و شما باید بتوانید با استفاده از هر كاربر FTP که برای استفاده از رمزگذاری TLS مانند FileZilla پیکربندی شده است ، به سرور خود متصل شوید.

### غیرفعال کردن دسترسی Shell

به طور پیش فرض ، هنگام ایجاد کاربر ، اگر صریحا مشخص نشده باشد ، کاربر SSH به سرور دسترسی خواهد داشت.

برای غیرفعال کردن دسترسی به پوسته ، ما یک پوسته جدید ایجاد خواهیم کرد که به سادگی پیامی را چاپ خواهد کرد که به کاربر می گوید حساب آنها فقط به دسترسی FTP محدود است.

```bash
echo -e '#!/bin/sh\necho "This account is limited to FTP access only."' | sudo tee -a  /bin/ftponly

sudo chmod a+x /bin/ftponly

sudo usermod FTP_USER -s /bin/ftponly
```
برای تغییر پوسته کلیه کاربرانی که می خواهید فقط به FTP دسترسی داشته باشید ، از همان دستور استفاده کنید.

