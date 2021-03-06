---
layout: page
title: /database
permalink: /database
---

### Database (frequent update)

`24.05.2021-`
@yb park 

---

CONTENTS
[1. Configuration & Execution :: How to Connect MariaDB](#1-configuration--execution--how-to-connect-mariadb)
    [1-0. Reference](#1-0-reference)
    [1-1. Introduction](#1-1-introduction)
    [1-2. Explanation & Syntax](#1-2-explanation--syntax)
        [1-2-1. **Installation**](#1-2-1-installation)
        [1-2-2. **Execution**](#1-2-2-execution)
        [1-1-3. Remark](#1-1-3-remark)
[2. Relational Data Modeling](#2-relational-data-modeling)
    [2-0. Reference](#2-0-reference)
    [2-1. Introduction](#2-1-introduction)
        [2-1-1. Data Modeling](#2-1-1-data-modeling)
        [2-1-2.Conceptual-Logical-Physical Data Model](#2-1-2conceptual-logical-physical-data-model)
        [2-1-3. Conceptual Data Modeling | Conceptual Schema](#2-1-3-conceptual-data-modeling--conceptual-schema)
        [2-1-4. Logical Data Modeling](#2-1-4-logical-data-modeling)
        [2-1-5. Physical Data Modeling](#2-1-5-physical-data-modeling)
    [2-2. Explanation](#2-2-explanation)
        [2-2-1. Entity](#2-2-1-entity)
        [2-2-2. Attribute](#2-2-2-attribute)
        [2-2-3. Relationship](#2-2-3-relationship)
    [2-3. Example](#2-3-example)
        [2-3-1. Conceptual Schema](#2-3-1-conceptual-schema)
        [2-3-2. Logical Schema](#2-3-2-logical-schema)
        [2-3-3. Physical Schema](#2-3-3-physical-schema)
    [2-4. Remark](#2-4-remark)
[3. Non-Clustered Index & Clustered Index](#3-non-clustered-index--clustered-index)
    [3-0. Reference](#3-0-reference)
    [3-1. Introduction](#3-1-introduction)
        [3-1-1. **Clustered**](#3-1-1-clustered)
        [3-1-2. **Non-Clustered**](#3-1-2-non-clustered)
    [3-2. Explanation](#3-2-explanation)
        [3-2-1. Indexes and Constraints](#3-2-1-indexes-and-constraints)
        [3-2-2. How Indexes are used by the Query Optimizer┬╣](#3-2-2-how-indexes-are-used-by-the-query-optimizer)
    [3-3. Syntax](#3-3-syntax)
        [3-3-0. Basic Statement](#3-3-0-basic-statement)
        [3-3-1. **Clustered Index**](#3-3-1-clustered-index)
        [3-3-2. Non-Clustered](#3-3-2-non-clustered)
    [3-4. Conclusion](#3-4-conclusion)
    [3-5. Remark](#3-5-remark)
[4. Debug](#4-debug)
    [4-1. (MariaDB / WSL2) access denied for user 'root'@'localhost'](#4-1-mariadb--wsl2-access-denied-for-user-rootlocalhost)
        [Reference](#reference)
        [Cause](#cause)
        [Solution](#solution)
[5. Tablespaces](#5-tablespaces)
    [5-1. Reference](#5-1-reference)
    [5-2. Introduction](#5-2-introduction)
    [5-3. Explanation](#5-3-explanation)
        [Benefits of tablespaces in database](#benefits-of-tablespaces-in-database)
        [Container](#container)
        [Default tablespaces](#default-tablespaces)
    [5-4. Syntax](#5-4-syntax)
    [5-5. Remark](#5-5-remark)

---

# 1. Configuration & Execution :: How to Connect MariaDB
## 1-0. Reference
[WSL2 Mariadb ýäĄý╣śÝĽśŕŞ░.](https://blog.dalso.org/linux/wsl2/16154)
[Connect to MariaDb](https://dbschema.com/documentation/MariaDb/)

## 1-1. Introduction
MariaDB users are a combination of username and host name which are allowed to connect (can be '%' for any host). For example you can create an user in the database which may connect only from 'localhost'.

{% highlight sql %}
CREATE USER 'sample'@'localhost' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON *.* 'sample'@'localhost';
FLUSH PRIVILEGES;
{% endhighlight %}

If you try to connect from another host or from outside a docker container, you may receive the error 'Host ...is not allowed to connect'. In this case you better create the user with the right to connect from any host ('%'), or use the host which is shown in the error message.

{% highlight sql %}
CREATE USER 'sample'@'%' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON *.* TO 'sample'@'%';
FLUSH PRIVILEGES;
{% endhighlight %}

Connecting from DbSchema you should use the user 'sample'. By default MariaDb already has an user 'root'@'localhost', with a password you should have already set during installation. This user will work only if you run DBSchema on the same machine where MySql is running (if you use docker containers it is a vital separate machine). If this is not the case, you should create an user with a the a host which is matching your host. Also check the chapter which is explaining how to enable remote connections (disable bind_address in my.ini or my.cnf).

## 1-2. Explanation & Syntax

### 1-2-1. **Installation**

{% highlight bash %}
sudo apt-get install software-properties-common
sudo apt-key adv --fetch-keys 'https://mariadb.org/mariadb_release_signing_key.asc'
sudo add-apt-repository 'deb [arch=amd64,arm64,i386,ppc64el] http://mirror2.hs-esslingen.de/mariadb/repo/10.4/ubuntu xenial main'

sudo apt update
sudo apt install mariadb-server

sudo service mysql status

sudo cp /usr/share/mysql/mysql.init /etc/init.d/mysql
sudo chmod 755 /etc/init.d/mysql
sudo service mysql start
{% endhighlight %}

### 1-2-2. **Execution**

{% highlight bash %}
mysql -u root -p
{% endhighlight %}

### 1-1-3. Remark

**WSL : Switch the account to root**

{% highlight bash %}
    su - root
{% endhighlight %}

---

# 2. Relational Data Modeling
## 2-0. Reference
[ŕ┤Çŕ│äÝśĽ ŰŹ░ýŁ┤Ýä░ Ű¬ĘŰŹŞŰžü 4.2. ßäÇßů¬ßćźßäÇßůĘßäĺßůžßć╝ ßäâßůŽßäőßůÁßäÉßůąßäçßůŽßäőßůÁßäëßů│ ßäâßůíßäőßů«ßćź ßäÇßůóßäéßůžßćĚßäőßů┤ ßäÇßů«ßäîßůę](https://www.youtube.com/watch?v=hbZ96tnbN4M&list=PLuHgQVnccGMDF6rHsY9qMuJMd295Yk4sa&index=6&t=184s)
[ŰŹ░ýŁ┤Ýä░Ű▓áýŁ┤ýŐĄ ŕ░ťŰůÉýáü ŰŹ░ýŁ┤Ýä░ Ű¬ĘŰŹŞ_ŕ░ťý▓┤-ŕ┤Çŕ│ä Ű¬ĘŰŹŞ (1)](https://d9249.tistory.com/17?category=848157)
[Conceptual schema Wikipedia](https://en.wikipedia.org/wiki/Conceptual_schema)
[ŰŹ░ýŁ┤Ýä░ Ű¬ĘŰŹŞýŁś ýŁ┤ÝĽ┤](https://dataonair.or.kr/db-tech-reference/d-guide/sql/?pageid=5&mod=document&uid=330)

## 2-1. Introduction
### 2-1-1. Data Modeling
ŰŹ░ýŁ┤Ýä░ Ű¬ĘŰŹŞŰžüýŁÇ ŰőĄýŁîŕ│╝ ŕ░ÖýŁÇ 3ŕ░ÇýžÇŰíť ýáĽýŁśŰÉťŰőĄ: 
    **1. ýáĽŰ│┤ ýőťýŐĄÝůťýŁä ŕÁČýÂĽÝĽśŕŞ░ ýťäÝĽť ŰŹ░ýŁ┤Ýä░ ŕ┤ÇýáÉýŁś ýŚůŰČ┤ ŰÂäýäŁ ŕŞ░Ű▓Ľ**
    **2. ÝśäýőĄýŁś ŰŹ░ýŁ┤Ýä░ýŚÉ ŰîÇÝĽ┤ ýĽŻýćŹŰÉť ÝĹťŕŞ░Ű▓ĽýŚÉ ýŁśÝĽ┤ ÝĹťÝśäÝĽśŰŐö ŕ│╝ýáĽ**
    **3. ŰŹ░ýŁ┤Ýä░Ű▓áýŁ┤ýŐĄŰą╝ ŕÁČýÂĽÝĽśŕŞ░ ýťäÝĽť ŰÂäýäŁ Ű░Ć ýäĄŕ│äýŁś ŕ│╝ýáĽ**
   
ýŁ┤ ŕ▓âýŁä Ű│┤ŰőĄ ýőĄŰČ┤ýáüýť╝Űíť ÝĽ┤ýäŁ ÝĽ┤ Ű│┤Űę┤ ýŚůŰČ┤ýŚÉýäť ÝĽäýÜöŰíť ÝĽśŰŐö ŰŹ░ýŁ┤Ýä░Űą╝ ýőťýŐĄÝůť ŕÁČýÂĽ Ű░ęŰ▓ĽŰíáýŚÉ ýŁśÝĽ┤ ŰÂäýäŁÝĽśŕ│á ýäĄŕ│äÝĽśýŚČ ýáĽŰ│┤ ýőťýŐĄÝůťýŁä ŕÁČýÂĽÝĽśŰŐö ŕ│╝ýáĽýť╝Űíť ýáĽýŁśÝĽá ýłś ý×łŰőĄ. ŰŹ░ýŁ┤Ýä░ Ű¬ĘŰŹŞŰžüýŁä ÝĽśŰŐö ýú╝ýÜöÝĽť ýŁ┤ýťáŰŐö ýŚůŰČ┤ ýáĽŰ│┤Űą╝ ŕÁČýä▒ÝĽśŰŐö ŕŞ░ý┤łŕ░Ç ŰÉśŰŐö ýáĽŰ│┤ŰôĄýŁä ýŁ╝ýáĽÝĽť ÝĹťŕŞ░Ű▓ĽýŚÉ ýŁśÝĽ┤ ÝĹťÝśäÝĽĘýť╝ŰíťýŹĘ ýáĽŰ│┤ ýőťýŐĄÝůť ŕÁČýÂĽýŁś ŰîÇýâüýŁ┤ ŰÉśŰŐö ýŚůŰČ┤ Űé┤ýÜęýŁä ýáĽÝÖĽÝĽśŕ▓î ŰÂäýäŁÝĽśŰŐö ŕ▓âýŁ┤ ý▓ź Ű▓łýžŞ Ű¬ęýáüýŁ┤ŰőĄ. ŰĹÉ Ű▓łýžŞŰŐö ŰÂäýäŁŰÉť Ű¬ĘŰŹŞýŁä ÝćÁÝĽ┤ ýőĄýáť ŰŹ░ýŁ┤Ýä░ Ű▓áýŁ┤ýŐĄŰą╝ ýâŁýä▒ÝĽśýŚČ ŕ░ťŰ░ť Ű░Ć ŰŹ░ýŁ┤Ýä░ ŕ┤ÇŰŽČýŚÉ ýŁ┤ýÜęÝĽśŕŞ░ ýťäÝĽť ŕ▓âýŁ┤ŰőĄ.
ýŽë, **ŰŹ░ýŁ┤Ýä░ Ű¬ĘŰŹŞŰžüýŁ┤ŰŁ╝ŰŐö ŕ▓âýŁÇ ŰőĘýžÇ ŰŹ░ýŁ┤Ýä░ Ű▓áýŁ┤ýŐĄŰžîýŁä ŕÁČýÂĽÝĽśŕŞ░ ýťäÝĽť ýÜęŰĆäŰíťŰžî ýô░ýŁ┤ŰŐö ŕ▓âýŁ┤ ýĽäŰőłŰŁ╝ ŰŹ░ýŁ┤Ýä░ Ű¬ĘŰŹŞŰžü ý×Éý▓┤Űíťýäť ýŚůŰČ┤Űą╝ ýäĄŰ¬ůÝĽśŕ│á ŰÂäýäŁ**ÝĽśŰŐö ŰÂÇŰÂäýŚÉŰĆä ŰžĄýÜ░ ýĄĹýÜöÝĽť ýŁśŰ»ŞŰą╝ ŕ░ÇýžÇŕ│á ý×łŰőĄ.

<br/>
### 2-1-2.Conceptual-Logical-Physical Data Model
![img](/uploads/Untitled%20(111).png)
ÝŐ╣Ű│äÝ×ł ŰŹ░ýŁ┤Ýä░ Ű¬ĘŰŹŞýŁÇ ŰŹ░ýŁ┤Ýä░ Ű▓áýŁ┤ýŐĄŰą╝ ŰžîŰôĄýľ┤ Űé┤ŰŐö ýäĄŕ│äýäťŰíťýäť ŰÂäŰ¬ůÝĽť Ű¬ęÝĹťŰą╝ ŕ░ÇýžÇŕ│á ý×łŰőĄ. ÝśäýőĄ ýäŞŕ│äýŚÉýäť ŰŹ░ýŁ┤Ýä░ Ű▓áýŁ┤ýŐĄŕ╣îýžÇ ŰžîŰôĄýľ┤ ýžÇŰŐö ŕ│╝ýáĽýŁÇ ýťäýÖÇ ŕ░ÖýŁ┤ ýőťŕ░äýŚÉ Űö░ŰŁ╝ ýžäÝľëŰÉśŰŐö ŕ│╝ýáĽýť╝Űíťýäť ýÂöýâüÝÖö ýłśýĄÇýŚÉ Űö░ŰŁ╝ ŕ░ťŰůÉýáü ŰŹ░ýŁ┤Ýä░ Ű¬ĘŰŹŞ(Conceptual Data Model | Conceptual Schema), ŰČ╝ŰŽČýáü ŰŹ░ýŁ┤Ýä░ Ű¬ĘŰŹŞ(Logical Data Model), ŰČ╝ŰŽČýáü ŰŹ░ýŁ┤Ýä░ Ű¬ĘŰŹŞ(Physical Data Model)Űíť ýáĽŰŽČ ÝĽá ýłś ý×łŰőĄ.


ý▓śýŁî ÝśäýőĄ ýäŞŕ│äýŚÉýäť ýÂöýâüÝÖö ýłśýĄÇýŁ┤ ŰćĺýŁÇ ýâüýťä ýłśýĄÇýŁä ÝśĽýâüÝÖö ÝĽśŕŞ░ ýťäÝĽ┤ ŕ░ťŰůÉýáü ŰŹ░ýŁ┤Ýä░ Ű¬ĘŰŹŞŰžüýŁä ýáäŕ░ťÝĽťŰőĄ. **ŕ░ťŰůÉýáü ŰŹ░ýŁ┤Ýä░ Ű¬ĘŰŹŞýŁÇ ýÂöýâüÝÖö ýłśýĄÇýŁ┤ Űćĺŕ│á, ýŚůŰČ┤ ýĄĹýőČýáüýŁ┤Űę░ ÝĆČŕ┤äýáüýŁŞ ýłśýĄÇýŁś Ű¬ĘŰŹŞŰžüýŁä ýžäÝľëÝĽťŰőĄ.** ý░Şŕ│áŰíť *EA(Enterprise Architecture)┬╣* ŕŞ░Ű░śýŁś ýáäýéČýáüýŁŞ ŰŹ░ýŁ┤Ýä░ Ű¬ĘŰŹŞŰžüýŁä ýáäŕ░ťÝĽá ŰĽîŰŐö Ű│┤ŰőĄ ýâüýťä ýłśýĄÇýŁŞ ŕ░ťŕ┤äýáüýŁŞ ŰŹ░ýŁ┤Ýä░ Ű¬ĘŰŹŞŰžüýŁä ŰĘ╝ýáÇ ýłśÝľëÝĽśŕ│á, ýŁ┤ ÝŤäýŚÉ ýŚůŰČ┤ ýśüýŚşýŚÉ Űö░ŰąŞ ŕ░ťŰůÉýáü Ű¬ĘŰŹŞŰžüýŁä ýáäŕ░ťÝĽťŰőĄ. Entity ýĄĹýőČýŁś ýâüýťä ýłśýĄÇ ŰŹ░ýŁ┤Ýä░ Ű¬ĘŰŹŞýŁ┤ ýÖäýä▒ŰÉśŰę┤ **ýŚůŰČ┤ýŁś ŕÁČý▓┤ýáüýŁŞ Ű¬ĘýŐÁŕ│╝ ÝŁÉŰŽäýŚÉ Űö░ŰąŞ ŕÁČý▓┤ÝÖöŰÉť ýŚůŰČ┤ ýĄĹýőČýŁś ŰŹ░ýŁ┤Ýä░ Ű¬ĘŰŹŞýŁä ŰžîŰôĄýľ┤ Űé┤ŰŐöŰŹ░, ýŁ┤ ŕ▓âýŁ┤ Űů╝ŰŽČýáü ŰŹ░ýŁ┤Ýä░ Ű¬ĘŰŹŞýŁ┤ŰőĄ. Űů╝ŰŽČýáüýŁŞ ŰŹ░ýŁ┤Ýä░ Ű¬ĘŰŹŞŰžü ýŁ┤ÝŤä ŰŹ░ýŁ┤Ýä░ Ű▓áýŁ┤ýŐĄýŁś ýáÇý×ą ŕÁČýí░ýŚÉ Űö░ŰąŞ Table Space Űô▒ýŁä ŕ│áŰáĄÝĽť Ű░ęýőŁýŁä ŰČ╝ŰŽČýáüýŁŞ ŰŹ░ýŁ┤Ýä░ Ű¬ĘŰŹŞŰžü**ýŁ┤ŰŁ╝ŕ│á ÝĽťŰőĄ. ýŁ┤Űą╝ ýáĽŰŽČÝĽśŰę┤ ýĽäŰ×śýŁś ÝĹťýÖÇ ŕ░ÖŰőĄ. 
![img](/uploads/Untitled%20(12).png)
<br/>
### 2-1-3. Conceptual Data Modeling | Conceptual Schema
**ŕ░ťŰůÉýáü ŰŹ░ýŁ┤Ýä░ Ű▓áýŁ┤ýŐĄ ýäĄŕ│äŰŐö ýí░ýžü, ýéČýÜęý×ÉýŁś ŰŹ░ýŁ┤Ýä░ ýÜöŕÁČ ýéČÝĽşýŁä ý░żŕ│á ŰÂäýäŁÝĽśŰŐö ŰŹ░ýŚÉýäť ýőťý×ĹÝĽťŰőĄ.ýŁ┤ ŕ│╝ýáĽýŁÇ ýľ┤ŰľáÝĽť ý×ÉŰúîŕ░Ç Ű│┤ŰőĄ ýĄĹýÜöÝĽśŰę░ ŰśÉ ýľ┤ŰľáÝĽť ý×ÉŰúîŕ░Ç ýťáýžÇŰÉśýľ┤ýĽ╝ ÝĽśŰŐö ýžÇŰą╝ ŕ▓░ýáĽÝĽśŰŐö ŕ▓â**ŰĆä ÝĆČÝĽĘÝĽťŰőĄ. ýŁ┤ ŰőĘŕ│äýŚÉ ý×łýľ┤ýäť ýú╝ýÜöÝĽť ÝÖťŰĆÖýŁÇ ÝĽÁýőČ EntityýÖÇ ŕĚŞŰôĄ ŕ░äýŁś ŕ┤Çŕ│äŰą╝ Ű░ťŕ▓ČÝĽśŕ│á, ŕĚŞ ŕ▓âýŁä ÝĹťÝśäÝĽśŕŞ░ ýťäÝĽ┤ýäť Entity-Relationship Diagram(ýŁ┤ÝĽś ERD)ýŁä ýâŁýä▒ÝĽśŰŐö ŕ▓âýŁ┤ŰőĄ. ERDŰŐö ýí░ýžüŕ│╝ ŰőĄýľĹÝĽť ŰŹ░ýŁ┤Ýä░ Ű▓áýŁ┤ýŐĄ ýéČýÜęý×ÉýŚÉŕ▓î ýľ┤ŰľáÝĽť ŰŹ░ýŁ┤Ýä░ŕ░Ç ýĄĹýÜöÝĽťýžÇ ŰéśÝâÇŰé┤ŕŞ░ ýťäÝĽ┤ýäť ýéČýÜęŰÉťŰőĄ. ŰŹ░ýŁ┤Ýä░ Ű¬ĘŰŹŞŰžü ŕ│╝ýáĽýŁ┤ ýáä ýí░ýžüýŚÉ ŕ▒Şý│É ýŁ┤ŰúĘýľ┤ýžäŰőĄŰę┤, ŕĚŞ ŕ▓âýŁÇ ýáäýéČýáü ŰŹ░ýŁ┤Ýä░ Ű¬ĘŰŹŞ(Enterprise Data Model)ýŁ┤ŰŁ╝ŕ│á ŰÂłŰŽ░ŰőĄ. 

ŕ░ťŰůÉýáü ŰŹ░ýŁ┤Ýä░ Ű¬ĘŰŹŞýŁä ÝćÁÝĽ┤ ýí░ýžüýŁś ŰŹ░ýŁ┤Ýä░ ýÜöŕÁČŰą╝ ŕ│ÁýőŁÝÖö ÝĽśŰŐö ŕ▓âýŁÇ ŰőĄýŁîýŁś ŰĹÉ ŕ░ÇýžÇ ýĄĹýÜöÝĽť ŕŞ░ŰŐąýŁä ýžÇýŤÉÝĽśŰŐöŰŹ░, **ý▓ź ýžŞ, ŕ░ťŰůÉýáü ŰŹ░ýŁ┤Ýä░ Ű¬ĘŰŹŞýŁÇ ýéČýÜęý×ÉýÖÇ ýőťýŐĄÝůť ŕ░ťŰ░ťý×Éŕ░Ç ŰŹ░ýŁ┤Ýä░ ýÜöŕÁČ ýéČÝĽşýŁä Ű░ťŕ▓ČÝĽśŰŐö ŕ▓âýŁä ýžÇýŤÉÝĽťŰőĄ.** ŕ░ťŰůÉýáü ŰŹ░ýŁ┤Ýä░ Ű¬ĘŰŹŞýŁÇ ýÂöýâüýáüýŁ┤ŰőĄ. ŕĚŞŰáçŕŞ░ ŰĽîŰČŞýŚÉ ŕĚŞ Ű¬ĘŰŹŞýŁÇ ýâüýťäýŁś ŰČŞýáťýŚÉ ŰîÇÝĽť ŕÁČýí░ÝÖöŰą╝ ýëŻŕ▓î ŰžîŰôĄŰę░, ýéČýÜęý×ÉýÖÇ ŕ░ťŰ░ťý×Éŕ░Ç ýőťýŐĄÝůť ŕŞ░ŰŐąýŚÉ ŰîÇÝĽ┤ýäť Űů╝ýŁśÝĽá ýłś ý×łŰŐö ŕŞ░Ű░śýŁä ÝśĽýä▒ÝĽťŰőĄ. **ŰĹś ýžŞ ŕ░ťŰůÉýáü ŰŹ░ýŁ┤Ýä░ Ű¬ĘŰŹŞýŁÇ Ýśä ýőťýŐĄÝůťýŁ┤ ýľ┤Űľ╗ŕ▓î Ű│ÇÝśĽŰÉśýľ┤ýĽ╝ ÝĽśŰŐö ŕ░ÇŰą╝ ýŁ┤ÝĽ┤ÝĽśŰŐö ŰŹ░ýŚÉ ýťáýÜęÝĽśŰőĄ.** ýŁ╝Ű░śýáüýť╝Űíť ŰžĄýÜ░ ŕ░äŰőĘÝĽśŕ▓î ŕ│áŰŽŻŰÉť(Stand Alone) ýőťýŐĄÝůťŰĆä ýÂöýâüýáü Ű¬ĘŰŹŞŰžüýŁä ÝćÁÝĽ┤ýäť Ű│┤ŰőĄ ýëŻŕ▓î ÝĹťÝśäŰÉśŕ│á ýäĄŰ¬ůŰÉťŰőĄ.
 
<br/>
### 2-1-4. Logical Data Modeling
**Űů╝ŰŽČýáü ŰŹ░ýŁ┤Ýä░ Ű¬ĘŰŹŞŰžüýŁÇ ŰŹ░ýŁ┤Ýä░ Ű▓áýŁ┤ýŐĄ ýäĄŕ│ä ÝöäŰíťýäŞýŐĄýŁś inputýť╝ŰíťýŹĘ, Ű╣äýŽłŰőłýŐĄ ýáĽŰ│┤ýŁś Űů╝ŰŽČýáüýŁŞ ŕÁČýí░ýÖÇ ŕĚťý╣ÖýŁä Ű¬ůÝÖĽÝĽśŕ▓î ÝĹťÝśäÝĽśŰŐö ŕŞ░Ű▓Ľ ŰśÉŰŐö ŕ│╝ýáĽ**ýŁ┤ŰŁ╝ŕ│á ÝĽá ýłś ý×łŰőĄ. ŰŹ░ýŁ┤Ýä░ Ű¬ĘŰŹŞŰžüýŁ┤Ű×Ç Ű¬ĘŰŹŞŰžü ŕ│╝ýáĽýŁ┤ ýĽäŰőî Ű│äŰĆäýŁś ŕ│╝ýáĽýŁä ÝćÁÝĽ┤ýäť ýí░ýéČÝĽśŕ│á ŕ▓░ýáĽÝĽť ýéČýőĄýŁä ŰőĘýžÇ ERDŰŁ╝ŰŐö ŕĚŞŰŽ╝ýť╝Űíť ŕĚŞŰáĄŰé┤ŰŐö ŕ│╝ýáĽýŁä ŰžÉÝĽśŰŐö ŕ▓âýŁ┤ ýĽäŰőłŰőĄ. ýőťýŐĄÝůť ŕÁČýÂĽýŁä ýťäÝĽ┤ýäť ŕ░Çý×ą ŰĘ╝ýáÇ ýőťý×ĹÝĽá ŕŞ░ý┤łýáüýŁŞ ýŚůŰČ┤ ýí░ýéČŰą╝ ÝĽśŰŐö ý┤łŕŞ░ ŰőĘŕ│äŰÂÇÝä░ ýŁŞŕ░äýŁ┤ ŕ▓░ýáĽÝĽ┤ýĽ╝ ÝĽá ŰîÇŰÂÇŰÂäýŁś ýéČÝĽşýŁä Ű¬ĘŰĹÉ ýáĽýŁśÝĽśŰŐö ýőťýŐĄÝůť ýäĄŕ│äýŁś ýáä ŕ│╝ýáĽýŁä ýžÇýŤÉÝĽśŰŐö 'ŕ│╝ýáĽýŁś ŰĆäŕÁČ'ŰŁ╝ŕ│á ÝĽ┤ýĽ╝ ÝĽá ŕ▓âýŁ┤ŰőĄ. 

**ýŁ┤ ŰőĘŕ│äýŚÉýäť ýłśÝľëÝĽśŰŐö ŰśÉ ÝĽť ŕ░ÇýžÇ ýĄĹýÜöÝĽť ŕ│╝ýáĽýŁÇ ýáĽŕĚťÝÖöýŁ┤ŰőĄ. ýáĽŕĚťÝÖöŰŐö Űů╝ŰŽČ ŰŹ░ýŁ┤Ýä░ Ű¬ĘŰŹŞ ýâüýäŞÝÖö ŕ│╝ýáĽýŁś ŰîÇÝĹťýáüýŁŞ ÝÖťŰĆÖýť╝Űíť, Űů╝ŰŽČ ŰŹ░ýŁ┤Ýä░ Ű¬ĘŰŹŞýŁś ýŁ╝ŕ┤Çýä▒ýŁä ÝÖĽŰ│┤ÝĽśŕ│á ýĄĹŰ│ÁýŁä ýáťŕ▒░ÝĽśýŚČ ýćŹýä▒ŰôĄýŁ┤ ŕ░Çý×ą ýáüýáłÝĽť ŕ░ťý▓┤ýŚÉ Ű░░ý╣śŰÉśŰĆäŰíŁ ÝĽĘýť╝ŰíťýŹĘ Ű│┤ŰőĄ ýőáŰó░ýä▒ ý×łŰŐö ŰŹ░ýŁ┤Ýä░ ŕÁČýí░Űą╝ ýľ╗ŰŐö ŰŹ░ýŚÉ Ű¬ęýáüýŁ┤ ý×łŰőĄ.** Űů╝ŰŽČ ŰŹ░ýŁ┤Ýä░ Ű¬ĘŰŹŞýŁś ýâüýäŞÝÖöŰŐö ýőŁŰ│äý×É ÝÖĽýáĽ, ýáĽŕĚťÝÖö, m:m ŕ┤Çŕ│ä ÝĽ┤ýćî, ý░Şýí░ ŰČ┤ŕ▓░ýä▒ ŕĚťý╣Ö ýáĽýŁś Űô▒ýŁä ŰôĄ ýłś ý×łýť╝Űę░, ýÂöŕ░Çýáüýť╝Űíť ýŁ┤Űáą ŕ┤ÇŰŽČýŚÉ ŰîÇÝĽť ýáäŰ×ÁýŁä ýáĽýŁśÝĽśýŚČ ýŁ┤Űą╝ Űů╝ŰŽČ ŰŹ░ýŁ┤Ýä░ Ű¬ĘŰŹŞýŚÉ Ű░śýśüÝĽĘýť╝ŰíťýŹĘ ŰŹ░ýŁ┤Ýä░ Ű¬ĘŰŹŞŰžüýŁä ýÖäŰúîÝĽśŕ▓î ŰÉťŰőĄ.

<br/>
### 2-1-5. Physical Data Modeling
ŰŹ░ýŁ┤Ýä░ Ű▓áýŁ┤ýŐĄ ýäĄŕ│ä ŕ│╝ýáĽýŁś ýäŞ Ű▓łýžŞ ŰőĘŕ│äýŁŞ **ŰČ╝ŰŽČ ŰŹ░ýŁ┤Ýä░ Ű¬ĘŰŹŞŰžüýŁÇ Űů╝ŰŽČ ŰŹ░ýŁ┤Ýä░ Ű¬ĘŰŹŞýŁ┤ ŰŹ░ýŁ┤Ýä░ ýáÇý×ąýćîŰíťýäť ýľ┤Űľ╗ŕ▓î ý╗┤ÝôĘÝä░ ÝĽśŰôťýŤĘýľ┤ýŚÉ ÝĹťÝśäŰÉá ŕ▓â ýŁŞŕ░ÇŰą╝ ŰőĄŰúČŰőĄ.** ŰŹ░ýŁ┤Ýä░ŕ░Ç ŰČ╝ŰŽČýáüýť╝Űíť ý╗┤ÝôĘÝä░ýŚÉ ýľ┤Űľ╗ŕ▓î ýáÇý×ąŰÉá ŕ▓â ýŁŞŕ░ÇýŚÉ ŰîÇÝĽť ýáĽýŁśŰą╝ Physical SchemaŰŁ╝ŕ│á ÝĽťŰőĄ. ýŁ┤ ŰőĘŕ│äýŚÉýäť ŕ▓░ýáĽŰÉśŰŐö ŕ▓âýŁÇ Table, Column Űô▒ýť╝Űíť ÝĹťÝśäŰÉśŰŐö ŰČ╝ŰŽČýáüýŁŞ ýáÇý×ą ŕÁČýí░ýÖÇ ýéČýÜęŰÉá ýáÇý×ą ý×ąý╣ś, ý×ÉŰúîŰą╝ ýÂöýÂťÝĽśŕŞ░ ýťäÝĽ┤ ýéČýÜęŰÉá ýáĹŕĚ╝ Ű░ęŰ▓Ľ Űô▒ýŁ┤ ý×łŰőĄ. ŕ│äýŞÁýáü ŰŹ░ýŁ┤Ýä░ Ű▓áýŁ┤ýŐĄ ŕ┤ÇŰŽČ ýőťýŐĄÝůť ÝÖśŕ▓ŻýŚÉýäťŰŐö ŰŹ░ýŁ┤Ýä░ Ű▓áýŁ┤ýŐĄ ŕ┤ÇŰŽČý×Éŕ░Ç Physical SchemaŰą╝ ýäĄŕ│äÝĽśŕ│á ŕÁČÝśäÝĽśŕŞ░ ýťäÝĽ┤ýäť Ű│┤ŰőĄ ŰžÄýŁÇ ýőťŕ░äýŁä ÝłČý×ÉÝĽśýŚČýĽ╝ ÝĽťŰőĄ.

ýőĄýžłýáüýŁŞ ÝöäŰíťýáŁÝŐŞýŚÉýäťŰŐö ýťäýŁś Ű¬ĘŰôá ŰŹ░ýŁ┤Ýä░ Ű¬ĘŰŹŞŰžüýŁä ýłśÝľëÝĽśŰŐö ŕ▓ŻýÜ░ŰŐö ŰôťŰČ╝Űę░, ŕ░ťŰůÉýáü ŰŹ░ýŁ┤Ýä░ Ű¬ĘŰŹŞŰžüŕ│╝ Űů╝ŰŽČýáü ŰŹ░ýŁ┤Ýä░ Ű¬ĘŰŹŞŰžüýŁä ÝĽť Ű▓łýŚÉ ýłśÝľëÝĽť ÝŤä ŰŹ░ýŁ┤Ýä░ Ű¬ĘŰŹŞŰžüýť╝Űíť ýłśÝľëÝĽśŰŐö ŕ▓ŻýÜ░ŕ░Ç ŰîÇŰÂÇŰÂäýŁ┤ŰőĄ. 

<br/>
## 2-2. Explanation
![img](/uploads/Untitled%20(13).png)

<br/>
### 2-2-1. Entity
ýéČŰ×î, ýéČŰČ╝, ŕ░ťŰůÉ, ýéČŕ▒┤ŕ│╝ ŕ░ÖýŁ┤ ŰĆůŰŽŻýáüýť╝Űíť ýí┤ý×ČÝĽśŰę┤ýäť ŕ│áýťáÝĽśŕ▓î ýőŁŰ│äýŁ┤ ŕ░ÇŰŐąÝĽť ýőĄ ýäŞŕ│äýŁś ŕ░Łý▓┤Űą╝ ŰžÉ ÝĽťŰőĄ. ýŁ┤ ýĄĹ, ýáÇý×ąÝĽá ŕ░Çý╣śŕ░Ç ý×łŰŐö ýĄĹýÜö ŰŹ░ýŁ┤Ýä░Űą╝ ŕ░ÇýžÇŕ│á ý×łŰŐö ýéČŰ×î, ýéČŰČ╝, ŕ░ťŰůÉ, ýéČŕ▒┤ Űô▒ýŁä ýÂöŰŽ░ ŕ▓âýŁ┤ŰőĄ. ýŁ┤ ŰôĄýŁÇ ŰőĄŰąŞ ŕ░ťý▓┤ýÖÇ ŕÁČŰ│äŰÉśŰŐö ýŁ┤ŰŽäýŁä ŕ░ÇýžÇŕ│á ý×łýť╝Űę░, ŕ░ü ŕ░ťý▓┤ ŰžîýŁś ŕ│áýťáÝĽť ÝŐ╣ýä▒ýŁ┤Űéś ýâüÝâť, ýŽë ýćŹýä▒(Attribute)Űą╝ Ű░śŰôťýőť ÝĽśŰéś ýŁ┤ýâü ýćîýťáÝĽťŰőĄ.

### 2-2-2. Attribute
Mandatory(ÝĽäýłś, Not Null) / Optional(ŰÂÇŕ░Ç, Nullable) ŰĹÉ ŕ░ÇýžÇ ŕ▓ŻýÜ░ŕ░Ç ýí┤ý×ČÝĽá ýłś ý×łýť╝Űę░, ýŁ┤ŰŐö ŕ░ťý▓┤Űéś ŕ┤Çŕ│äŕ░Ç ŕ░ÇýžÇŕ│á ý×łŰŐö ŕ│áýťáýŁś ÝŐ╣ýä▒ýŁä ŰžÉ ÝĽťŰőĄ. ÝĽśŰéśýŁś EntityŰŐö ýŚ░ŕ┤ÇŰÉť AttributeýŁś ýžĹÝĽęýť╝Űíť ýäĄŰ¬ůŰÉśŰę░, ÝĽť AttributeýŁś domainýŁÇ ŕĚŞ ýćŹýä▒ýŁ┤ ŕ░Çýžł ýłś ý×łŰŐö Ű¬ĘŰôá ŕ░ĺŰôĄýŁś ýžĹÝĽęýŁä ýŁśŰ»ŞÝĽťŰőĄ. ýŚČŰčČ AttributeŰŐö ŰĆÖýŁ╝ÝĽť domainýŁä ŕ░Çýžł ýłś ý×łýť╝Űę░, key AttributeŰŐö ÝĽť Attribute ŰśÉŰŐö AttributeŰôĄýŁś Ű¬Ęý×äýť╝Űíťýäť ÝĽť Entity ÝâÇý×ů Űé┤ýŚÉýäť ŕ░ü ŕ░ťý▓┤Űą╝ ŕ│áýťáÝĽśŕ▓î ýőŁŰ│äÝĽťŰőĄ.

ýŁ┤ ýĄĹ, ýćŹýä▒ýŁÇ ŰőĄýľĹÝĽť ŰÂäŰąśŰą╝ ŕ▒░ý│É ý×Ĺýä▒ Ű▓ĽýŁ┤ ŰőČŰŁ╝ýžÇŰŐöŰŹ░, ŕĚŞ ý×Ĺýä▒ Ű▓ĽýŁÇ ŰőĄýŁîŕ│╝ ŕ░ÖŰőĄ:

1. **Single-Valued Attribute (ŰőĘýŁ╝ ŕ░ĺ ýćŹýä▒)**
    1ŕ░ťýŁś ŕ░ĺýŁä ŕ░ÇýžÇŰŐö, ýŽë ýŤÉý×Éýä▒ýŁś ýćŹýä▒ýŁ┤ŰőĄ.

2. **Multi-Valued Attribute (ŰőĄýĄĹ ŕ░ĺ ýćŹýä▒)**
    ýŚČŰčČ ŕ░ťýŁś ŕ░ĺýŁä ŕ░Çýžł ýłś ý×łŰŐö ýćŹýä▒ýť╝Űíť, ýŁ┤ýĄĹ ýäá ÝâÇýŤÉýť╝Űíť ÝĹťÝśäÝĽťŰőĄ.

3. **Simple Attribute (ŰőĘýłť ýćŹýä▒)**
    ýŁśŰ»ŞŰą╝ ŰÂäÝĽ┤ÝĽá ýłś ýŚćŰŐö ýćŹýä▒ýŁ┤ŰőĄ.

4. **Composite Attribute (Ű│ÁÝĽę ýćŹýä▒)**
    ýŁśŰ»ŞýŁś ŰÂäÝĽ┤ŕ░Ç ŕ░ÇŰŐąÝĽť ýćŹýä▒ýť╝Űíť, ýäŞŰÂäÝÖöŰÉť ýćŹýä▒ýŁ┤ŰőĄ.

5. **Stored Attribute (ýáÇý×ą ýćŹýä▒)**
    ŰőĄŰąŞ ýćŹýä▒ýŁś ŕ░ĺýť╝ŰíťŰÂÇÝä░ ýľ╗ýŁä ýłś ýŚćŰŐö ýćŹýä▒ýŁ┤ŰőĄ.

6. **Derived Attribute (ÝîîýâŁ ýćŹýä▒)**
    ŰőĄŰąŞ ýćŹýä▒ýŁś ŕ░ĺýť╝ŰíťŰÂÇÝä░ ýťáŰĆäŰÉśýľ┤ ŕ▓░ýáĽŰÉśŰŐö ýćŹýä▒ýŁ┤ŰőĄ.

### 2-2-3. Relationship
ŕ░ťý▓┤ŰôĄ ýéČýŁ┤ýŚÉ Űž║ŕ│á ý×łŰŐö ýŚ░ŕ┤Ç ŰśÉŰŐö ýŚ░ŕ▓░, ŰîÇýŁĹ(mapping)ýŁä ýŁśŰ»ŞÝĽťŰőĄ. ýŁ┤ŰôĄýŁÇ ŰžłŰŽäŰ¬Ę ŕ╝┤ýŁś ÝśĽÝâťŰíť ý×Ĺýä▒ÝĽśŕ│á, ýäťŰíť ýŚ░ŕ┤ÇŰÉť ŕ░ťý▓┤ŰôĄýŁä ýőĄýäáýť╝Űíť ýŚ░ŕ▓░ÝĽťŰőĄ. 

![img](/uploads/Untitled%20(14).png)
![img](/uploads/Untitled%20(15).png)
![img](/uploads/Untitled%20(16).png)

## 2-3. Example
### 2-3-1. Conceptual Schema
![img](/uploads/Conceptual-Schema.png)

### 2-3-2. Logical Schema
![img](/uploads/Logical-Model.png)

### 2-3-3. Physical Schema
![img](/uploads/Physical-Model.png)

## 2-4. Remark

---

# 3. Non-Clustered Index & Clustered Index
![img](/uploads/Untitled%20(17).png)

## 3-0. Reference
## 3-1. Introduction
An index is an on-disk structure associated with a table or view that speeds retrieval of rows from the table or view. An index contains keys built from one or more columns in the table or view. There keys are stored in a structure (B-tree) that enables SQL Server to find the row or rows associated with the key values quickly and efficiently.

A table or view can contain the following types of indexes.

### 3-1-1. **Clustered**
Clustered indexes sort and store the data rows in the table or view based on their key values. These are the columns included in the index definition. There can be only one clustered index per table, because the data rows themselves can be stored in only one order.
The only time the data rows in a table are stored in sorted order is when the table contains a clustered index. When a table has a clustered index, the table is called a clustered table. If a table has no clustered index, its data rows are stored in an unordered structure called a heap.

### 3-1-2. **Non-Clustered**
Non-Clustered indexes have a structure separate from the data rows. A non-clustered index contains the non-clustered index key values and each key value entry has a pointer to the data row that contains the key value.
The pointer from an index row in a non-clustered index to a data row is called a row locator. The structure of the row locator depend on whether the data pages are stored in a heap or a clustered table. For a heap, a row locator is a pointer to the row. For a clustered table, the row locator is the clustered index key.
You can add non-key columns to the leaf level of the non-clustered index to by-pass existing index key limits, and execute fully covered, indexed, queries.

Both clustered and non-clustered indexes can be unique. This means no two rows can have the same value for the index key. Otherwise, the index is not unique and multiple rows can share the same key value. 

Indexes are automatically maintained for a table or view whenever the table data is modified.

## 3-2. Explanation
### 3-2-1. Indexes and Constraints
Indexes are automatically created when Primary Key and Unique constraints are defined on table columns. i.e. when you create a table with a Unique constraint, Database Engine automatically creates a non-clustered Index. if you configure a Primary Key, Database Engine automatically creates a clustered Index, unless a clustered index already exists. When you try to enforce a Primary Key constraint on an existing table and clustered index already exists on that table, SQL Server enforces the primary key using a non clustered index.

### 3-2-2. How Indexes are used by the [Query Optimizer┬╣](https://en.wikipedia.org/wiki/Query_optimization#:~:text=Query%20optimization%20is%20a%20feature,considering%20the%20possible%20query%20plans.)
Well-designed Indexes can reduce disk I/O Operations and consume fewer system resources therefore improving query performance. Indexes can be helpful for a variety of queries that contain SELECT, UPDATE, DELETE, or MERGE statements. Consider the query `SELECT Title, HireDate FROM humanResources. Employee WHERE EmployeeID = 250` in the AdventureWorks2012 database. When this query is executed, the query Optimizer evaluates each available method for retrieving the data and selects the most efficient method. The method may be a table scan, or may be scanning one or more indexes if they exist.

When performing a table scan, the query optimizer reads all the rows in the table, and extracts the rows that meet the criteria of the query. A table scan generates many disk I/O operations and can be resource intensive. However, a table scan could be the most efficient method if, i.e. the result set of the query is a high percentage of rows from the table.

When the query optimizer uses an index, it searches the index key columns, finds the storage location of the rows needed by the query and extracts the matching rows from the location. Generally, searching the index is much faster than search the table because unlike a table, an index frequently contains very few columns per row and the rows are in sorted order.

The query optimizer typically selects the most efficient method when executing queries. However, if no indexes are available, the query optimizer must use a table scan. Your task is to design and create indexes that are best suited to your environment so that the query optimizer has a selection of efficient indexes from which to select. SQL Server provides the Database Engine Tuning Advisor to help with the analysis of your database environment and in the selection of appropriate indexes.

## 3-3. Syntax
### 3-3-0. Basic Statement
{% highlight sql %}
CREATE [ UNIQUE | FULLTEXT | SPATIAL ] INDEX index_name
[ index_type ] ON table_name (key_part)
[ index_option ][ algorithm_option | lock_option ]

key_part: { col_name [ ( length ) | ( expr ) ] } [ ASC | DESC ]
index_option: {
	KEY_BLOCK_SIZE [=] VALUE
	| index_type
	| WITH PARSER parser_name
	| COMMENT parser_name
	| { VISIBLE | INVISIBLE }
	| ENGINE_ATTRIBUTE [=] 'string'
	| SECONDARY_ENGINE_ATTRIBUTE [=] 'string'
}

index_type :
	USING { BTREE | HASH }
algorithm_option :
 ALGORITHM [=] { DEFAULT | INPLACE | COPY }
lock_option:
LOCK [=] { DEFAULT | NONE | SHARED | EXCLUSIVE }
{% endhighlight %}

### 3-3-1. **Clustered Index**
{% highlight sql %}
CREATE INDEX idx_index_name ON table_name(column1, column2);
{% endhighlight %}

*e.g.*
{% highlight sql %}
CREATE TABLE `student_info` (
	`id` INT NOT NULL AUTO_INCREMENT,
	`name` VARCHAR(45) DEFAULT NULL,
	`age` INT DEFAULT NULL,
	`mobile` VARCHAR(20) DEFAULT NULL,
	`email` VARCHAR(25) DEFAULT NULL,
	PRIMARY KEY (`student_id`),
	UNIQUE KEY `email_UQ` (`email`)
)
{% endhighlight %}

### 3-3-2. Non-Clustered
*e.g.*
{% highlight sql %}
CREATE NonClustered INDEX index_name ON table_name (column_name ASC);
{% endhighlight %}

## 3-4. Conclusion
![img](/uploads/Untitled%20(18).png)

## 3-5. Remark

---

# 4. Debug
## 4-1. (MariaDB / WSL2) access denied for user 'root'@'localhost'
### Reference
[MySQL Error: : 'Access denied for user 'root'@'localhost'](https://stackoverflow.com/questions/41645309/mysql-error-access-denied-for-user-rootlocalhost)
[ALTER USER](https://mariadb.com/kb/en/alter-user/#authentication-options)

### Cause
UserýŁś ýŁŞýŽŁýŚÉ ŰČŞýáťŕ░Ç ý×łŕŞ░ ŰĽîŰČŞýŚÉ Ű░ťýâŁÝĽťŰőĄ.

### Solution
ŰőĄýŁîŕ│╝ ŕ░ÖýŁÇ ýłťýäťŰíť ÝĽ┤ŕ▓░ÝĽťŰőĄ.

1. `su - root` ubuntu shell ý░ŻýŁä ý╝á ÝŤä, root ŕÂîÝĽťýť╝Űíť ýőĄÝľëÝĽťŰőĄ.
2. `mysql -u root -p` Űíť MariaDBŰą╝ ýőĄÝľëÝĽťŰőĄ.
    ýŁ┤ŰŐö root user ŕÂîÝĽťýť╝Űíť passwordŰą╝ ý×ůŰáąÝĽśýŚČ ýőĄÝľëÝĽťŰőĄŰŐö ýŁśŰ»ŞýŁ┤ŰőĄ.
3. `root@localhost`ýŁś ýŁŞýŽŁ Ű░ęýőŁýŁä ÝîĘýŐĄýŤîŰôťŰíť ýäĄýáĽÝĽśŕ│á, ŕĚŞ ÝîĘýŐĄýŤîŰôťŰą╝ rootŰíť ýäĄýáĽÝĽ┤ ýĄÇŰőĄ.
    {% highlight sql %} ALTER USER 'root'@'localhost' IDENTIFIED VIA mysql_native_password USING PASSWORD('root'); {% endhighlight %}
ŰőĄýőť ÝĽť Ű▓ł connection ýâŁýä▒ýŁä ýőťŰĆäÝĽśŰę┤ ýä▒ŕ│Áýáüýť╝Űíť ýÖäŰúîŰÉťŰőĄ.


---

# 5. Tablespaces
## 5-1. Reference
[DB2 Tablespaces](https://www.tutorialspoint.com/db2/db2_tablespaces.htm)

## 5-2. Introduction
A table space is a storage structure, it contains tables, indexes, large objects, and long data. It can be used to organize data in a database into logical storage which is related with where data stored on a system. This tablespaces are stored in database partition groups. 

TablespaceŰŐö tables, indexes, Ýü░ ŕ░Łý▓┤ŰôĄ, ŕĚŞŰŽČŕ│á ŕŞ┤ ŰŹ░ýŁ┤Ýä░Űą╝ ÝĆČÝĽĘÝĽť ýáÇý×ą ŕÁČýí░ýŁ┤ŰőĄ. ýŁ┤ŰŐö ŰŹ░ýŁ┤Ýä░Ű▓áýŁ┤ýŐĄ ýĽłýŚÉýäť ýőťýŐĄÝůťýŚÉýäť ŰŹ░ýŁ┤Ýä░ŕ░Ç ýľ┤ŰööýŚÉ ýáÇý×ąŰÉśýľ┤ ý×łŰŐö ýžÇýÖÇ ýŚ░ŕ┤ÇŰÉť Űů╝ŰŽČýáüýŁŞ ýáÇý×ą ŕ│Áŕ░ä Űé┤ýŚÉýäť ŰŹ░ýŁ┤Ýä░Űą╝ ýí░ýžüÝÖö ÝĽśŰŐö ŰŹ░ýŚÉ ýô░ýŁ╝ ýłś ý×łŰőĄ.

## 5-3. Explanation
### Benefits of tablespaces in database
The table spaces are beneficial in database in various ways given as follows:

**Recoverability:** Tablespaces make backup and restore operations more convenient. Using a single command, you can make backup or restore all the database objects in tablespaces.

**Automatic storage Management:** Database manager creates and extends containers depending on the needs.

**Memory utilization**: A single [bufferpool┬╣](https://www.ibm.com/docs/en/db2/11.5?topic=databases-buffer-pools) can manage multiple tablespaces. You can assign temporary tablespaces to their own bufferpool to increase the performance of activities such as sorts or joins.

**ŰŹ░ýŁ┤Ýä░Ű▓áýŁ┤ýŐĄ Űé┤ TablespaceýŁś ýŁ┤ýáÉ**
**Ű│ÁŕÁČ ŕ░ÇŰŐąÝĽť:** TablespaceŰŐö Ű░▒ýŚůýŁä ŰžîŰôĄŕ│á Ű│ÁŕÁČ ýÜ┤ýśüÝĽśŰŐö ŕ▓âýŁä ŰŹöýÜ▒ ÝÄŞŰŽČÝĽśŕ▓î ÝĽťŰőĄ. ŰőĘýŁ╝ Ű¬ůŰá╣ýľ┤Űą╝ ýéČýÜęÝĽśŰŐö ŕ▓âýť╝Űíť tablespace ýĽłýŁś Ű¬ĘŰôá ŰŹ░ýŁ┤Ýä░Ű▓áýŁ┤ýŐĄ ŕ░Łý▓┤ŰôĄýŁä Ű░▒ýŚů ŰśÉŰŐö Ű│ÁŕÁČÝĽá ýłś ý×łŰőĄ. 

**ýáÇý×ąýćî ý×ÉŰĆÖ ŕ┤ÇŰŽČ:** ŰŹ░ýŁ┤Ýä░Ű▓áýŁ┤ýŐĄ ŕ┤ÇŰŽČý×ÉŰŐö ÝĽäýÜöýŚÉ Űö░ŰŁ╝ ý╗ĘÝůîýŁ┤ŰäłŰą╝ ŰžîŰôĄŕ│á ÝÖĽý×ąÝĽťŰőĄ.

**ŰęöŰ¬ĘŰŽČ ÝÖťýÜę :** ŰőĘýŁ╝ [Bufferpool┬╣](https://www.ibm.com/docs/en/db2/11.5?topic=databases-buffer-pools)ýŁ┤ Ű│ÁýłśýŁś TablespaceŰą╝ ŕ┤ÇŰŽČÝĽá ýłś ý×łŰőĄ. joinýŁ┤Űéś ýáĽŰáČŕ│╝ ŕ░ÖýŁÇ ÝÖťŰĆÖýŁś ÝŹ╝ÝĆČŰĘ╝ýŐĄŰą╝ ŰćĺýŁ┤ŕŞ░ ýťäÝĽ┤ ý×äýőť TablespaceŰôĄýŁä ŕĚŞŰôĄýŁ┤ ýćîýťáÝĽť bufferpoolýŚÉ ÝĽáŰő╣ÝĽá ýłś ý×łŰőĄ.

### Container
Tablespaces contains one or more containers. A container can be a directory name, a device name, or a filename. In a database, a single tablespace can have several containers on the same physical storage device. If the tablespace is created with automatic storage tablespace option, the creation and management of containers handled automatically by the database manager. If it is not created with automatic storage tablespace option, you need to define and manage the containers yourself.

**ý╗ĘÝůîýŁ┤Űäł**
TablespaceŰôĄýŁÇ ÝĽśŰéś ŰśÉŰŐö ŕĚŞ ýŁ┤ýâüýŁś ý╗ĘÝůîýŁ┤ŰäłŰôĄýŁä ÝĆČÝĽĘÝĽťŰőĄ. ÝĽśŰéśýŁś ý╗ĘÝůîýŁ┤ŰäłŰŐö ŰööŰáëÝćáŰŽČŰ¬ů, ŰööŰ░öýŁ┤ýŐĄŰ¬ů, ŰśÉŰŐö ÝîîýŁ╝Ű¬ůýŁ┤ ŰÉá ýłś ý×łŰőĄ. ŰŹ░ýŁ┤Ýä░Ű▓áýŁ┤ýŐĄýŚÉýäť, ŰőĘýŁ╝ TablespaceŰŐö ŰĆÖýŁ╝ÝĽť ŰČ╝ŰŽČýáü ýáÇý×ą ý×ąý╣ś ýťäýŚÉ Ű¬ç ý╗ĘÝůîýŁ┤ŰäłŰôĄýŁä ŕ░Çýžł ýłś ý×łŰőĄ. ŰžîýĽŻ Tablespaceŕ░Ç ý×ÉŰĆÖ storage Tablespace ýśÁýůśýŚÉ ýŁśÝĽ┤ ýâŁýä▒ŰÉśýŚłŰőĄŰę┤, ý╗ĘÝůîýŁ┤ŰäłýŁś ýâŁýä▒ŕ│╝ ŕ┤ÇŰŽČŰŐö ŰŹ░ýŁ┤Ýä░Ű▓áýŁ┤ýŐĄ ŕ┤ÇŰŽČý×ÉýŚÉ ýŁśÝĽ┤ ý×ÉŰĆÖýť╝Űíť ÝĽŞŰôĄŰžüŰÉťŰőĄ. ý×ÉŰĆÖ storage tablespace ýśÁýůśýť╝Űíť ýâŁýä▒ŰÉśýžÇ ýĽŐŰŐöŰőĄŰę┤, ýžüýáĹ ý╗ĘÝůîýŁ┤ŰäłŰą╝ ýáĽýŁśÝĽśŕ│á ŕ┤ÇŰŽČÝĽśŰŐö ŕ▓âýŁ┤ ÝĽäýÜöÝĽśŰőĄ.

### Default tablespaces
when you create a new database, the database manager creates some default tablespaces for database. There tablespace is used as a storage for user and temporary data. Each database must contain at lease three tablespaces as given here:

ýâłŰíťýÜ┤ ŰŹ░ýŁ┤Ýä░Ű▓áýŁ┤ýŐĄŰą╝ ýâŁýä▒ÝĽśŰę┤, ŰŹ░ýŁ┤Ýä░Ű▓áýŁ┤ýŐĄ ŕ┤ÇŰŽČý×ÉŰŐö ŰŹ░ýŁ┤Ýä░Ű▓áýŁ┤ýŐĄŰą╝ ýťäÝĽ┤ default tablespacesŰą╝ ýâŁýä▒ÝĽťŰőĄ. tablespacesŰŐö ýéČýÜęý×ÉýÖÇ ý×äýőť ŰŹ░ýŁ┤Ýä░Űą╝ ýťäÝĽť ýáÇý×ą ŕ│Áŕ░äýť╝Űíť ýéČýÜęŰÉťŰőĄ. ŕ░ü ŰŹ░ýŁ┤Ýä░Ű▓áýŁ┤ýŐĄŰŐö Ű░śŰôťýőť ýú╝ýľ┤ýžä ýŁ┤ ŕ▓âŰôĄŕ│╝ ŕ░ÖýŁÇ tablespaceŰôĄýŁä ÝĆČÝĽĘÝĽťŰőĄ:

1. **Catalog tablespace**
2. **User tablespace**
3. **Temporary tablespaces**

<br/>

**Catalog tablespace :** It contains ***system catalog┬▓*** tables for the database. It is named as SYSCATSPACE and it cannot be dropped. ŰŹ░ýŁ┤Ýä░Ű▓áýŁ┤ýŐĄŰą╝ ýťäÝĽť ***System Catalog┬▓*** ÝůîýŁ┤ŰŞöŰôĄýŁä ÝĆČÝĽĘÝĽťŰőĄ. **SYSCATSPACE**Űíť Ű¬ůŰ¬ůŰÉśŰę░ dropÝĽá ýłś ýŚćŰőĄ.

**User tablespace**: This tablespace contains user-defined tables. In a database, we have one default user tablespace, named as USERSPACE1. If you do not specify user-defined tablespace for a table at the time you create it, then the database manager chooses default user tablespace for you. ýŁ┤ tablespaceŰŐö ýéČýÜęý×ÉýŚÉ ýŁśÝĽ┤ ýáĽýŁśŰÉť ÝůîýŁ┤ŰŞöŰôĄýŁä ÝĆČÝĽĘÝĽťŰőĄ. ŰŹ░ýŁ┤Ýä░Ű▓áýŁ┤ýŐĄýŚÉýäťŰŐö, USERSPACE1Űíť Ű¬ůŰ¬ůŰÉť ÝĽśŰéśýŁś ýéČýÜęý×É ÝůîýŁ┤ŰŞöýŁä ŕŞ░Ű│Ş ŕ░ĺýť╝Űíť ŕ░ÇýžäŰőĄ. ÝůîýŁ┤ŰŞö ýâŁýä▒ Űő╣ýőťýŚÉ ýéČýÜęý×É ýáĽýŁśŰÉť ÝůîýŁ┤ŰŞöýŁä ÝŐ╣ýáĽÝĽśýžÇ ýĽŐýť╝Űę┤, ŰŹ░ýŁ┤Ýä░Ű▓áýŁ┤ýŐĄ ŕ┤ÇŰŽČý×ÉŰŐö default ýéČýÜęý×É ÝůîýŁ┤ŰŞöýŁä ýäáÝâŁÝĽśŕ▓î ŰÉťŰőĄ.

**Temporary and storage management**: Tablespaces can be setup in different ways, depending on how you want to use them. You can setup the operating system to manage tablespace allocation, you can let the database manager allocate space or you can choose automatic allocation of tablespace for your data. tablespacesŰŐö ýľ┤Űľ╗ŕ▓î ýéČýÜęÝĽśŰŐö ýžÇýŚÉ Űö░ŰŁ╝ ŰőĄŰąŞ Ű░ęýőŁýť╝Űíť ýäĄýáĽÝĽá ýłś ý×łŰőĄ. tablespace ÝĽáŰő╣ýŁä ŕ┤ÇŰŽČÝĽá operating systemýŁä ýäĄýáĽÝĽá ýłś ý×łýť╝Űę░, database ŕ┤ÇŰŽČý×Éŕ░Ç ŕ│Áŕ░äýŁä ÝĽáŰő╣ÝĽśŕ▓î ÝĽśŕ▒░Űéś ŰŹ░ýŁ┤Ýä░Űą╝ ýťäÝĽť tablespaceýŁś ý×ÉŰĆÖ ÝĽáŰő╣ýŁä ýäáÝâŁÝĽá ýłś ý×łŰőĄ.

***System Catalog┬▓* :** A system catalog is a group of tables and views that incorporate vital details regarding a database. Every database comprised of a system catalog and the information in the system catalog specifies the framework of the database. e.g. the data dictionary/data definition language(DDL) for every table in the database is saved in the system catalog. ***system catalog**ŰŐö ŰŹ░ýŁ┤Ýä░ýŚÉ ŕ┤ÇÝĽ┤ ÝĽäýłśýáüýŁŞ ýäŞŰÂÇ ýéČÝĽşŰôĄýŁä ÝĆČÝĽĘÝĽť viewýÖÇ tableýŁś ŕĚŞŰú╣ýŁ┤ŰőĄ. Ű¬ĘŰôá databaseŰŐö system catalogýÖÇ system catalogŕ░Ç ÝŐ╣ýáĽÝĽť databaseýŁś framework Űé┤ ýáĽŰ│┤ŰôĄŰíť ŕÁČýä▒ŰÉťŰőĄ. ýśłŰą╝ ŰôĄýľ┤, ŰŹ░ýŁ┤Ýä░Ű▓áýŁ┤ýŐĄ Űé┤ýŁś Ű¬ĘŰôá ÝůîýŁ┤ŰŞöŰôĄýŁä ýťäÝĽť DDLýŁ┤ system catalogýŚÉ ýáÇý×ąŰÉśýľ┤ ý×łŰőĄ.

<br/>
<h4>The following three types of managed spaces are available :</h4>
**System Managed Space(SMS)**: The operating system's file system manager allocates and manages the space where the table is stored. Storage space is allocated on demand. This model consists of files representing database objects. This tablespace type has been deprecated in Version 10.1 for user-defined tablespaces, and it is not deprecated for catalog and temporary tablespaces. ýÜ┤ýśü ýőťýŐĄÝůťýŁś ÝîîýŁ╝ ýőťýŐĄÝůť ŕ┤ÇŰŽČý×ÉŰŐö ÝůîýŁ┤ŰŞöýŁ┤ ýáÇý×ąŰÉť ŕ│Áŕ░äýŁä ÝĽáŰő╣ÝĽśŕ│á ŕ┤ÇŰŽČÝĽťŰőĄ. ýáÇý×ą ŕ│Áŕ░äýŁÇ ýÜöŕÁČýŚÉ Űö░ŰŁ╝ ÝĽáŰő╣ŰÉťŰőĄ. ýŁ┤ Ű¬ĘŰŹŞýŁÇ ŰŹ░ýŁ┤Ýä░Ű▓áýŁ┤ýŐĄ ŕ░Łý▓┤ýŁś ŰîÇÝĹť ŕ▓ę ÝîîýŁ╝ŰôĄŰíť ŕÁČýä▒ŰÉťŰőĄ. ýŁ┤ tablespace ÝâÇý×ůýŁÇ ýéČýÜęý×É-ýáĽýŁśŰÉť tablespaceŰôĄýŁÇ Ű▓äýáä 10.1ýŚÉ deprecated ŰÉśýŚłŕ│á, catalog(= SYSCAT)ŕ│╝ temporary tablespacesŰŐö deprecated ŰÉśýžÇ ýĽŐýĽśŰőĄ.

**Database Managed Space(DMS):** The Database Server controls the storage space. Storage space is pre-allocated on the file system based on container definition that you specify when you create the DMS table space. It is deprecated from version 10.1 fix pack 1 for user-defined tablespaces, but it is not deprecated for system tablespace and temporary tablespace.
ŰŹ░ýŁ┤Ýä░Ű▓áýŁ┤ýŐĄ ýäťŰ▓äŰŐö ýáÇý×ą ŕ│Áŕ░äýŁä ýáťýľ┤ÝĽťŰőĄ. ýáÇý×ą ŕ│Áŕ░äýŁÇ DMS tablespaceŰą╝ ýâŁýä▒ÝĽá ŰĽî Ű¬ůýőťÝĽť, ý╗ĘÝůîýŁ┤Űäł ýáĽýŁśýŚÉ ŕŞ░Ű░śÝĽśýŚČ ÝîîýŁ╝ ýőťýŐĄÝůťýŚÉ ÝĽáŰő╣ŰÉśýľ┤ ý×łŰőĄ. ýŁ┤ŰŐö 10.1 Ű▓äýáäŰÂÇÝä░ ýéČýÜęý×É-ýáĽýŁśŰÉť tablespacesŰą╝ ýťäÝĽť ***fix pack┬│*** 1ýŚÉ deprecated ŰÉśýŚłýť╝Űę░, system tablespacesýÖÇ temporary tablespacesŰŐö deprecated ŰÉśýžÇ ýĽŐýĽśŰőĄ.

 ***fix pack┬│** : A cumulative collection of fixes that is made available between scheduled refresh packs, manufacturing refreshes or releases. it is intended to allow customers to move to specific maintenance level.*

**Automatic Storage Tablespace:** Database server can be managed automatically. Database server creates and extends containers depend on data on database. With automatic storage management, it is not required to provide container definitions. The database server looks after creating and extending containers to make use of the storage allocated to the database. If you add storage space to a storage group, new containers are automatically created when the existing container reach their maximum capacity. if you want to use the newly-added immediately, you can rebalance the tablespace. ŰŹ░ýŁ┤Ýä░Ű▓áýŁ┤ýŐĄ ýäťŰ▓äŰŐö ý×ÉŰĆÖýť╝Űíť ŕ┤ÇŰŽČŰÉá ýłś ý×łŰőĄ. ŰŹ░ýŁ┤Ýä░Ű▓áýŁ┤ýŐĄ ýäťŰ▓äŰŐö ŰŹ░ýŁ┤Ýä░Ű▓áýŁ┤ýŐĄýŚÉ ýśČŰáĄýžä ŰŹ░ýŁ┤Ýä░ýŚÉ Űö░ŰŁ╝ýäť ý╗ĘÝůîýŁ┤ŰäłŰôĄýŁä ýâŁýä▒ÝĽśŕ│á ÝÖĽý×ąÝĽťŰőĄ. ý×ÉŰĆÖ ýáÇý×ąýćî ŕ┤ÇŰŽČŰą╝ ýéČýÜęÝĽťŰőĄŰę┤ ý╗ĘÝůîýŁ┤Űäł ýáĽýŁśýŁś ŕĚťýáĽýŁÇ ÝĽäýłśŕ░Ç ýĽäŰőłŰőĄ. ŰŹ░ýŁ┤Ýä░Ű▓áýŁ┤ýŐĄ ýäťŰ▓äŰŐö ŰŹ░ýŁ┤Ýä░Ű▓áýŁ┤ýŐĄýŚÉ ÝĽáŰő╣ŰÉť ýáÇý×ąýćîŰôĄýŁä ýéČýÜęÝĽśŕŞ░ ýťäÝĽ┤ ý╗ĘÝůîýŁ┤Űäł ýâŁýä▒ Ű░Ć ÝÖĽý×ąýŁä ýłśÝľëÝĽťŰőĄ. storage groupýŚÉ ýáÇý×ą ŕ│Áŕ░äýŁä ýÂöŕ░ÇŰíť ŰŹöÝĽťŰőĄŰę┤, ýâłŰíťýÜ┤ ý╗ĘÝůîýŁ┤ŰäłŰŐö ŕŞ░ýí┤ýŁś ý╗ĘÝůîýŁ┤Űäłŕ░Ç ŕĚŞŰôĄýŁś ýÁťŰîÇ ýÜęŰčëýŚÉ ŰĆäŰőČÝľłýŁä ŰĽî ý×ÉŰĆÖýť╝Űíť ýâŁýä▒ŰÉťŰőĄ. ýâłŰíť ýÂöŕ░ÇŰÉť ŕ▓âýŁä ýŽëýőť ýéČýÜęÝĽśŕ│áý×É ÝĽťŰőĄŰę┤ tablespaceŰą╝ ý×Č ýí░ýáĽÝĽá ýłś ý×łŰőĄ.

**Page, table and tablespace size:** Temporary DMS and automatic storage tablespaces, the page size you choose for your database determines the maximum limit for the tablespace size. For table SMS and temporary automatic storage tablespaces, the page size constrains the size of table itself. The page sizes can be 4kb, 8kb, 16kb, 32kb. ý×äýőť DMS(Database managed Space)ýÖÇ ý×ÉŰĆÖ ýáÇý×ąýćî tablespaces, ŰŹ░ýŁ┤Ýä░Ű▓áýŁ┤ýŐĄŰą╝ ýťäÝĽť page sizeŰŐö tablespaceýŁś ýÁťŰîÇ ÝüČŕŞ░ýŁś ÝĽťŰĆäŰą╝ ŕ▓░ýáĽÝĽťŰőĄ. table System managed SpaceýÖÇ ý×äýőť ý×ÉŰĆÖ ýáÇý×ąýćî tablespaces, ÝÄśýŁ┤ýžÇ ÝüČŕŞ░ŰŐö ÝůîýŁ┤ŰŞöýŁś ÝüČŕŞ░Űą╝ ýŐĄýŐĄŰíť ýáťÝĽťÝĽťŰőĄ. page ÝüČŕŞ░ŰŐö 4kb, 8kb, 16kb, 32kbýŁ╝ ýłś ý×łŰőĄ. 

## 5-4. Syntax
## 5-5. Remark

---