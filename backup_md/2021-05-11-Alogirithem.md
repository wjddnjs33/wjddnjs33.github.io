---
layout: post
date: 2021-05-09 00:00:11
title: "BAE𝘒JOON Write Up"
categories: Algorithm
tags: [Algorithm, Python, C]
author:
  - Jeongwon Jo

---
요즘 롤, 넷플릭스, 웹해킹 공부만 순환으로 돌리다 보니 조금 지루해서 처음으로 백준을 풀어보기로 했다. 알고리즘 공부를 한 적이 없어 매우 화가 난다. 시간 복잡도와 같은 것들을 깐깐하게 맥이는 거 같아서 맞는 알고리즘을 짜도 정답 처리를 잘 해주지 않는다. 재미는 웹 해킹 미만 잡이다.

---
## Input and Output and arithmetic operations

> Stage 1<br>
 ```python
 print('Hello World!')
 ```
> Stage 2<br>
```python
print('강한친구 대한육군\n강한친구 대한육군')
```

> Stage 3<br>
```python
print('\    /\\')
print(' )  ( \')')
print('(  /  )')
print(' \(__)|')
```

> Stage 4<br>
```python
print('|\_/|')
print('|q p|   /}')
print('( 0 )"""\\')
print('|"^"`    |')
print('||_/=\\\\__|')
```

> Stage 5<br>
```python
n = input().split(' ')
print(int(n[0]) + int(n[1]))

n = input().split(' ')
print(int(n[0]) - int(n[1]))

n = input().split(' ')
print(int(n[0]) * int(n[1]))

n = input().split(' ')
print(int(n[0]) / int(n[1]))
```

> Stage 6<br>
```c
// 왠지 모르겠는데 파이썬으로 짜니 자꾸 틀렸다고 한다.
#include <stdio.h>

int main()
{
	int a, b, c;
	scanf("%d %d %d", &a, &b, &c);
	printf("%d\n", (a+b)%c);
	printf("%d\n", (a%c + b%c)%c);
	printf("%d\n", (a*b)%c);
	printf("%d\n", (a%c * b%c)%c);
 	return 0;
}
```

> Stage 7<br>
```python
n = int(input())
p = input()
result = []

result.append(n * int(p[2]))
result.append(n * int(p[1]))
result.append(n * int(p[0]))

print(result[0])
print(result[1])
print(result[2])
print(result[0] + (result[1]*10) + (result[2]*100))
```

---
## If Statement

> Stage 1<br>
```python
n = input().split(' ')

if int(n[0]) > int(n[1]):
    print('>')
elif int(n[0]) < int(n[1]):
    print('<')
elif int(n[0]) == int(n[1]):
    print('==')
```

> Stage 2<br>
```python
score = int(input())

if 90 <= score <= 100:
    print('A')
elif 80 <= score <= 89:
    print('B')
elif 70 <= score <= 79:
    print('C')
elif 60 <= score <= 69:
    print('D')
else:
    print('F')
```

> Stage 3<br>
```python
year = int(input())

if (year % 4 == 0) and ((year % 100 != 0) or (year % 400 == 0)):
    print(1)
else:
    print(0)
```

> Stage 4<br>
```python
p = int(input())
q = int(input())

if p > 0:
    if q > 0:
        print(1)
    else:
        print(4)
else:
    if q > 0:
        print(2)
    else:
        print(3)
```

> Stage 5<br>
```python
time = list(map(int, input().split(' ')))
h = time[0] == 0 and 24 or time[0]

if (time[1] - 45) >= 0:
    p = time[1] - 45
    print(h, p)
else:
    p = 60 - (abs(time[1] - 45))
    print(h - 1, p)
```

---
## For Statement

> Stage 1<br>
```python
N = int(input())

for i in range(9):
    print("{} * {} = {}".format(N, i + 1, N * (i + 1)))
```

> Stage 2<br>
```python
count = int(input())

for i in range(count):
    n = input().split(' ')
    print(int(n[0]) + int(n[1]))
```

> Stage 3<br>
```python
count = int(input())

result = 0
for i in range(count):
    result += i + 1
print(result)
```

> Stage 4<br>
```python
import sys

count = int(sys.stdin.readline().rstrip())
for i in range(count):
    n = sys.stdin.readline().rstrip().split(' ')
    print(int(n[0]) + int(n[1]))
```

> Stage 5<br>
```python
import sys

count = int(sys.stdin.readline().rstrip())
for i in range(count):
    print(i + 1)
```

> Stage 6<br>
```python
count = int(input())

for i in range(count, 0, -1):
    print(i)
```

> Stage 7<br>
```python
import sys

