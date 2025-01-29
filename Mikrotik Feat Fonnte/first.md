
##Check Send Message Fonnte Via Terminal :speech_balloon:
/tool fetch url="https://api.fonnte.com/send" http-header-field="Authorization:#TOKEN" http-method=post http-data="target=#NOHP&message=Tes API dari MikroTik" output=user


------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
+++Netwatch Check Status Connection UP/DOWN+++
:local hh $host
:local wa "#NOHP"
:local datetime "Tanggal:%20$[/system clock get date]%0AJam:%20$[/system clock get time]";
/tool fetch url="https://api.fonnte.com/send" http-header-field="Authorization:#TOKEN" http-method=post http-data="target=$wa&message=\E2\9C\85 JALUR KITA UP%0A$datetime%0AServer:%20$hh%0AStatus:%20ON" keep-result=no;

:local hh $host
:local wa "#NOHP"
:local datetime "Tanggal:%20$[/system clock get date]%0AJam:%20$[/system clock get time]"
:local com [/tool netwatch get value-name=comment [find host=$hh] comment];
/tool fetch url="https://api.fonnte.com/send" http-header-field="Authorization:#TOKEN" http-method=post http-data="target=$wa&message=\E2\9D\8C JALUR KITA DOWN %0A$datetime%0ARouter:%20$hh%20ON" keep-result=no;
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------



#NOHP = isi dengan nomer tujuan pesan (jika dikirim ke grup pastikan dahulu idgrup mu lalu tambahkan @g.us)
#TOKEN = isi dengan token fonnte
