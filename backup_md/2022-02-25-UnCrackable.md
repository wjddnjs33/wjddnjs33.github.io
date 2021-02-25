---
layout: post
date: 2021-02-23 00:00:10
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

---
## <span style="color:#21C587"></span> UnCrackable Level 3

---
