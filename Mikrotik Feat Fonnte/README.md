## Mikrotik Feat Fonnte 🚀
Connect Mikrotik Whatapp With Fonnte for status connection POP or SITE and Server and then the user PPPOE in profile with script.

🎖️ **Tested on Mikrotik v.7.0**


Send Message Fonnte Via Terminal :speech_balloon:
```
/tool fetch url="https://api.fonnte.com/send" http-header-field="Authorization:@token" http-method=post http-data="target=@nohp&message=Tes API dari MikroTik" output=user
```

PPPOE Profile Script Status UP :heavy_check_mark:
```
:local nama "$user";
:local wa "@nohp";
:local ips [/ppp active get [find name=$nama] address];
:local up [/ppp active get [find name=$nama] uptime];
:local caller [/ppp active get [find name=$nama] caller-id];
:local service [/ppp active get [find name=$nama] service];
:local active [/ppp active print count];
:local datetime "Tanggal: $[/system clock get date] %0AJam: $[/system clock get time]";
:local lastdisc [/ppp secret get [find name=$user] last-disconnect-reason];
:local lastlogout [/ppp secret get [find name=$user] last-logged-out];
:local lastcall [/ppp secret get [find name=$user] last-caller-id];
/tool fetch url="https://api.fonnte.com/send" http-header-field="Authorization@token" http-method=post http-data="target=$wa&message=\E2\9C\85 PPPoE LOGIN%0A$datetime%0AUser: $user%0AIP Client: $ips%0ACaller ID: $caller%0AUptime: $up%0ATotal Active: $active Client%0AService: $service%0ALast Disconnect Reason: $lastdisc %0ALast Logout: $lastlogout %0ALast Caller ID: $lastcall" keep-result=no;
```
PPPOE Profile Script Status DOWN :x:
```
:local wa "@nohp";
:local lastdisc [/ppp secret get [find name=$user] last-disconnect-reason];
:local lastlogout [/ppp secret get [find name=$user] last-logged-out];
:local lastcall [/ppp secret get [find name=$user] last-caller-id];
:local active [/ppp active print count];
:local datetime "Tanggal: $[/system clock get date] %0AJam: $[/system clock get time]";
/tool fetch url="https://api.fonnte.com/send" http-header-field="Authorization:@token" http-method=post http-data="target=$wa&message=\E2\9D\8CPPPOE-TERPUTUS %0A$datetime%0AUSER: $user%0ALast Disconnect Reason: $lastdisc %0ALast Logout: $lastlogout %0ALast Caller ID: $lastcall %0ATotal active: $active Client"  keep-result=no;
```
Netwatch Check Status UP :heavy_check_mark:
```
:local hh $host
:local wa "@nohp"
:local datetime "Tanggal:%20$[/system clock get date]%0AJam:%20$[/system clock get time]";
/tool fetch url="https://api.fonnte.com/send" http-header-field="Authorization:@token" http-method=post http-data="target=$wa&message=\E2\9C\85 JALUR KITA UP%0A$datetime%0AServer:%20$hh%0AStatus:%20ON" keep-result=no;
```

Netwatch Check Status DOWN :x:
```
:local hh $host
:local wa "@nohp"
:local datetime "Tanggal:%20$[/system clock get date]%0AJam:%20$[/system clock get time]"
:local com [/tool netwatch get value-name=comment [find host=$hh] comment];
/tool fetch url="https://api.fonnte.com/send" http-header-field="Authorization:@token" http-method=post http-data="target=@wa&message=\E2\9D\8C JALUR KITA DOWN %0A$datetime%0ARouter:%20$hh%20ON" keep-result=no;
```

	
:pushpin: __Spesial Noted in Script__
| Variable     | Information |
|:---------|:----|
|**@nohp**|add number-phone or grupid@g.us|
|**@token**|token fonnte|