count = int(sys.stdin.readline().rstrip())
for i in range(count):
    n = sys.stdin.readline().rstrip().split(' ')
    print("Case #{}: {}".format(i+1, int(n[0]) + int(n[1])))
```

> Stage 8<br>
```python
import sys

count = int(sys.stdin.readline().rstrip())
for i in range(count):
    n = sys.stdin.readline().rstrip().split(' ')
    print("Case #{}: {} + {} = {}".format(i+1, n[0], n[1], int(n[0]) + int(n[1])))
```

> Stage 9<br>
```python
import sys

count = int(sys.stdin.readline().rstrip())
for i in range(count):
    print('*' * (i+1))
```

> Stage 10<br>
```python
import sys

count = int(sys.stdin.readline().rstrip())
j = 4
for i in range(count, 0, -1):
    print(' '*(i-1) + '*' * (5 - j))
    j = j - 1
```

> Stage 11<br>
```python
N, X = map(int, input().split())
A = list(map(int, input().split()))
for i in A:
    if i < X:
        print(i, end=" ")
```

---
## While Statement

> Stage 1<br>
```python
while(True):
    A, B = list(map(int, input().split()))
    if(A == 0 and B == 0):
        break
    else:
        print(A + B)
```

> Stage 2<br>
```python
try:
    while 1:
        A, B = list(map(int, input().split()))
        print(A + B)
except:
    exit()
```

> Stage 3<br>
```python
n = input()
cycle = 1
if 99 >= int(n) >= 10:
    f = n
    while(True):
        p = str(int(n[0]) + int(n[1]))
        n = n[1] + p[-1]
        if n == f:
            print(cycle)
            break
        else:
            cycle = cycle + 1
elif 0 <= int(n) < 10:
    n = '0' + n
    f = n
    while(True):
        p = str(int(n[0]) + int(n[1]))
        n = n[1] + p[-1]
        if n == f:
            print(cycle)
            break
        else:
            cycle = cycle + 1
```

---
## 1-dimensional array

> Stage 1<br>
```python
count = int(input())
p = list(map(int, input().split()))

minn, maxn = p[0], p[0]
for j in range(count):
    if minn > p[j]:
        minn = p[j]
        
    if maxn < p[j]:
        maxn = p[j]
        
print(minn, maxn)
```

> Stage 2<br>
```python
# Sollution 1
number_arr = []
for i in range(9):
    number_arr.append(int(input()))

max_value = number_arr[0]
max_value_index = 0

for i in range(len(number_arr)):
    if max_value < number_arr[i]:
        max_value = number_arr[i]
        max_value_index = i + 1
        
print(max_value)
print(max_value_index)

# Sollution 2
number_arr = []
for i in range(9):
    number_arr.append(int(input()))
        
print(max(number_arr))
print(number_arr.index(max(number_arr))+1)
```

> Stage 3<br>
```python
# Sollution 1
num1 = int(input())
num2 = int(input())
num3 = int(input())
result = str(num1 * num2 * num3)

for i in range(10):
    print(result.count(str(i)))

# Sollution 2
import sys
number = list(map(int, sys.stdin.readline().rstrip().split()))
result = str(number[0] * number[1] * number[2])

for i in range(10):
    print(result.count(str(i)))
```

> Stage 4<br>
```python
# set 함수를 사용하면 중복 값을 제거 해줌.
result = []
for i in range(10):
    result.append(int(input()) % 42)

print(len(set(result)))
```

> Stage 5<br>
```python
count = int(input())
score = list(map(int, input().split()))

max_value = score[0]
for s in score:
    if max_value < s:
        max_value = s

sum = 0
for i in range(count):
    sum += score[i]/max_value*100

print(sum/count)
```

> Stage 6<br>
```python
count = int(input())

for i in range(count):
    sum, value = 0, 0
    q = input()
    for j in range(len(q)):
        if q[j] == 'O':
            value += 1
        else:
            value = 0
        sum += value
    print(sum)
```

> Stage 7<br>
```python
test_count = int(input())

for i in range(test_count):
    sum, up_score = 0, 0
    information = list(map(int, input().split()))
    for j in range(1, len(information)):
        sum += information[j]
    avg = sum/information[0]

    for j in range(1, len(information)):
        if avg < information[j]: up_score += 1
    final = (up_score/information[0])*100
    print("{:.3f}%".format(final))
```

---
## Function

> Stage 1<br>
```python
def solve(a):
    ans = sum(a)
    return ans

print(solve(10))
```

> Stage 2<br>
```python
# 알고리즘이 헷갈려서 고민 중
```

> Stage 3<br>
```python
# None
```

---
## String

---
## Basic math 1

---
## Basic math 2

---
## Recursion

---
## Brute force

---
> 순서대로 풀거임

---
