---
layout: post
date: 2021-02-25 00:00:10
title: "UnCrackable Wrtie Up"
categories: CTF
tags: [Android]
author:
  - Jeongwon Jo

---
## <span style="color:#21C587"></span> UnCrackable Level 1 

![](https://github.com/wjddnjs33/image/blob/main/UnCrackable/Level1/1.png?raw=true)

앱을 실행하면 루팅 탐지가 되었다는 팝업이 뜨는 것을 볼 수 있습니다.

```Java
public class MainActivity extends Activity {
    private void a(String str) {
        AlertDialog create = new AlertDialog.Builder(this).create();
        create.setTitle(str);
        create.setMessage("This is unacceptable. The app is now going to exit.");
        create.setButton(-3, "OK", new DialogInterface.OnClickListener() {
            /* class sg.vantagepoint.uncrackable1.MainActivity.AnonymousClass1 */

            @Override // android.content.DialogInterface.OnClickListener
            public void onClick(DialogInterface dialogInterface, int i) {
                System.exit(0);
            }
        });
        create.setCancelable(false);
        create.show();
    }
    
    (생략)
```
루팅 탐지 로직의 코드를 확인해보니 `MainActivity`에 있는 것을 볼 수 있고, `onClick()` 메서드를 이용해서 버튼을 클릭하면 `exit()` 메서드를 호출하는 것을 볼 수 있습니다. 그렇기 때문에 `OK` 버튼을 누르면 `exit()` 메서드에 의해서 프로그램이 종료가 되는 것 입니다. 일단 `onClick()` 메서드를 후킹해서 우회를 해보겠습니다.

```javascript
setImmediate(function(){
    Java.perform(function(){
        Java.choose("sg.vantagepoint.uncrackable1.MainActivity$1",{
            onMatch:function(a){
                a.onClick.implementation = function(){
                    console.log("\n[*] Root Detected Bypass");
                }
            },
            onComplete:function(){
                console.log("\n[*] Stage 1, Success")
            }
        });
    });
});
```
자바스크립트를 이용해서 위와 같이 익스 코드를 작성해주었습니다. `Java.choose()` 메서드를 이용해서 클래스를 불러와서 `onClick()` 메서드를 후킹해서 버튼을 클릭할 때 `exit()` 메서드가 아닌 `console.log()`가 실행이 되게 해주었습니다. 그리고 클래스를 불러올 때 `$1`를 붙혀주지 않으면 `onClick()` 메서드 후킹이 되지 않았습니다.

```
 py@py  ~/app/UnCrackable/Level1  frida -U -l exploit.js owasp.mstg.uncrackable1
     ____
    / _  |   Frida 12.11.7 - A world-class dynamic instrumentation toolkit
   | (_| |
    > _  |   Commands:
   /_/ |_|       help      -> Displays the help system
   . . . .       object?   -> Display information about 'object'
   . . . .       exit/quit -> Exit
   . . . .
   . . . .   More info at https://www.frida.re/docs/home/
Attaching...                                                            

[*] Stage 1, Success
[SM-G965N::owasp.mstg.uncrackable1]->  
[*] Root Detected Bypass
```
![](https://github.com/wjddnjs33/image/blob/main/UnCrackable/Level1/2.png?raw=true)

함수 후킹을 해주고 버튼을 클릭해주니 루팅 탐지가 잘 우회된 것을 볼 수 있습니다. 이제 특정한 값을 입력해주면 문제가 풀리게 됩니다.

![](https://github.com/wjddnjs33/image/blob/main/UnCrackable/Level1/3.png?raw=true)

아무 값이나 입력하고 보내주니 값이 틀렸다고, 다시 입력해주라는 팝업이 뜨는 것을 볼 수 있습니다.

```Java
    (생략)
    public void verify(View view) {
        String str;
        String obj = ((EditText) findViewById(R.id.edit_text)).getText().toString();
        AlertDialog create = new AlertDialog.Builder(this).create();
        if (a.a(obj)) {
            create.setTitle("Success!");
            str = "This is the correct secret.";
        } else {
            create.setTitle("Nope...");
            str = "That's not it. Try again.";
        }
        create.setMessage(str);
        create.setButton(-3, "OK", new DialogInterface.OnClickListener() {
            /* class sg.vantagepoint.uncrackable1.MainActivity.AnonymousClass2 */

            @Override // android.content.DialogInterface.OnClickListener
            public void onClick(DialogInterface dialogInterface, int i) {
                dialogInterface.dismiss();
            }
        });
        create.show();
    }
}
```
검증하는 함수를 찾아보니 `MainActivity` 제일 밑에 있었습니다. 코드를 보면 입력값을 `obj` 변수에 넣고, a 클래스의 `a()` 메서드의 인자값으로 `obj` 변수를 넘겨주고 있고, `a()`의 값이 참이면 문제가 풀리고, 거짓이면 아까와 같이 틀렸다고 팝업이 뜨는 로직인 것을 볼 수 있습니다.<br>

```Java
public class a {
    public static boolean a(String str) {
        byte[] bArr;
        byte[] bArr2 = new byte[0];
        try {
            bArr = sg.vantagepoint.a.a.a(b("8d127684cbc37c17616d806cf50473cc"), Base64.decode("5UJiFctbmgbDoLXmpL12mkno8HT4Lv8dlat8FxR2GOc=", 0));
        } catch (Exception e) {
            Log.d("CodeCheck", "AES error:" + e.getMessage());
            bArr = bArr2;
        }
        return str.equals(new String(bArr));
    }
```
그리고 위 코드가 a 클래스의 `a()` 메서드의 코드입니다. 코드를 보면 `sg.vantagepoint.a.a.a()` 메서드를 이용해서 특정한 값을 만들고, 만들어진 `bArr` 값과 우리가 입력한 값을 비교하는 것을 볼 수 있습니다. 이때 우리가 입력한 값과 위에서 만들어진 값이 동일하면 `true`를 반환해 문제가 풀리는 구조입니다.<br>

이제 생각해 볼 수 있는 건 `a()` 메서드를 후킹해서 반환값을 항상 참을 만들거나, `a()` 메서드에서 `bArr`의 값을 만들 때 사용되는 `sg.vantagepoint.a.a.a()`의 값을 뽑는 방법이 있겠습니다. 일단 2가지 익스를 모두 해보겠습니다.

```javascript
setImmediate(function(){
    Java.perform(function(){
        Java.choose("sg.vantagepoint.uncrackable1.MainActivity$1",{
            onMatch:function(a){
                a.onClick.implementation = function(){
                    console.log("\n[*] Root Detected Bypass");
                }
            },
            onComplete:function(){
                console.log("\n[*] Stage 1, Success")
            }
        });

        // Sollution 1
        const cls = Java.use("sg.vantagepoint.a.a");
        cls.a.implementation = function(a, b){
            const bytes = cls.a(a, b);
            const secret = '';
            for(var i = 0; i < bytes.length; ++i){
                secret += (String.fromCharCode(bytes[i] & 0xff));
            }
            console.log("\n[*] Find Secret : " + secret);
            console.log("\n[*] Stage 2, Success")
            return bytes;
        }

        // Sollution 2
        const cls2 = Java.use("sg.vantagepoint.uncrackable1.a");
        cls2.a.implementation = function(a){
            console.log("\n[*] Stage 2, Success")
            return true;
        }
    });
});
```
자바스크립트를 이용해서 위와 같이 `Sollution 1/2`를 모두 작성해주었습니다. `Sollution 1`은 `bArr`의 값을 만드는 `sg.vantagepoint.a.a.a()` 메서드를 후킹해서 여기서 만들어지는 값을 그대로 뽑아오는 것 익스입니다. 뽑아 오는 데이터는 바이트 객체이므로 `new Buffer`를 이용해서 문자열로 변환해주려 했지만 잘 되지 않아서 `String.fromCharCode()` 메서드를 이용해서 바이트 값을 문자열로 변환시켜주었습니다.<br>

`Sollution 2`는 a 클래스의 `a()` 메서드를 후킹해서 어떤 값을 입력해도 항상 `true`를 반환하게 만들어주었습니다.

```
 py@py  ~/app/UnCrackable/Level1  frida -U -l exploit.js owasp.mstg.uncrackable1
     ____
    / _  |   Frida 12.11.7 - A world-class dynamic instrumentation toolkit
   | (_| |
    > _  |   Commands:
   /_/ |_|       help      -> Displays the help system
   . . . .       object?   -> Display information about 'object'
   . . . .       exit/quit -> Exit
   . . . .
   . . . .   More info at https://www.frida.re/docs/home/
Attaching...                                                            

[*] Stage 1, Success
[SM-G965N::owasp.mstg.uncrackable1]->  
[*] Find Secret : I want to believe

[*] Stage 2, Success
```
`Sollution 1`을 이용해 후킹 해주니 Secret 값을 잘 뽑는 것을 볼 수 있습니다. Secret 값은 `"I want to believe"`입니다.

![](https://github.com/wjddnjs33/image/blob/main/UnCrackable/Level1/4.png?raw=true)

`Sollution 2`를 이용해 함수를 후킹하고, 아무 값이나 입력해줘도 잘 풀리는 것을 볼 수 있습니다.

---
## <span style="color:#21C587"></span> UnCrackable Level 2

![](https://github.com/wjddnjs33/image/blob/main/UnCrackable/Level2/1.png?raw=true)

앱을 실행하면 Level1과 마찬가지로 루팅 탐지가 되었다는 팝업이 뜨는 것을 볼 수 있습니다.

```java
public class MainActivity extends c {
    private CodeCheck m;

    static {
        System.loadLibrary("foo");
    }

    /* access modifiers changed from: private */
    /* access modifiers changed from: public */
    private void a(String str) {
        AlertDialog create = new AlertDialog.Builder(this).create();
        create.setTitle(str);
        create.setMessage("This is unacceptable. The app is now going to exit.");
        create.setButton(-3, "OK", new DialogInterface.OnClickListener() {
            /* class sg.vantagepoint.uncrackable2.MainActivity.AnonymousClass1 */

            @Override // android.content.DialogInterface.OnClickListener
            public void onClick(DialogInterface dialogInterface, int i) {
                System.exit(0);
            }
        });
        create.setCancelable(false);
        create.show();
    }
(생략)
```
코드를 보면 일단 `System.loadLibrary()` 메서드를 이용해서 어떤 라이브러리를 불러오는 것을 볼 수 있습니다. 그리고 밑에서는 `onClick()` 메서드를 이용해서 버튼을 누르면 `System.exit()` 메서드를 호출하는 것을 볼 수 있습니다. 해당 로직도 Level1과 같이 `onClick()` 메서드를 후킹해주면 됩니다.

```javascript
setImmediate(function(){
    Java.perform(function(){
        const cls = Java.use("sg.vantagepoint.uncrackable2.MainActivity$1");
        cls.onClick.implementation = function(){
            console.log("\n[*] Root Detected Bypass");
        }
    });
});
```
```
 py@py  ~/app/UnCrackable/Level2  frida -U -l exploit.js owasp.mstg.uncrackable2
     ____
    / _  |   Frida 12.11.7 - A world-class dynamic instrumentation toolkit
   | (_| |
    > _  |   Commands:
   /_/ |_|       help      -> Displays the help system
   . . . .       object?   -> Display information about 'object'
   . . . .       exit/quit -> Exit
   . . . .
   . . . .   More info at https://www.frida.re/docs/home/
                                                                                
[SM-G965N::owasp.mstg.uncrackable2]->  
[*] Root Detected Bypass
```
![](https://github.com/wjddnjs33/image/blob/main/UnCrackable/Level2/2.png?raw=true)

자바스크립트를 이용해서 위와 같이 루팅 탐지를 우회하는 익스 코드를 작성해서 후킹을 해주니 루팅 탐지를 잘 우회하는 것을 볼 수 있습니다. 이제 해야 할 건 Level1처럼 시크릿 키를 획득해서 입력해주면 되는 거 같습니다. 일단 verify 로직의 코드를 분석해보겠습니다.

```java
(생략)
    public void verify(View view) {
        String str;
        String obj = ((EditText) findViewById(R.id.edit_text)).getText().toString();
        AlertDialog create = new AlertDialog.Builder(this).create();
        if (this.m.a(obj)) {
            create.setTitle("Success!");
            str = "This is the correct secret.";
        } else {
            create.setTitle("Nope...");
            str = "That's not it. Try again.";
        }
        create.setMessage(str);
        create.setButton(-3, "OK", new DialogInterface.OnClickListener() {
            /* class sg.vantagepoint.uncrackable2.MainActivity.AnonymousClass3 */

            @Override // android.content.DialogInterface.OnClickListener
            public void onClick(DialogInterface dialogInterface, int i) {
                dialogInterface.dismiss();
            }
        });
        create.show();
    }
}
```
위 코드 바로 위에 디버깅을 탐지하는 코드가 있는데 해당 부분은 문제 풀 때 필요가 없는 거 같아서 분석하지 않겠습니다. `verify()` 함수를 보면 입력값을 받아 와서 `obj` 변수에 넣어주고, `this.m.a()` 메서드에 `obj` 변수를 넘겨주고 있는 것을 볼 수 있습니다. 여기서 `this`는 현재 객체인 `MainActivity`를 가르키고, `m`은 아까 맨 위에서 `private CodeCheck m;`와 같이 `m`을 선언했기 때문에 `CodeCheck` 클래스를 가르키게 됩니다. 그럼 `CodeCheck` 클래스 내부에 있는 `a()` 메서드를 분석해보겠습니다.

```java
package sg.vantagepoint.uncrackable2;

public class CodeCheck {
    private native boolean bar(byte[] bArr);

    public boolean a(String str) {
        return bar(str.getBytes());
    }
}
```
`CodeCheck` 메서드를 확인해보니 복잡한 로직은 없습니다. 그냥 입력값을 받아와서 `getBytes()` 메서드를 이용해 문자열을 바이트로 변환해주고, 변환된 값을 네이티브 함수인 `bar()`에 인자로 넘겨주고 있는 것을 볼 수 있습니다. 네이티브 함수는 C/C++로 구현된 함수인데 이는 라이브러리 파일을 통해 분석할 수 있습니다. 라이브러리 파일은 아까 `System.loadLibrary("foo");`에 의해서 로드가 되었습니다. 이제 ida를 이용해 라이브러리 파일을 확인해보겠습니다.<br>

라이브러리 파일은 apk 파일을 unzip 하게 되면 `/lib/x86`에 존재합니다.

![](https://github.com/wjddnjs33/image/blob/main/UnCrackable/Level2/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202021-03-02%20%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE%209.08.49.png?raw=true)

일단 ida로 라이브러리 파일을 열어서 search 기능을 이용해 `bar`라고 검색을 해보니 위와 같이 `Java_sg_vantagepoint_uncrackable2_CodeCheck_bar()` 함수가 존재하는 것을 볼 수 있었습니다.

![](https://github.com/wjddnjs33/image/blob/main/UnCrackable/Level2/ida-1.png?raw=true)

`Java_sg_vantagepoint_uncrackable2_CodeCheck_bar()` 함수를 분석해보면 제일 하단에 `strncmp()` 함수를 이용해서 `v3`와 `(const char *)&s2`의 값을 23 바이트만큼 비교하는 것을 볼 수 있습니다. 이때 비교를 해서 비교한 값이 23 바이트 모두 일치하면 1을 반환하고, 그렇지 않으면 0을 반환하는 구조입니다. 그리고 바로 위에 if문이 하나 존재하는데 특정 값이 23이 아니면 goto 문을 이용해서 0을 반환하는 곳으로 이동하게 되는데 아마 해당 로직이 우리가 입력한 값의 길이를 검증하는 거 같습니다. 이유는 바로 밑에서 비교하는 길이가 23바이트이니 23바이트보다 크거나 작은 값은 검증할 필요도 없기 때문인 거 같습니다.<br>

그럼 이렇게 분석을 해본 결과 `strncmp()` 함수의 첫 번째 값이 우리가 입력한 값이고, 두 번째 값이 시크릿 값을 거라는 추측을 할 수 있습니다. 그럼 우리는 `strncmp()` 함수를 후킹해서 그냥 두 번째 인자 값을 긁어오면 시크릿 값을 알 수 있습니다.

```javascript
setImmediate(function(){
    Java.perform(function(){
        const cls = Java.use("sg.vantagepoint.uncrackable2.MainActivity$1");
        cls.onClick.implementation = function(){
            console.log("\n[*] Root Detected Bypass");
        }

        // native method hooking
        // strncmp addr leak
        const strncmp_addr = 0;
        const so = Module.enumerateImportsSync("libfoo.so")
        for(var i = 0; i < so.length; i++){
            if(so[i].name == 'strncmp'){strncmp_addr = so[i].address;console.log("[*] strncmp addr : " + strncmp_addr)}
        }
        // strncmp hooking
        Interceptor.attach(strncmp_addr, {
            // strncmp(args[0], args[1], args[2]);
            onEnter: function(args){              
                if(args[2] == 23){
                    console.log("[*] Secret Key : " + Memory.readCString(args[1]));
                }
            }
        });
    });
});
```
자바스크립트를 이용해서 위와 같이 익스 코드를 작성해주었습니다. 제일 먼저 `Module.enumerateImportsSync()` 메서드를 이용해서 라이브러리 파일 내에 있는 함수들을 모두 긁어와서 for문을 이용해 이름이 `strncmp`인 함수의 주소를 릭을 해주었습니다.<br>

그리고 라이브러리 내에 있는 함수를 조작하려면 `Interceptor.attach()` 메서드를 이용해야 합니다. 그래서 해당 메서드를 이용해서 `strncmp()` 함수에서 3번 째 인자, 즉 23 바이트만큼 비교하는 `strncmp()` 함수에 2번째 인자값을 모두 출력하게 해주었습니다. 2번째 인자값에는 우리가 입력한 값과 비교하는 값이 들어있기 때문에 해당 값을 뽑아주면 시크릿 키를 뽑을 수 있습니다. 그리고 문제를 다 풀고 알게 된 것인데 `strncmp()` 함수의 주소를 직접 릭을 하지 않아도 `Module.getExportByname("libaryName", "functionName")`를 이용해서 주소를 바로 긁어 올 수 있습니다.

```
 py@py  ~/app/UnCrackable/Level2  frida -U -l exploit.js owasp.mstg.uncrackable2
     ____
    / _  |   Frida 12.11.7 - A world-class dynamic instrumentation toolkit
   | (_| |
    > _  |   Commands:
   /_/ |_|       help      -> Displays the help system
   . . . .       object?   -> Display information about 'object'
   . . . .       exit/quit -> Exit
   . . . .
   . . . .   More info at https://www.frida.re/docs/home/
Attaching...                                                            
[*] strncmp addr : 0xd3d1bfc0
[SM-G965N::owasp.mstg.uncrackable2]-> [*] Secret Key : persist.sys.ui.hw
[*] Secret Key : Thanks for all the fish
[*] Secret Key : debug.layout
(생략)
```
익스 코드를 이용해서 함수를 후킹해주니 `strncmp()` 함수의 주소 릭이 잘 되는 것을 볼 수 있고, 입력폼에 `'a'`를 23 바이트만큼 입력을 해주니 `strncmp()` 함수가 실행돼서 두 번째 인자 값이 출력되는 것을 볼 수 있습니다. 아마도 두 번째로 출력된 `Thanks for all the fish`가 시크릿 값인 거 같습니다. 그리고 `'a'`를 23 바이트만큼 입력한 이유는 문제 로직에서 입력한 값의 길이가 23이여야 `strncmp()` 함수가 실행되기 때문입니다.

![](https://github.com/wjddnjs33/image/blob/main/UnCrackable/Level2/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202021-03-02%20%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE%209.32.58.png?raw=true)

`Thanks for all the fish`를 입력해주니 잘 풀리는 것을 볼 수 있습니다. 이번 문제를 통해서는 네이터브 함수 후킹을 하는 법을 알 수 있었습니다.

---
## <span style="color:#21C587"></span> UnCrackable Level 3

너무 어렵다.. ㅋㅋ

---
