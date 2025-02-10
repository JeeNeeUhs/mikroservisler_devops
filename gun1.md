# Mikroservisler ve Devops

-[Mikroservisler ve Devops](#mikroservisler-ve-devops)
   -[Ozgur Yazilim Nedir](#ozgur-yazilim-nedir)
      -[kodlama lisanslari](#kodlama-lisanslari)
      -[genel icerik lisanslari](#genel-icerik-lisanslari)

## ozgur yazilim nedir?

### kodlama lisanslari
* lisansta 4 madde var. GNU GPL, GNU LGPL, GNU AGPL, GNU FDL
temel degisken, kendi kodunu yazilimin koduna da eklemen gerekir.
* apacik public license'ta ise farkli: apache license, mit license, bsd license
istersen kendi kodunu kapat, ister acik birakabilirsin.
* mozilla'da referans vermen lazim (hakan'in kodunu kullaniyorum.)

### genel icerik lisanslari
* CC BY-SA: creative commons, attribution, share alike
ikonlar, icerikler, fotograflar, javascriptler, cssler, html'ler
* sadece BY geciiyorsa benim adimi mutlaka orada soyleyeceksin
* NC: non commercial, ucretsiz dagitabilirsin. ticari amaclarla kullanamazsin:
  t-shirti istedigim gibi bastirabilirim ama bu basili olani satamam.
* github'ta public olarak yayinlansa bile alip kullanamazsin, herhangi bir kod parcasinin anonimligi yok.
* ffmpeg'in gpl'li oldugu icin yakalanip kodlari acik yayinlamak zorunda kaldilar.

ozgur yazilim olmayan bir seyde hic sansin yok?

## Mikroservis Mimarisine Gecis

Vikipedi'deki tanimi aldik. (buraya koyabilirsin)
ilk yazilan mainframe'ler de servis mimarisinde calisiyordu (RPC) ve
bir procedure 1 servis olarak kabul ediliyordu.
birbirleriyle konusabilmeleri icin kendi aralarinda bir networkleri vardi.
sun microsystems java'yi da iot icin gelistirdi. ama asil kitlesi kurumsal
sirketlerin main frame'leri. 90'larin basinda 10tane islemci mimarisi vardi.
java'yi kurumsal pazara getirmek istediler. RMI, as400'lerle de konusuyor, kucuk uygulama main frame'e bunulna baglanip  is yaptirip geri gelecek.
sonra webservis cikti, binary protokollerle is yapmak zor
html cagirmak yerine herkes kendi servis api'sini yazip, ama burada bir standart
yok, uygulama yaziyorum, a firmasinda farkli , b'de farkli
iste burada SOAP geldi. XML'i butun teknolojiler okuyabiliyor, herhangi bir SOAP servisi yazabilirsin.  wsdl ve ubdi
biz bunlara web servis demeyelim, jan jan li olsn; SoA Svice Oriented Architecture.


* Servis NEdir?
  * SoA: normal bir SOAP web servie olusturuyorsun, soa urunleri izliyor,ayaka kaldirmak icin aracgerecler sagliyor. onlari kontrol ediyor
  * RPC/RMI/IPC
  * WebService
  * SOAP
  * REST
  * gRPC: benim cok servisim var, surekli json dolastirip parse ederen surekli network trafigi olusuoyr, protobuff olarak bunu binary tanimlayayim. burada apache de avro mu diye bir binary protocol tanimladi burada apache de avro mu diye bir binary protocol tanimladi.
    http put-post yetmedi, bazen action command gondermen gerekiyor. tasarim deseni olarak yetmemeye basladi. parametrelerini tanimlayarak RPC yapmaya basladi, onun karsisinda rest console (?)
  * GraphQL: **hoca buraya girmedi** facebook'un gelistirdigi, benim servisimdeki veriyi istedigim gibi alayim, istedigim gibi

REST soa'ya alternatif cikti, xml ile ugrasmayalim, CRUD icin biz http metodlarini kullanalim. icerik gondermek icin de json kullanalim.
xml sayesinde html'imiz var, sayesinde bir cok is yapiyoruz.

json validasyonu, aradigim sey, validasyonunu nasil yaparim: json schema geldi.
tipki xml'de oldugu gibi standardize edilmeye calisildi.

json'a bir alternatif olarak YAML geldi.
artik kendisini bir json subset'i olarak tanimliyor, onunla ayni kurallari
uygulamaya calisiyor

mimari tasarlarken : ucgende kalite, zemin zaman ve maliyet var.
burada hep 2 seyi optimize edebilirsin, ucuncu acikta kalir.
(acaba bunun 3 cisim problemiyle alakasi var mi?)

makinanin scsi kartinin pili bitmis.
kirim analizi yapiliyor (bizim deadline'i gecince yapilan toplanti)
