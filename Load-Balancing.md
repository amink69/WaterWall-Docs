---
title: 14 - LoadBalancing 
layout: default
---

<div dir="rtl">

اینجا چند روش کاربردی لود بالانسینگ رو توضیح میدم روی کانفیگ های مستقیم و ریورس

.تعریف لود بالانسینگ : حالت اول یعنی یه سرور ایران داریم که به چند تا خارج وصل می شه و کاربر هارو بینشون پخش می کنه

حالت دوم هم که چند سرور ایران به یک خارج وصل بشن خیلی ساده هست و اصلا نیاز به قابلیت لود بالانسینگ اضافه کردن نداره ؛ صرفا روی سرور خارج برای هر سرور ایران یه کانفیگ اضافه می کنید؛ یه مثال هم توی .همین ازش می زنیم؛ ولی تمرکز این آموزش روی همون حالت اول هست

بالانسینگ چیز خیلی خوبیه ؛ احتمال فیلتری سرور خارج و یا ایران اکسس شدن سرور ایران رو خیلی کم می کنه و در عین حال دیتا سنترها هم دیگه نمی فهمن ترافیک داره یک به یک تونل میشه به یه مقصد



---

# Direct Tunnel 

برای توضیح لود بالانس من یه کانفیگ مستقیم رو اول نشان می دهنم

فرضیات:

ایپی سرور ایران 

1.1.1.1

ایپی سرور آلمان

2.2.2.2

ایپی سرور هلند

3.3.3.3

اول بریم یه پورت به پورت ساده بین ایران و آلمان برقرار کنیم

</div>

```json
{
    "name": "simple_port_to_port",
    "nodes": [
        {
            "name": "input",
            "type": "TcpListener",
            "settings": {
                "address": "0.0.0.0",
                "port": 443,
                "nodelay": true
            },
            "next": "output"
        },
        {
            "name": "output",
            "type": "TcpConnector",
            "settings": {
                "nodelay": true,
                "address": "1.1.1.1",
                "port": 443
            }
        }
    ]
}

```
<div dir="rtl">


این کانفیگ پورت به پورته ؛ نیاز به ران بودن واتروال روی سرور خارج نداره ولی اگه حالتی استفاده می کنید که نیاز داشت واتروال اجرا بشه در سرور خارج ؛ خیلی ساده در سرور های خارج کانفیگ مخصوص به خودشو اجرا کنید.

اینجا پورت ۴۴۳ سرور ایران را وصل کردیم به پورت ۴۴۳ سرور خارج آلمان ؛ حالا میخوام همین پورت ۴۴۳ به سرور هلند هم بشه وصل کرد و کاربری که به سرور ایران 
متصل میشه ؛ به صورت رندوم یا به المان وصل شه یا به هلند.


کانفیگ رو تغییر میدم و هلند رو اضافه میکنم.


</div>

```json
{
    "name": "simple_port_to_port_2_kharej",
    "nodes": [
        {
            "name": "input1",
            "type": "TcpListener",
            "settings": {
                "address": "0.0.0.0",
                "port": 443,
                "nodelay": true
            },
            "next": "output_alman"
        },
        {
            "name": "output_alman",
            "type": "TcpConnector",
            "settings": {
                "nodelay": true,
                "address": "1.1.1.1",
                "port": 443
            }
        },



       {
            "name": "input2",
            "type": "TcpListener",
            "settings": {
                "address": "0.0.0.0",
                "port": 443,
                "nodelay": true
            },
            "next": "output_holand"
        },
        {
            "name": "output_holand",
            "type": "TcpConnector",
            "settings": {
                "nodelay": true,
                "address": "2.2.2.2",
                "port": 443
            }
        }

    ]
}

```
<div dir="rtl">


اینجا ما ۲ تا input داریم روی یک پورت و کاملا شبیه به هم هستن ؛ اگه الان کانفیگ رو اجرا کنیم متوجه خواهیم شد که همه کاربر ها به input1 وصل می شوند در نتیجه همشون میرن آلمان

اما برای اینکه بالانسینگ شکل بگیره باید بین input1 و input2 یک balance-group ایجاد کنیم که خیلی ساده انجام میشه

</div>

