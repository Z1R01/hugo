 ---
title: "Ansible Role"
date: 2021-02-06T19:57:23+03:30
draft: true
toc: false
image: 'ansible.jpg'
tags:
  - Ansible
dir : "rtl"
---

Ansible ابزاری برای مدیریت و پیکربندی سرورها است که برای کنترل و اتوماسیون task ها برای ادمین ها و تیم های عملیاتی طراحی شده است. با Ansible می توانید از یک سرور مرکزی واحد برای کنترل و پیکربندی سیستم های مختلف از راه دور با استفاده از پروتکل SSH استفاده کنید.

### مقدمه ای بر ansible role : 

هرچه تعداد و تنوع سیستم ها بیشتر شود.مدیریت و کنترل node ها در ansible پیچیده تر می شود. اگر با ansible آشنا باشید منطقی است که باید task ها را با هم در playbook انسیبل تعریف کنیم. با استفاده از playbook ، دیگر نیازی به اجرای بسیاری از کارهای جداگانه روی سیستم های از راه دور نیست ، در عوض به شما امکان می دهد تمام محیط ها را همزمان با یک فایل پیکربندی کنید.

با این حال ، playbook ها هم میتوانند به مرور پیچیده و سخت شوند هنگامی که سیستم های متفاوت با task های مختلف تعریف شوند، بنابراین Ansible به شما این امکان را می دهد وظایف را در یک ساختار دایرکتوری به نام Role سازماندهی کنید.

 در این پیکربندی ، playbook ها به جای وظایف ، role ها را فراخوانی می کنند ، بنابراین می توانید وظایف را با هم گروه بندی کنید و سپس از role ها در سایر playbook ها دوباره استفاده کنید. role ها همچنین به شما امکان می دهند الگوها ، پرونده های ثابت و متغیرها را به همراه وظایف خود در یک قالب جمع آوری کنید.

بگذارید این را با یک مثال توضیح دهم. فرض کنید شما می خواهید playbook ای را ایجاد کنید که 10 وظیفه مختلف را بر روی 5 سیستم مختلف انجام دهد ، آیا برای این کار از یک playbook استفاده می کنید؟ خیر ، استفاده از یک playbook  گیج کننده و مستعد اشتباه است. در عوض ، می توانید 10 نقش مختلف ایجاد کنید ، جایی که هر نقش یک وظیفه را انجام دهد. سپس ، تنها کاری که شما باید انجام دهید این است که نام نقش را در داخل playbook ذکر کنید تا آنها را فراخوانی کنید.

### قابلیت استفاده مجدد از ansible role :

ansible role ها مستقل از یکدیگر می باشند و اجرای هر role وابسته به دیگر role ها نمی باشد لذا شما می توانید طبق خواسته و نیاز تان role ها را هر زمان تغییر و شخصی سازی کنید.
این کار  وظیفه ما را برای نوشتن مجدد یک بخش کامل از کد که هر بار به آن نیاز داریم کاهش می دهد ، در نتیجه کار ما ساده تر می شود.
برای مثال شما برای تنظیم LAMP stack باید یک playbook بنویسید و 4 نقش ایجاد کنید  اگر playbook دیگری برای تنظیم LAMP stack و همچنین نصب وردپرس بخواهید ، آیا دوباره role های جدیدی برای LAMP stack و WordPress ایجاد خواهید کرد؟ نه شما می توانید به سادگی از role های قدیمی (که برای LAMP stack ایجاد کردید) دوباره استفاده کنید و علاوه بر این role جدیدی را برای وردپرس ایجاد می کنید.

### ساختار دایرکتوری role ها :

 Ansible ویژگی به نام Ansible Galaxy را فراهم می کند که به شما کمک می کن تا با نقش ها بازی کنید. اگر ansible را نصب دارید به مسیر /etc/ansible بروید سپس وارد پوشه roles می شوید و اگر نبود آنرا ایجاد می کنید سپس برای ایجاد یک role جدید دستور زیر را می زنیم:

```
sudo ansible-galaxy init <role-name>

```
شما می توانید role های دیگری نیز در همین دایرکتوری ایجاد کنید.
حال با دستور tree دایرکتوری هایی که ایجاد شده است می توانیم مشاهده کنیم که در ادامه به آنها می پردازیم

![tree1](/images/tree1.png)

**Task**

شامل کلیه وظایفی است که قرار است توسط role ها اجرا شوند بدین معنی که تمام task های خودمان را در این مسیر تعریف می کنیم.

**Handlers**

شامل کاربرانی است که ممکن است توسط role جاری یا در هر جای دیگر از role جاری استفاده شود.

**Defaults**

شامل متغیرهای پیش فرضی است که قرار است توسط این role استفاده شود.

**Vars**

این دایرکتوری شامل متغیرهای دیگری است که قرار است توسط این role استفاده شوند. این متغیرها را می توان در playbook نیز تعریف کرد ، اما تعریف آنها در این بخش بهتر است.

**Files**

شامل کلیه فایل هایی است که می خواهید توسط این role ایجاد شوند. این شامل فایل هایی است که هنگام پیکربندی role باید به میزبانان ارسال شود.

