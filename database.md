---
layout: page
title: /database
permalink: /database
---

### Spring Boot Theory (frequent update)

`21.05.2021-`
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
        [3-2-2. How Indexes are used by the Query Optimizer¹](#3-2-2-how-indexes-are-used-by-the-query-optimizer)
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
[WSL2 Mariadb 설치하기.](https://blog.dalso.org/linux/wsl2/16154)
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
    su root
{% endhighlight %}

---

# 2. Relational Data Modeling
## 2-0. Reference
[관계형 데이터 모델링 4.2. 관계형 데이터베이스 다운 개념의 구조](https://www.youtube.com/watch?v=hbZ96tnbN4M&list=PLuHgQVnccGMDF6rHsY9qMuJMd295Yk4sa&index=6&t=184s)
[데이터베이스 개념적 데이터 모델_개체-관계 모델 (1)](https://d9249.tistory.com/17?category=848157)
[Conceptual schema Wikipedia](https://en.wikipedia.org/wiki/Conceptual_schema)
[데이터 모델의 이해](https://dataonair.or.kr/db-tech-reference/d-guide/sql/?pageid=5&mod=document&uid=330)

## 2-1. Introduction
### 2-1-1. Data Modeling
데이터 모델링은 다음과 같은 3가지로 정의된다: 
    **1. 정보 시스템을 구축하기 위한 데이터 관점의 업무 분석 기법**
    **2. 현실의 데이터에 대해 약속된 표기법에 의해 표현하는 과정**
    **3. 데이터베이스를 구축하기 위한 분석 및 설계의 과정**
   
이 것을 보다 실무적으로 해석 해 보면 업무에서 필요로 하는 데이터를 시스템 구축 방법론에 의해 분석하고 설계하여 정보 시스템을 구축하는 과정으로 정의할 수 있다. 데이터 모델링을 하는 주요한 이유는 업무 정보를 구성하는 기초가 되는 정보들을 일정한 표기법에 의해 표현함으로써 정보 시스템 구축의 대상이 되는 업무 내용을 정확하게 분석하는 것이 첫 번째 목적이다. 두 번째는 분석된 모델을 통해 실제 데이터 베이스를 생성하여 개발 및 데이터 관리에 이용하기 위한 것이다.
즉, **데이터 모델링이라는 것은 단지 데이터 베이스만을 구축하기 위한 용도로만 쓰이는 것이 아니라 데이터 모델링 자체로서 업무를 설명하고 분석**하는 부분에도 매우 중요한 의미를 가지고 있다.

### 2-1-2.Conceptual-Logical-Physical Data Model
![img](/uploads/Untitled%20(111).png)
<br/>
특별히 데이터 모델은 데이터 베이스를 만들어 내는 설계서로서 분명한 목표를 가지고 있다. 현실 세계에서 데이터 베이스까지 만들어 지는 과정은 위와 같이 시간에 따라 진행되는 과정으로서 추상화 수준에 따라 개념적 데이터 모델(Conceptual Data Model | Conceptual Schema), 물리적 데이터 모델(Logical Data Model), 물리적 데이터 모델(Physical Data Model)로 정리 할 수 있다.


처음 현실 세계에서 추상화 수준이 높은 상위 수준을 형상화 하기 위해 개념적 데이터 모델링을 전개한다. **개념적 데이터 모델은 추상화 수준이 높고, 업무 중심적이며 포괄적인 수준의 모델링을 진행한다.** 참고로 *EA(Enterprise Architecture)¹* 기반의 전사적인 데이터 모델링을 전개할 때는 보다 상위 수준인 개괄적인 데이터 모델링을 먼저 수행하고, 이 후에 업무 영역에 따른 개념적 모델링을 전개한다. Entity 중심의 상위 수준 데이터 모델이 완성되면 **업무의 구체적인 모습과 흐름에 따른 구체화된 업무 중심의 데이터 모델을 만들어 내는데, 이 것이 논리적 데이터 모델이다. 논리적인 데이터 모델링 이후 데이터 베이스의 저장 구조에 따른 Table Space 등을 고려한 방식을 물리적인 데이터 모델링**이라고 한다. 이를 정리하면 아래의 표와 같다. 

![img](/uploads/Untitled%20(12).png)

### 2-1-3. Conceptual Data Modeling | Conceptual Schema
**개념적 데이터 베이스 설계는 조직, 사용자의 데이터 요구 사항을 찾고 분석하는 데에서 시작한다.이 과정은 어떠한 자료가 보다 중요하며 또 어떠한 자료가 유지되어야 하는 지를 결정하는 것**도 포함한다. 이 단계에 있어서 주요한 활동은 핵심 Entity와 그들 간의 관계를 발견하고, 그 것을 표현하기 위해서 Entity-Relationship Diagram(이하 ERD)을 생성하는 것이다. ERD는 조직과 다양한 데이터 베이스 사용자에게 어떠한 데이터가 중요한지 나타내기 위해서 사용된다. 데이터 모델링 과정이 전 조직에 걸쳐 이루어진다면, 그 것은 전사적 데이터 모델(Enterprise Data Model)이라고 불린다. 

개념적 데이터 모델을 통해 조직의 데이터 요구를 공식화 하는 것은 다음의 두 가지 중요한 기능을 지원하는데, **첫 째, 개념적 데이터 모델은 사용자와 시스템 개발자가 데이터 요구 사항을 발견하는 것을 지원한다.** 개념적 데이터 모델은 추상적이다. 그렇기 때문에 그 모델은 상위의 문제에 대한 구조화를 쉽게 만들며, 사용자와 개발자가 시스템 기능에 대해서 논의할 수 있는 기반을 형성한다. **둘 째 개념적 데이터 모델은 현 시스템이 어떻게 변형되어야 하는 가를 이해하는 데에 유용하다.** 일반적으로 매우 간단하게 고립된(Stand Alone) 시스템도 추상적 모델링을 통해서 보다 쉽게 표현되고 설명된다.


### 2-1-4. Logical Data Modeling
**논리적 데이터 모델링은 데이터 베이스 설계 프로세스의 input으로써, 비즈니스 정보의 논리적인 구조와 규칙을 명확하게 표현하는 기법 또는 과정**이라고 할 수 있다. 데이터 모델링이란 모델링 과정이 아닌 별도의 과정을 통해서 조사하고 결정한 사실을 단지 ERD라는 그림으로 그려내는 과정을 말하는 것이 아니다. 시스템 구축을 위해서 가장 먼저 시작할 기초적인 업무 조사를 하는 초기 단계부터 인간이 결정해야 할 대부분의 사항을 모두 정의하는 시스템 설계의 전 과정을 지원하는 '과정의 도구'라고 해야 할 것이다. 

**이 단계에서 수행하는 또 한 가지 중요한 과정은 정규화이다. 정규화는 논리 데이터 모델 상세화 과정의 대표적인 활동으로, 논리 데이터 모델의 일관성을 확보하고 중복을 제거하여 속성들이 가장 적절한 개체에 배치되도록 함으로써 보다 신뢰성 있는 데이터 구조를 얻는 데에 목적이 있다.** 논리 데이터 모델의 상세화는 식별자 확정, 정규화, m:m 관계 해소, 참조 무결성 규칙 정의 등을 들 수 있으며, 추가적으로 이력 관리에 대한 전략을 정의하여 이를 논리 데이터 모델에 반영함으로써 데이터 모델링을 완료하게 된다.


### 2-1-5. Physical Data Modeling
데이터 베이스 설계 과정의 세 번째 단계인 **물리 데이터 모델링은 논리 데이터 모델이 데이터 저장소로서 어떻게 컴퓨터 하드웨어에 표현될 것 인가를 다룬다.** 데이터가 물리적으로 컴퓨터에 어떻게 저장될 것 인가에 대한 정의를 Physical Schema라고 한다. 이 단계에서 결정되는 것은 Table, Column 등으로 표현되는 물리적인 저장 구조와 사용될 저장 장치, 자료를 추출하기 위해 사용될 접근 방법 등이 있다. 계층적 데이터 베이스 관리 시스템 환경에서는 데이터 베이스 관리자가 Physical Schema를 설계하고 구현하기 위해서 보다 많은 시간을 투자하여야 한다.

실질적인 프로젝트에서는 위의 모든 데이터 모델링을 수행하는 경우는 드물며, 개념적 데이터 모델링과 논리적 데이터 모델링을 한 번에 수행한 후 데이터 모델링으로 수행하는 경우가 대부분이다. 


## 2-2. Explanation
![img](/uploads/Untitled%20(13).png)

### 2-2-1. Entity
사람, 사물, 개념, 사건과 같이 독립적으로 존재하면서 고유하게 식별이 가능한 실 세계의 객체를 말 한다. 이 중, 저장할 가치가 있는 중요 데이터를 가지고 있는 사람, 사물, 개념, 사건 등을 추린 것이다. 이 들은 다른 개체와 구별되는 이름을 가지고 있으며, 각 개체 만의 고유한 특성이나 상태, 즉 속성(Attribute)를 반드시 하나 이상 소유한다.

### 2-2-2. Attribute
Mandatory(필수, Not Null) / Optional(부가, Nullable) 두 가지 경우가 존재할 수 있으며, 이는 개체나 관계가 가지고 있는 고유의 특성을 말 한다. 하나의 Entity는 연관된 Attribute의 집합으로 설명되며, 한 Attribute의 domain은 그 속성이 가질 수 있는 모든 값들의 집합을 의미한다. 여러 Attribute는 동일한 domain을 가질 수 있으며, key Attribute는 한 Attribute 또는 Attribute들의 모임으로서 한 Entity 타입 내에서 각 개체를 고유하게 식별한다.

이 중, 속성은 다양한 분류를 거쳐 작성 법이 달라지는데, 그 작성 법은 다음과 같다:

1. **Single-Valued Attribute (단일 값 속성)**
    1개의 값을 가지는, 즉 원자성의 속성이다.

2. **Multi-Valued Attribute (다중 값 속성)**
    여러 개의 값을 가질 수 있는 속성으로, 이중 선 타원으로 표현한다.

3. **Simple Attribute (단순 속성)**
    의미를 분해할 수 없는 속성이다.

4. **Composite Attribute (복합 속성)**
    의미의 분해가 가능한 속성으로, 세분화된 속성이다.

5. **Stored Attribute (저장 속성)**
    다른 속성의 값으로부터 얻을 수 없는 속성이다.

6. **Derived Attribute (파생 속성)**
    다른 속성의 값으로부터 유도되어 결정되는 속성이다.

### 2-2-3. Relationship
개체들 사이에 맺고 있는 연관 또는 연결, 대응(mapping)을 의미한다. 이들은 마름모 꼴의 형태로 작성하고, 서로 연관된 개체들을 실선으로 연결한다. 

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

### 3-2-2. How Indexes are used by the [Query Optimizer¹](https://en.wikipedia.org/wiki/Query_optimization#:~:text=Query%20optimization%20is%20a%20feature,considering%20the%20possible%20query%20plans.)
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
User의 인증에 문제가 있기 때문에 발생한다.

### Solution
다음과 같은 순서로 해결한다.

1. `su root` ubuntu shell 창을 켠 후, root 권한으로 실행한다.
2. `mysql -u root -p` 로 MariaDB를 실행한다.
    이는 root user 권한으로 password를 입력하여 실행한다는 의미이다.
3. `root@localhost`의 인증 방식을 패스워드로 설정하고, 그 패스워드를 root로 설정해 준다.
    {% highlight sql %}
    ALTER USER 'root'@'localhost' IDENTIFIED VIA mysql_native_password USING PASSWORD('root');
    {% endhighlight %}
4. 다시 한 번 connection 생성을 시도하면 성공적으로 완료된다.

---

# 5. Tablespaces
## 5-1. Reference
[DB2 Tablespaces](https://www.tutorialspoint.com/db2/db2_tablespaces.htm)

## 5-2. Introduction
A table space is a storage structure, it contains tables, indexes, large objects, and long data. It can be used to organize data in a database into logical storage which is related with where data stored on a system. This tablespaces are stored in database partition groups. 

Tablespace는 tables, indexes, 큰 객체들, 그리고 긴 데이터를 포함한 저장 구조이다. 이는 데이터베이스 안에서 시스템에서 데이터가 어디에 저장되어 있는 지와 연관된 논리적인 저장 공간 내에서 데이터를 조직화 하는 데에 쓰일 수 있다.

## 5-3. Explanation
### Benefits of tablespaces in database
The table spaces are beneficial in database in various ways given as follows:

**Recoverability:** Tablespaces make backup and restore operations more convenient. Using a single command, you can make backup or restore all the database objects in tablespaces.

**Automatic storage Management:** Database manager creates and extends containers depending on the needs.

**Memory utilization**: A single [bufferpool¹](https://www.ibm.com/docs/en/db2/11.5?topic=databases-buffer-pools) can manage multiple tablespaces. You can assign temporary tablespaces to their own bufferpool to increase the performance of activities such as sorts or joins.

**데이터베이스 내 Tablespace의 이점**
**복구 가능한:** Tablespace는 백업을 만들고 복구 운영하는 것을 더욱 편리하게 한다. 단일 명령어를 사용하는 것으로 tablespace 안의 모든 데이터베이스 객체들을 백업 또는 복구할 수 있다. 

**저장소 자동 관리:** 데이터베이스 관리자는 필요에 따라 컨테이너를 만들고 확장한다.

**메모리 활용 :** 단일 [Bufferpool¹](https://www.ibm.com/docs/en/db2/11.5?topic=databases-buffer-pools)이 복수의 Tablespace를 관리할 수 있다.    join이나 정렬과 같은 활동의 퍼포먼스를 높이기 위해 임시 Tablespace들을 그들이 소유한 bufferpool에 할당할 수 있다.

### Container
Tablespaces contains one or more containers. A container can be a directory name, a device name, or a filename. In a database, a single tablespace can have several containers on the same physical storage device. If the tablespace is created with automatic storage tablespace option, the creation and management of containers handled automatically by the database manager. If it is not created with automatic storage tablespace option, you need to define and manage the containers yourself.

**컨테이너**
Tablespace들은 하나 또는 그 이상의 컨테이너들을 포함한다. 하나의 컨테이너는 디렉토리명, 디바이스명, 또는 파일명이 될 수 있다. 데이터베이스에서, 단일 Tablespace는 동일한 물리적 저장 장치 위에 몇 컨테이너들을 가질 수 있다. 만약 Tablespace가 자동 storage Tablespace 옵션에 의해 생성되었다면, 컨테이너의 생성과 관리는 데이터베이스 관리자에 의해 자동으로 핸들링된다. 자동 storage tablespace 옵션으로 생성되지 않는다면, 직접 컨테이너를 정의하고 관리하는 것이 필요하다.

### Default tablespaces
when you create a new database, the database manager creates some default tablespaces for database. There tablespace is used as a storage for user and temporary data. Each database must contain at lease three tablespaces as given here:

새로운 데이터베이스를 생성하면, 데이터베이스 관리자는 데이터베이스를 위해 default tablespaces를 생성한다. tablespaces는 사용자와 임시 데이터를 위한 저장 공간으로 사용된다. 각 데이터베이스는 반드시 주어진 이 것들과 같은 tablespace들을 포함한다:

1. **Catalog tablespace**
2. **User tablespace**
3. **Temporary tablespaces**

<br/>

**Catalog tablespace :** It contains ***system catalog²*** tables for the database. It is named as SYSCATSPACE and it cannot be dropped. 데이터베이스를 위한 ***System Catalog²*** 테이블들을 포함한다. **SYSCATSPACE**로 명명되며 drop할 수 없다.

**User tablespace**: This tablespace contains user-defined tables. In a database, we have one default user tablespace, named as USERSPACE1. If you do not specify user-defined tablespace for a table at the time you create it, then the database manager chooses default user tablespace for you. 이 tablespace는 사용자에 의해 정의된 테이블들을 포함한다. 데이터베이스에서는, USERSPACE1로 명명된 하나의 사용자 테이블을 기본 값으로 가진다. 테이블 생성 당시에 사용자 정의된 테이블을 특정하지 않으면, 데이터베이스 관리자는 default 사용자 테이블을 선택하게 된다.

**Temporary and storage management**: Tablespaces can be setup in different ways, depending on how you want to use them. You can setup the operating system to manage tablespace allocation, you can let the database manager allocate space or you can choose automatic allocation of tablespace for your data. tablespaces는 어떻게 사용하는 지에 따라 다른 방식으로 설정할 수 있다. tablespace 할당을 관리할 operating system을 설정할 수 있으며, database 관리자가 공간을 할당하게 하거나 데이터를 위한 tablespace의 자동 할당을 선택할 수 있다.

***System Catalog²* :** A system catalog is a group of tables and views that incorporate vital details regarding a database. Every database comprised of a system catalog and the information in the system catalog specifies the framework of the database. e.g. the data dictionary/data definition language(DDL) for every table in the database is saved in the system catalog.
***system catalog**는 데이터에 관해 필수적인 세부 사항들을 포함한 view와 table의 그룹이다. 모든 database는 system catalog와 system catalog가 특정한 database의 framework 내 정보들로 구성된다. 예를 들어, 데이터베이스 내의 모든 테이블들을 위한 DDL이 system catalog에 저장되어 있다.*

<br/>
<h4>The following three types of managed spaces are available :</h4>
**System Managed Space(SMS)**: The operating system's file system manager allocates and manages the space where the table is stored. Storage space is allocated on demand. This model consists of files representing database objects. This tablespace type has been deprecated in Version 10.1 for user-defined tablespaces, and it is not deprecated for catalog and temporary tablespaces. 운영 시스템의 파일 시스템 관리자는 테이블이 저장된 공간을 할당하고 관리한다. 저장 공간은 요구에 따라 할당된다. 이 모델은 데이터베이스 객체의 대표 격 파일들로 구성된다. 이 tablespace 타입은 사용자-정의된 tablespace들은 버전 10.1에 deprecated 되었고, catalog(= SYSCAT)과 temporary tablespaces는 deprecated 되지 않았다.

**Database Managed Space(DMS):** The Database Server controls the storage space. Storage space is pre-allocated on the file system based on container definition that you specify when you create the DMS table space. It is deprecated from version 10.1 fix pack 1 for user-defined tablespaces, but it is not deprecated for system tablespace and temporary tablespace.
데이터베이스 서버는 저장 공간을 제어한다. 저장 공간은 DMS tablespace를 생성할 때 명시한, 컨테이너 정의에 기반하여 파일 시스템에 할당되어 있다. 이는 10.1 버전부터 사용자-정의된 tablespaces를 위한 ***fix pack³*** 1에 deprecated 되었으며, system tablespaces와 temporary tablespaces는 deprecated 되지 않았다.

 ***fix pack³** : A cumulative collection of fixes that is made available between scheduled refresh packs, manufacturing refreshes or releases. it is intended to allow customers to move to specific maintenance level.*

**Automatic Storage Tablespace:** Database server can be managed automatically. Database server creates and extends containers depend on data on database. With automatic storage management, it is not required to provide container definitions. The database server looks after creating and extending containers to make use of the storage allocated to the database. If you add storage space to a storage group, new containers are automatically created when the existing container reach their maximum capacity. if you want to use the newly-added immediately, you can rebalance the tablespace. 데이터베이스 서버는 자동으로 관리될 수 있다. 데이터베이스 서버는 데이터베이스에 올려진 데이터에 따라서 컨테이너들을 생성하고 확장한다. 자동 저장소 관리를 사용한다면 컨테이너 정의의 규정은 필수가 아니다. 데이터베이스 서버는 데이터베이스에 할당된 저장소들을 사용하기 위해 컨테이너 생성 및 확장을 수행한다. storage group에 저장 공간을 추가로 더한다면, 새로운 컨테이너는 기존의 컨테이너가 그들의 최대 용량에 도달했을 때 자동으로 생성된다. 새로 추가된 것을 즉시 사용하고자 한다면 tablespace를 재 조정할 수 있다.

**Page, table and tablespace size:** Temporary DMS and automatic storage tablespaces, the page size you choose for your database determines the maximum limit for the tablespace size. For table SMS and temporary automatic storage tablespaces, the page size constrains the size of table itself. The page sizes can be 4kb, 8kb, 16kb, 32kb.
임시 DMS(Database managed Space)와 자동 저장소 tablespaces, 데이터베이스를 위한 page size는 tablespace의 최대 크기의 한도를 결정한다. table System managed Space와 임시 자동 저장소 tablespaces, 페이지 크기는 테이블의 크기를 스스로 제한한다. page 크기는 4kb, 8kb, 16kb, 32kb일 수 있다. 

## 5-4. Syntax
## 5-5. Remark

---