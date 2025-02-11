# Mikroservisler ve Devops 2

-[Mikroservisler ve Devops 2](#mikroservisler-ve-devops-2)
   -[Sistem Mimarisinin Yazilim Uzerine Etkisi](#sistem-mimarisinin-yazilim-uzerine-etkisi)

## Sistem Mimarisinin Yazilim Uzerine Etkisi

yazilimin ozelligi (uygulamanin), devopscu daha sonradan
mudahale ederek bunlar isaglayamaz.

- yuksek erisilebilirlik: cluster olusturmamizin 2 nedeni var:
  - HA: 7/24 fakat bunun bedeli var:
    - MW: middleware uygulamalarini coklamak gorece kolaydir. Active/Active
    - DB: dikine buyumek zorundadirlar: Active/Passive
      (otomatik olmasi icin farkli servisler var ama maliyet
      tabi ki, alternatifi manuel yapiyor.)
      pg icin patroni ile replikasyonlari garanti altina
      aliyor. patroni pgpool'dan daha iyi (best practice db
      makinalari uzerinde calistirmak)
      LB: de 2 tane olmasi lazim
- felaket kurtarma merkezi (disaster recovery): burada baska
  bir sehirde olmak gerektigi, kurallar biraz daha farkli
  otomatik olmazlar. manuel yapilir. Buradaki sistem canliyla
  birebir ayni olmak zorunda degil, sadece felaket aninda
  nelerin canlida tutulacagini onceden uygulama tarafinda
  belirleyip buna gore hareket edilir.

klasik veritabani disk uzerinde disk io yapmak isterler,
araya k8s'in controller'lari giriyor volume management
normalde ihtiyac duymadigin bir sey de; surekli backup
alman gerekiyor.

blok disk alanlariyla daha duzgun
disk seviyesinde replike edilmesinde garanti veriyor.
veri tabanlari makinanin diskinde durmazlar, nas'ta dururlar

aktif/aktif sistemde tum istekleri karsilar;
uygulamalari yaymak bu anlamda daha kolay
uygulamanin ayni anda 2 kopyasinin olmasinin getirdigi kurallari

lb round robin'i dagitiyor
browser app1'de session'u acti,sonra digerine dagitti,session
orda yok, yeniden session acti form doldurdu data'yi
session'da tutuyorsaniz o session'larin replike olmasi lazim
surekli veri paylasmanin bir yolu olmasi lazim: cache
yazilimlarinin distributed olmasi

http cache
session cache'lerini yonetimiyle ilgili bir yontem
verir: kafadan session replikasyonu yapma becerisiyle gelir.
node express de sagliyor.ana bu isi yapacak duzgun bir cache
sistemi soyle 2 app arasina oturacak hz'ye ihtiyac var
app cache:  uygulama icinde bazi seyleri hesapliyorum, sonra
tekrar yapmamak icin ram'de bir yerde tutuyorum. bir list
collection veya mapping yontemiyle data'yi koyuyor, x node'un
ram'ine koyuyor, diger node'un bundan haberi yok; mutlakada
duzgun bir distributecache yapisini tasarlamasi gerekiyor
bir api'yi kullandigin kutuphaneler tanimlar: spring cache
diye bir api var, o arkadaki confi'tan hangi implementasyonu
aliyor: in-memory mi hazelcast mi?
araclar hep var sizin implement etmeniz gerekmiyor.
db cache: jpa/hibernate second devel cache

phpciler memcache
javacilar infinity, hazelcast
webciler redis kullaniyor

eski sistemler icin pansuman tedbir network icinde lb'de
sticky session**

lb her zaman session yonetimi icin mutlaka bir coookie
gonderiyo, sonrasinda bu cookie'yi dolastir: sid random no.
tarayiciya gonderdi, sonrasinda lb bir liste tutuyor ve hep
ayni sunucuya gonderiyor.
ip bloklariyla da tutabilirsin x bloktan bu sunucuya, y
bloktan digerine

mesela; sabah basladi 100 kisinin 50sini a, 50sini b'ye verdi
1 saat sonra timeout'lar basladi, birinde 10, digerinde 40
kisi kaldi, yuk dengesiz dagildi.
mesela x sunucusu down olursa
sticy session yapsan bile veri tabani ve uygulama cache'iyle
ilgili sorun var
api cache'teki data yeniden hesaplanabilir bir sey ise bunu
kaybedebilirim. illa replike olmasi gerekmiyor, diger yontem
eviction (x'te sorun olursa y'yi de sil.)
memory grid (veritabani gibi kullaniyorsam, datanin
kaybolmamasi gerek)

gelistiricinin bunu ongormesi gerekiyor, cunku devopscu
bunlarin konfigurasyonlarini yapacak.

**http cache'i sor** : syn cookie kullanmiyoruz, o
networkculerin isi

her makinadan 2tane: sizin kayip siniriniz? 1 tane kaybolursa
ben onu otomatikmtan cover ederim, arkadaslar 2.yi kaldirir,
yok kaldiramazlarsa
db maliyetli bir lisans her seyi kaybedresem yedekten geri
donecegim diyorsun ama bu da saatlerce surebilir.

ikitelli'yi su basti
vodafone veri merkezinin tavani coktu

**DNS gecisi de otomatik olabiliyor. buna bak**

daha modern sistemlere ihtiyac var:
mongodb kumesi kurdugumda:

- primary - leader election: en az 2 replika, quorum: oylama
  yapiliyor. bazen birisi disk i/o yavas, buna gore configure
  ediyorsun. bunlar hep aralarinda konusuyor protokolun adi da
  gossip. artik aglarimiz da sanal ya; arada kopmalar olursa?
  3servis ayaktayken1 tanesi yalniz kalirsa, 2sinden birisini
  primary seciyor. mongo'da leader secemiyorsa veri kaybetme
  riski oldugundan  kendisini pasife ceker
  automatic failover: datayi replike edip
  disk'leri de raid yaptik 6 tane

  ne yapiyoruz: sharding: yataya buume, 300gb'lik diskler alip
  bunlardan yana dogru 3x300gb

mongoose ( bir cesit lb) siz req'leri ona
gonderiyrosunuz,dagitma isini o yapiyor
1 tb'in 300gb'ini a1, a2, a3'e dagitiyor, lazim oldugunda da
bunlari topluyor. surekli de bunlar arasinda bir iletisim var.
(abi bendeki data col oldu sana gonderiyorum,) sharding'in de
replikasini aliyorsun ki bir node'u duserse data kaybi olmasin

boyle bir kume kurmak istediginde en az 10 makina kurmak
gerekiyor.
1tb'in fiyatiyla 300gb'liklar arasinda cok ciddi fiyat farki var:
relational yataya buyuyemez bu net

mongo verinin transactional yapisini sizin saga gibi
yontemlerle sizin garanti altina almaniz gerekir, benzer durum
cache sistemleri icin de gecerli.

hz 2 tane - digerleri min 3 tane oluyor

veriyi 3e bolelim datanin replike olmasi
A     B     C
c1    a1    b1

dikkat edersen 1 tanesi kaybolsa da tum data garanti altinda
cache sistemlerinin yataya buyume ihtiyaci
chip'in uzerindeki ram sayisi fiziki sinirli: 64 islemcinin
siniri var, onun icin yataya buyume var
iostore'daki sorgulara hizli cevap veriyormus

bunlarin kendi ic network'leri olusturulur, udp konusur,
multicast, unicast, fiziki network kartlarini bile ayirirlar

veriyi diske yazmak istiyorsaniz redis degil couchbase oneririm

sharding konusuna biraz bak

multicast, unicast yapamiyorsam ben tcp'den konusurum
diyebilirler bazilari, bazilari tcp'den konusamayabilir.
farkli port'lardan konusurlar o port'larin acilmasi gerekiyor
(kendi ic konusmalarini yapacaklari port'lar)
farkli cluster'daki makinalar
bazen hz'ler cok yavasliyor: arastirdiklarinda 1 gelistirici yerelinde servis
aciyor, datayi lokaline replike ediyor
sirket icindeki network'le prod network'u ayirmak gerekiyor
k8s kendi kapali network'uyle geliyor.

- Cluster state durumu: izlenmesi gerekiyor, kume saglikli mi?
  bunu izlemezseniz node'larin durumundan haberiniz olmaz.
  cluster'larda izlenecek sey: uygulamanin duzgun calismasi
  kumenizin saglikli oldugu anlamina gelmez.
- Distributed cache kullanimi: buna degindik. (tek request'le
  claisan bir collection'a yazdiniz okudunuz, lokal makinada
  sorun cikarmaz ama sunucuda ayni sn.'de 10 thread seklinde
  calisiyor, thread safe kodlanmayan programlardaki hatalari
  tespit etmek cok zor olabilir. hatayi tespit etmek icin
  breakpoint koyuyorsun, sorun cikmiyor. thread safe uygulama
  gelistirmek sikintilidir, duzgun bir distributed cache
  kutuphanesi kullanmak gerekir, butun her sey implement
  edilmistir. multithreaded programlama yaziyorsan bu fonk.
  thread safe'tir, bu degildir gibi uyarilar dokumana
  konulmali, bir BE uygulama mutlaka multi-thread calisir,
  fe'de de artik bunlar yapiliyor) daha basta duzgun yazilmasi
  lazim. sirf bunun icin bile distributed cache
  kullanilabilir.
  ornek: 1 fonkisyonun var, collection'a bakiyor, yoksa
  ekliyorum, 2 req geldi, x ve y, ayni anda collection'a
  bakti, yok, ekledi, bu kod blogu thread safety ise bekle,
  ben yazayim sonra sen yazarsin; cache sistemleri butun
  bunlari yaparlar. onlari kullanin (global cluster log gibi bu
  mekanizmaya ihtiyac var) bunlari yazmaya calismayin, duzgun
  implement etmek zor, hazir yazilmisi var.
  (sizin yazdiginiz sistemlerin de leader election yapmasi
  gerekir, mesela yukaridaki global cluster log sisteminde
  sadece leader yapsin dersen rahatlarsin, bunun icin de senin
  kullandigin sistemdeki lider secimini cozmen gerekiyor,
  kafka zookeeper'i bunun icin kullaniyor. bastan yazma,
  yazilmislari kullan. urunu sadece k8s'te kullaniyorsan
  k8s'in lider secimi icin olan api'lerini kullanirsin,
  boylece if blogunda bir lidersen bundan sonrasini yap, else
  yapma secenegini koyabilirsin)
  cogu programlama icin bunu yapan bir kutuphane vardir.
  zookeeper'la mi, k8s'le mi yapmak istiyorsun, sec
  kutuphaneyi kullan. lider seciimi icin kullanilan araclar,
  service discovery bonusu da veriyor, cogu ayni zamanda "ben
  boyle bir servis ariyorum nerde o? Bir  esit servis)

  otomatik testler icin test container kutuphanesi var bu
  docker kullaniyor.

- veritabani degisiklik yonetimi: uygulamanin x surumunden y
surumune gecilirken veri tabanina su tablodaki y kolonuna
eklenecek vs gibi degisikliklerin surumle beraber yonetilmesi
gerekir, bunu surum yonetimi sistemine dahil etmiyorsaniz
adam gitti pod acti veritabanina ekleme yapti. 2 tane ozgur
yazilim urun var: flyway - liquibase her ikisi de bu konu icin
gelistirilmis. ruby'deki migration rutinleri.
tek tek dosyalari sql cmleleri yaziyorsun, naming
conventionlari flyway bunlari tektek uyguluyor,
uyguladiklarini bir tabloda  tutuyor, bunlari uyguladim diye
flyway uygularken dosya ismi uzerinden siraliyor, uygun bir
naming convention tutmuyorsan veya ekipte birileri
rollback yetenekleri yok (liquibase'de var ama yazman lazim)
liquibase'de benim kodum oracle pl/sql'ine hem de pgsql'ine
uygun dataset'i uretebilir.
liquibase'de hangi dosyanin ne zaman uygulanacagini kendin
sirali verebiliyorsun (birden fazla gelistiricinin)
liquibase'i java kutuphanesi var, uygulamayi ayaga kaldirirken
once bunlari uyguluyor. repo'da kodla beraber yasamasi
gerekiyor; release note'ta bunlari da belirtmen gerekiyor.
veri tabaninda hangi surumde ne degistigini takip ediyorum.
surum ciktiginda bir yeni surum cikiyorsun: duzeltme surumu
geri almiyorsun. surum politikasini nasil yazdiginizla
iliskili. geri donmektense sadece o hatayi duzeltip yeni surum
cikmayi tercih ederim.
sql dosyalarini manuel yurutuyorlar dba'ler tek tek bakmak istiyor
ne zamna yedek alinmali, oncesinde bir yedek alinabilecek bir
sitemle alinamacay sistem arasinda fark var. politikanin
belirlenmesi lazim.
ci/cd kadar onemli
bu patch'i uygularsan bir patch cikartiyor sana, veri tabani
degisiklik yonetimi: ddl'den bahsediyorum, kodunuzdaki sinif
modeliyle veri tabaninin diff'ini aliyor.
12tb. datayi bir yerden bir yere networkte transf etmek uzun
zaman
failover senaryosu: bunlari calistirdim
varolan tablonun data tipini degistirim, artik rollback
yazamiyorum ya da drop ettim, artik donemiyorum

deployment'la release cikartmak ayri seyler

- servicebus/ messagequeue / distrubuted event bus/ event sourcing

yuk dengeleme soz konusu oldugunda once metriklere ihtiyacim
var. performans ihtiyaci nedir, onden alalim bunu da. ayni
anda gelebilecek max. request sayisi: sn.'de 10 mu 3000 mi?
response suresi: 1ms mi 1sn mi dakikalar surebilir mi?
bu bilgiyi zor da olsa alalim. nereden alinabilir?
peek time'lari bulmamiz lazim ki o zamanki yuku kaldirmak
lazim. bizim buna hazirlikli olmamiz lazim
benim burada yuku kaldirma derdim oldugundan sticky session
yapabilirim manuel. login olurken yuku dengeli dagitirim.
burada HA derdim yok
yukun nereye geldigi? yuk testi yaparak analiz etmek gerek.
monolith'ler x module de gelse biz burada coklarken tamamini
coklamak durumundayiz, mikroservislerde 1 servisi coklamam
yeterli (manuel)

- lb darbeyi yedi, test ettik kaldiracak hale getirdik.
- uygulama sunuculari aldi, kaldiracak hale getirdik.
- db'ler aldi, patladik

apache jmeter http client olarak calisiyor
paralel calisitabliiyorsun

benim yuku karsilayabilecek duzeyde donanima ihtiyacim var

mq sunuculari http degil tcp protokoluyle konusabilmek icin,
ingress buna izin vermiyor, o zaman worker node'un ip'sine
routing yazilacak, ben req'i ona atacagim, o da pod'a
yonlendirecek.

dns over http: dns'in portlarini kullanmiyorsun http uzerinden
gidiyor dns sorgusu yapiyor

once izle, sonra iyilestirme yaptim tekrar izleyelim
yuk testlerini kafaniza gore yapamazsiniz, planlamaniz lazim,
canli sistemleri calisamaz hale getireceksiniz, butun ilgili
taraflarin bu testin bir parcasi olmasi lazim. cunku burada
(sunu bir degistireyim hizlica bakalim), mesela networkcu o
arada bir seyi degistiriyor, veri tabanindaki arkadas
index'lemeyi duzeltilebilir, bu sebepten orada olmalilar.

testi sistem patlayana kadar yuk bindirmeye devam ediyorsun,
onun sinirlarini da ogrenemk icin. ileride bunu ne zaman
buyultebilecegimizi ogrenelim: 6 ay sonraki beklentiniz nedir?
cunku yazilimda surekli yeni isterlerle kullanici kitlesi
arttirilir. x yeni modulu gelince 5 kat artacak. bu karara
gore donanimi da planlamamiz lazim

genelde fiziki teknik sinirlara gelmeden once ekonomik
sinirlara geliyoruz.

kurumun verecegi karar: exadata'ya 1m $ vermek mi, musteri
hedef kitlemi degistiriyorum, yoksa yazilimimi mimari olarak
degistireyim:

bit vise operator'lerle 30 byte'lik datayi 1 byte'a
sikistirmayi bilirim

cogu programin icerisinde memory leak
tarayicida da garbage collector var: ram kullanma sinirina
gelir, ram'i temizler, sonra tekrar kullanir bu egri
monitoring aracinda grafigi surekli tepede gezinirse memory
leadk vardir. sorun nedir?
cok sik yapilan hata: jvm icin dogru yontem degil: 1 tomcat'e
1 wildfly'a 5-10 tane uygulama aciyor, jvm'in kaynak dagitiyor
size gelen metrikler jvm basina gelecek ve hangi uygulama
ne kadar tukettigini goremezsin

uygulamanin class'larinin memory'de durmasi gerekiyor,
undeploy ettiklerimiz de memory'de durmaya devam ediyor.
meta space size'imiza bakalim.
bunlarin tespiti icin monitoring yapilmasi gerekiyor

yazilimin okunakli ve surdurulebilir olmasi, onun 1 gb fazla
ram tuketmesinden daha onemli, cunku o 1 gb'in maliyeti
apex-opex ayrimi genelde bakim maliyeti satinalma maliyetinden
kat kat fazladir.

k8s'in bonusu
k8s'te service tanimla: buraya gelen servisilerin yukunu
izleyiver, x sinirini gecerse yeni bir node kaldir.

farkli farkli sistemler arkli peek zamanlarinda yuk aliyorlar,
birisi saat 5'te, digeri sabah saat 3'te, kaynak paylastirarak
auto scaling'le bosta yatmasini engelleyebiliriz. butun
bunlari manuel yapmak

hibrid cloud cozumu k8s'in otomatik olmaz, mesela rancher,
openshift bunlari yapabiliyor: otomatik req karsisinda size
kaynak verecek bir servisin olmasi lazim. amazon'la
anlasiliyor, donanim sinirina gelince oraya cikiyorlar, sonra
donanim temin edip tekrar cikiyorlar.

os calismaya devam edebilmek icin OOMKiller ile en cok ram
kullanan uygulamayi oldurur. genelde de jvm kullanir, mesela
tomcat'in uzerine apache koydun, statikleri oradan almak icin,
apache fork yapisinda calistigi icin kucuk boyutlu binlerce
child olusturur, ram doldugunda jvm'i oldurur.

production'da swap kullanilmamali. ama biz swap'leri acik
tutariz. bi sorun cikti iceri girip bi sey yapmam lazim, en
azindan

spring batch kutuphanesi yerine apache camel var, bunu
kullanabilirsin, dosyada filan da tutmaz.

**veri tabani degisiklik yonetimi politikasini arastiramaya devam et**

ic SLA: surum politikasi belgesi, kurumdaki ust yoneticilerden
onaylanmasi gerekir.

per service per db: oracle'da farkli bir schema, postgre'de
farkli bir instance olabilir. ayni schema'daki tablolara 2
farkli servisten gidemez.

cd sureci icindeki dml'e en basta dba'in bakmasi lazim.

ddl tablo desenleri de yaziliminizin bir parcasi: liquibase
relational'larla calisiyor, mongo icin de bir destekleri var,
mongo yari schemal diye gecer, collection'a bi dokuman
koydugunuzda semasinin suna uyup uymamasini denetleyebilirsin,
birbirinden farkli dokuman semalarini biraraya toplayabilirsin
ya da bunlari yapmayabilirsin,
key-value: dogal olarak semasizlar.
hiyerarsik verittabanlari: ldap, tablo deseni gibi olmuyor ama
onden bir sema tanimlaman gerekir.
protobuff, avro protokolleri icin de bir schema tanimliyorsun:
birinci field'in adi bu, data tipi bu: bunun icin de bir degisiklik
yonetimine ihtiyac var

buradaki hile, hicbir attirubute'u silmezsen eskiden onu
kullanan kullanmaya devam ediyor ama bu bir hile, bir sure
sonra sisiyor

rest yazarken degisiklik yonetimi icin kullanilan yontemler
var: url'in uzeirnde surum no /v3 gibi gidip gelen datanin
icinde olabilir, http header'ina koyulabilir, hangisi uygunsa
kullan.