**Meta**

metadate ها را برای این role تعریف می کند. اساساً ، فایل هایی است که وابستگی های role را ایجاد می کنند.

دقت داشته باشید که هر دایرکتوری باید از یک فایل main.yml تشکیل شود که در آن کد واقعی برای آن نقش خاص نوشته شده است.

![tree1](/images/tree2.png)

بیاید برای درک بهتر role ها در ansible با هم سناریویی را پیاده سازی کنیم.

### نصب MEAN Stack با ansible role :

برای پیاده سازی این سناریو نیاز به ایجاد سه role داریم.

1.نصب پیش نیاز ها

2.نصب MongoDB

3.نصب NodeJS

**قدم اول:**

به مسیر /etc/ansible/roles/ می رویم و دستورات زیر را می زنیم:

```
cd /etc/ansible/roles
sudo ansible-galaxy init prerequisites
sudo ansible-galaxy init mongodb
sudo ansible-galaxy init nodejs
```
خب پس از زدن دستورات بالا سه پوشه در دایرکتوری role ایجاد خواهد شد.


![tree1](/images/tree3.png)

**قدم دوم:**

نصب git برای role prerequisites با نوشتن در فایل /tasks/main.yml/ 

```
cd prerequisites/tasks/main.yml
---
- name: Install git
  apt:
     name: git
     state: present
     update_cache: yes
```
**قدم سوم:**

دستورات زیر را  در فایل main.yml برای MongoDB role اضافه می کنیم.

```
cd /mongodb/tasks/main.yml
---
- name: MongoDB - Import public key
  apt_key:
    keyserver: hkp://keyserver.ubuntu.com:80
    id: EA312927
 
- name: MongoDB - Add repository
  apt_repository:
    filename: '/etc/apt/sources.list.d/mongodb-org-3.2.list'
    repo: 'deb <a href="http://repo.mongodb.org/apt/ubuntu">http://repo.mongodb.org/apt/ubuntu</a> trusty/mongodb-org/3.2 multiverse'
  state: present
  update_cache: yes
 
- name: MongoDB - Install MongoDB
  apt:
    name: mongodb-org
    state: present
    update_cache: yes
 
- name: Start mongod
  shell: "mongod &"

```

**قدم چهارم:**

 دستورات زیر را در فایل main.yml برای nodjs role اضافه می کنیم.

```
cd nodejs/tasks/main.yml

---
- name: Node.js - Get script
  get_url:
    url: "<a href="http://deb.nodesource.com/setup_6.x">http://deb.nodesource.com/setup_6.x</a>"
    dest: "{{ var_node }}/nodejs.sh"
 
- name: Node.js - Set execution permission to script
  file:
    path: "{{ var_node }}/nodejs.sh"
    mode: "u+x"
 
- name: Node.js - Execute installation script
  shell: "{{ var_node }}/nodejs.sh"
 
- name: Node.js - Remove installation script
  file:
    path: "{{ var_node}}/nodejs.sh"
    state: absent
 
- name: Node.js - Install Node.js
  apt: name={{ item }} state=present update_cache=yes
  with_items:
    - build-essential
    - nodejs
 
- name: Node.js - Install bower and gulp globally
  npm: name={{ item }} state=present global=yes
  with_items:
    - bower
    - gulp

```
**قدم پنجم:**

خب پس از تکمیل شدن مراحل بالا نیاز داریم یک playbook ایجاد کنیم و role هارا در آن فراخوانی کنیم. پس به مسیر /etc/ansible/ می رویم و فایلی تحت عنوان playbook.yml ایجاد می کنیم و محتویات زیر را وارد می کنیم.

```
---
- hosts: nodes
  remote_user: ansible
  become: yes
  become_method: sudo
  vars:
    #variable needed during node installation
    var_node: /tmp
  roles:
      - prerequisites
      - mongodb
      - nodejs
```

پس از تکمیل مراحل بالا برای deploy کردن آنها نیاز داریم که playbook خودمان را برای سرور هایی که در inventory تعریف کریم اجرا کنیم. با استفاده از دستور زیر playbook خودمان را run می کنیم.

```
sudo ansible-playbook /etc/ansible/playbook.yml -K

```

![tree1](/images/tree4.png)

همانطور که می بینید ، همه task ها اجرا شده و وضعیت آنها تغییر کرده است. این بدان معناست که تغییرات playbook روی سرور و همچنین میزبان اعمال شده است.
 نصب MEAN Stack فقط یک مثال بود. شما می توانید با استفاده از ansible role به معنای واقعی کلمه هر چیزی را  در هر جایی تنظیم کنید.

**منابع :**

https://www.digitalocean.com/community/tutorials/how-to-use-ansible-roles-to-abstract-your-infrastructure-environment

https://www.learnitguide.net/2018/02/ansible-roles-explained-with-examples.html

https://www.edureka.co/blog/ansible-roles-setup-mean-stack

https://medium.com/@mitesh_shamra/ansible-roles-1d1954f9932a