```json
{
    "name": "simple_port_to_port_2_kharej",
    "nodes": [
        {
            "name": "input1",
            "type": "TcpListener",
            "settings": {
                "address": "0.0.0.0",
                "port": 443,
                "nodelay": true,
                "balance-group": "group name"
            },
            "next": "output_alman"
        },
        {
            "name": "output_alman",
            "type": "TcpConnector",
            "settings": {
                "nodelay": true,
                "address": "1.1.1.1",
                "port": 443
            }
        },



       {
            "name": "input2",
            "type": "TcpListener",
            "settings": {
                "address": "0.0.0.0",
                "port": 443,
                "nodelay": true,
                "balance-group": "group name"

            },
            "next": "output_holand"
        },
        {
            "name": "output_holand",
            "type": "TcpConnector",
            "settings": {
                "nodelay": true,
                "address": "2.2.2.2",
                "port": 443
            }
        }

    ]
}
```
<div dir="rtl">

اومدیم در Settings نود های input یک پارامتر جدید به اسم balance-group اضافه کردیم

این پارامتر یه اسم میگیره که دلخواه هست و ؛ نود های ورودی که این اسم گروه را داشته باشن با هم بالانس خواهند شد و بین خودشون کاربران را تقسیم خواهند کرد.

بعد اینکه یه کاربر وصل بشه ممکنه بیفته دست input 1 یا input 2 و تا زمانی که متصل باشه هم دست همون input خواهد بود ؛ اگه برای یک دقیقه کاربر متصل نباشه ( یعنی استفاده خاصی از vpn نکنه ؛ نه اینکه صرفا کامل v2rayng  رو قطع کنه)

اون موقع اگه دوباره وصل بشه ممکنه دست یه input جدید بیفته و مثلا به جای المان برسه به هلند

## balance-interval

اگه می خواهید برای تست یا هر منظوری این عدد ۱ دقیقه رو کم و زیاد کنید ؛ داخل همون Settings می توانید پارامتر balance-interval را اضافه کنید که باید عدد داخلش بزارید و به میلی ثانیه هم هست

این عدد رو نباید خیلی کم کنید؛ اگه بزارید مثلا ۱ ثانیه اونوقت کاربر خیلی سریع جابه جا میشه و وبسایت ها معمولا به چنین کاربری اجازه لاگین یا کار نمیدن و سایت برای طرف درست لود نمیشه و captcha ها هم گیر میکنن

پارامتر balance-interval رو می توانید برای هر input تعریف کنید ؛ معمولا کسی که میخواد تغییر بده برای همه نود ها عدد جدید رو یکی ست می کنه


اگه یکی ست نکنید مثلا برای input2 بیاید ۵ دقیقه ست کنید و برای input1 ؛‌ یک دقیقه ست کنید


اونوقت اینجوری کار می کنه که اگه کاربر به هلند وصل شد ؛‌۵ دقیقه باید کار نکنه تا دوباره شانسی به یه جا دیگه وصل بشه ؛ ولی اگه به آلمان وصل شد باید ۱ دقیقه کار نکنه تا دوباره شانسی بالانس بشه

این حرکت کاربردی نداره مگه اینکه واقعا طرف بخواد ترافیک بیشتری روی هلند ببره از المان

---

## نکته فنی

وقتی شما کاربری رو بین چند سرور خارج بالانس می کنید ؛ باید روی هردو سرور خارج برای کاربر یه پنل xui یا به هر حال یه راهی برای رسیدن به هسته وجود داشته باشه تا وصل بشه

معمولا یه بکاپ پنل روی ۲ سرور نصب میکنن ولی یه حالت کم طرفدار هم هست که روی سرور خارج دوم میان فوروارد میکنن به سرور خارج یک که در نهایت کاربر همیشه ایپی سرور ۱ را خواهد داشت ولی 

در سرور ایران ترافیک بین ۲ سرور خارج توضیع شده ؛ این نوع لودبالانس شاید تنها کاربردش کمتر کردن احتمال فیلتر شدن ایپی خارج باشه و یا مثلا وقتی هردو سرورتون آلمان هست و نمی خواهید دردسر ۲ تا پنل داشتن رو بکشید

---

## سرور سوم

اگه خواستید به این کانفیگی که ساختیم یه سرور مثلا فلاند هم اضافه کنید ؛ خیلی ساده عضو همون balance-group قرار میدید و تا 64 تا عضو میتونید داخل یک گروه اضافه کنید



ادامه این داکیومنت نوشته خواهد شد

</div>

[Homepage](.) | [Prev Page](HalfDuplex-Tunnel-or-Direct) | [Next Page](CDN-Tunnel)