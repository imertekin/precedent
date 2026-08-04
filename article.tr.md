# Planlama Ajanımı Projenin Tarihçisi Yaptım — Sonra Kendi Kararlarının Makbuzunu da Tutmasını Öğrettim

*Bu bir AI hafıza sistemi değil. Bir kararın, bir kere alındıktan sonra gerçekten ihtiyaç duyulduğu yere ulaşmasıyla ilgili — bu yılın moda hafıza mimarisinden çok daha uzun ömürlü bir problem.*

Bir projenin üçüncü haftasında, kendi AI asistanıma belirli bir veri modelini neden seçtiğimizi sordum. Bilmiyordu. Ben de bilmiyordum. Bunu çoktan kaybolmuş bir sohbet oturumunda konuşup karara bağlamış, devam etmiş, ne insan ne de ajan bir daha bakmayı düşüneceği hiçbir yere yazmamıştık.

Kaybedilecek küçük bir şey bu. Ama tam olarak, birkaç ay içinde birikince bir projeyi "kimse hiçbir şeyin neden böyle olduğunu açıklayamıyor" haline getiren türden küçük bir şey. Bir yıldan fazla süredir bir kod tabanında çalışmış herkes bunu hissetmiştir. Yeni olan şu: artık bu kararların bir kısmını bir AI ajanı da veriyor, kaybolup gidecek bir sohbet penceresinde, gece yarısı, kimseye sormadan.

Ben de küçük, gösterişsiz bir çözüm kurdum. Adı **Precedent** (emsal), ve özü tek cümleye sığıyor: bir AI ajanı, arkasındaki kararı bir sonraki oturumun — insan ya da ajan fark etmez — bulabileceği bir yere yazmadan çalıştırılabilir bir iş devralamaz.

Benimle konuşan, kapsamı netleştiren, brief'i yazan ajan artık sadece plan yapmıyor. Projenin tarihçisi olarak görev yapıyor.

## İki tür unutma

AI ajanlarıyla oturumlar boyunca çalışırken bağlamı iki farklı şekilde kaybediyorsunuz.

Birincisi **oturum kaybı** — yeni bir sohbet açıp projeyi sıfırdan anlatmak zorunda kalmak. Bu, çokça ilgi görüyor zaten. "Hafıza" özelliklerinin ve uzun context'li modellerin bu kadar konuşulmasının sebebi bu.

İkincisi daha sessiz ve bence daha pahalı: **karar kaybı**. Ne insan ne ajan, bir şeyin *neden* belirli bir şekilde yapıldığını hatırlamıyor. Kararın alındığı gerçeği değil — arkasındaki gerekçe, elenen alternatif, o kararı zorunlu kılan kısıt. Altı ay sonra bir şeyi güvenle değiştirip değiştiremeyeceğinizi belirleyen tam olarak bu, ve ilk buharlaşan da tam olarak bu.

Buna karşı iki geniş yaklaşım var. Biri otomatik: her şeyi embedding'leyip getiren bir hafıza sistemi çalıştırmak, ajan "benzer geçmiş durumları" istediğinde çekebilsin diye. Diğeri manuel: bir insanın gerçekten yazdığı, gerçekten güvendiği, az sayıda elle küratörlüğü yapılmış doküman tutmak. Ben ikincisini seçtim, ve neden seçtiğim konusunda dürüst olmakta fayda var — genel olarak daha iyi olduğu için değil, benim ölçeğime uyduğu için. Az sayıda proje, yavaş ve bilerek alınan kararlar, işin içinde bir ya da iki kişi. Yüksek frekanslı, düşük riskli kararlarla dolu onlarca paralel deney yürütüyorsanız, otomatik bir sistem muhtemelen bunu geçer. Bu, evrensel bir cevap değil, belirli bir tür kurulum için bir araç.

Ama burada bir şeyi işaret etmek istiyorum, çünkü gerisini nasıl okuyacağını değiştiriyor: bunun gerçekte bir hafıza problemi olduğunu düşünmüyorum, ve öyle çerçevelemek onu küçültüyor. Hafıza, retrieval ile ilgili — ne olduğunu geri çekmekle. Benim sürekli çarptığım şey bir retrieval hatası değildi, bir *yönlendirme* hatasıydı: bir karar alınıyordu, sonra bir daha hiç ihtiyaç duyulduğu yere ya da ana güvenilir şekilde ulaşmıyordu. Modeller değişmeye devam edecek. Context window'lar büyümeye devam edecek. Beş yıl sonra "ajan hafızası" dediğimiz şey bugünün embedding-ve-vektör-deposu yaklaşımına hiç benzemeyecek. Bunların hiçbiri asıl soruya dokunmuyor — bu kararı neden aldık — ki bu soru on yıl sonra da bugün olduğu kadar cevaba ihtiyaç duyacak. Altında yatan araçtan bağımsız olarak, üzerine bir alışkanlık inşa etmeye değer olan kısım bu.

## Gerçekte ne yapıyor

Her proje küçük bir dosya setiyle başlıyor:

```
<proje>/
  AGENTS.md                ← proje briefi, 200 satırın altında
  docs/
    PRD.md                  ← ne inşa ediyoruz
    TECH_SPEC.md             ← nasıl
    DECISIONS.md             ← neden (karar günlüğü)
    DECISIONS_DRAFT.md       ← işlenmemiş notlar için inbox
    BACKLOG.md               ← ne zaman, hangi sırayla
```

Bunların hiçbiri tek başına yeni değil. `DECISIONS.md`, Michael Nygard'ın 2011'de yazıya döktüğü ve o zamandan beri ciddi mühendislik kuruluşlarında sessizce standart haline gelmiş bir Architecture Decision Record formatı. `AGENTS.md` artık gerçek bir açık standart — OpenAI, Google, Cursor ve Factory 2025'te resmileştirdi, sonra Linux Foundation'a devredildi. PRD-sonra-spec-sonra-backlog sıralaması, GitHub'ın kendi Spec-Kit'inin AI-destekli geliştirme için önerdiğine yakın. Bu parçaların hiçbirini ben icat etmedim. Benim kurduğum şey, bunların birbirine bağlanma biçimi.

Her `DECISIONS.md` kaydı da klasik dört alandan fazlasını taşıyor. Düz bir evet/hayır yerine gerçek bir yaşam döngüsünden geçen bir statü — proposed, accepted, implemented, validated, ya da rejected, deprecated, superseded. Ve bir importance (önem) derecesi, öyle ki otomatik artan ID yerine UUID kullanma kararı, tüm backend'i event-driven'a çeviren bir kararla aynı görsel ağırlıkta günlükte durmasın. Bu ayrım, kırk kayıt arasında hangi on tanesinin gerçekten önemli olduğunu bulmaya çalışana kadar kozmetik gibi görünür.

Bu bağlantının iki parçası var, ve ikincisi asıl önemsediğim parça.

```
Planlama Ajanı → ADR yazar → Kod Ajanı
      ↑                              │
      └──────── Draft (inbox) ───────┘
```

**Birinci parça kapı.** Benimle konuşan, kapsamı netleştiren, brief'i yazan planlama ajanı, ilgili karar zaten `docs/`'ta yaşamadan kod ajanı için çalıştırılabilir bir prompt üretmiyor. "Yazmalı" değil. Yazmadan işi devredemiyor. Bu, birinin hatırlaması gereken bir kural değil, mimari.

**İkinci parça, benim de ilk seferinde yanlış yaptığım kısım.** Tek yönlü bir kapı yeterli değil. Kod, plana itaatkâr kalamaz — kalamaz, çünkü plan her zaman eksiktir. Bir kütüphane beklendiği gibi davranmaz, kimsenin görmediği bir kısıt ortaya çıkar, ve özelliği uygulayan ajan küçük, mantıklı bir karar verir ki bu karar hiçbir zaman "tek doğruluk kaynağı" olması gereken dokümana geri dönmez. Altı ay sonra o doküman yanlıştır, ve daha kötüsü, kimse yanlış olduğunu bilmez.

Bu yüzden tam olarak bir amaca hizmet eden ikinci bir dosya var, `DECISIONS_DRAFT.md`: bu küçük, o anlık kararları kaybolmadan önce yakalamak. Oraya düşen her şey hayatta kalmıyor — bazı notlar okunup açıkça reddediliyor, not kaybolmak yerine gerekçesi tutuluyor. Pratikte nasıl göründüğü şöyle.

## Adım adım bir örnek

**1. Oturum.** Planlama ajanından yeni bir proje kurmasını istiyorum — bir şeyi takip eden küçük bir dahili araç. Kapsam üzerine konuşuyoruz, bir PRD yazıyor, ve ilk gerçek kararı kaydediyoruz: hız sınırlaması olan harici bir API'yi veri kaynağı olarak kullanacağız, çünkü hacim bunun önemli olmayacağı kadar küçük. Bu, gerekçesiyle birlikte DECISIONS.md'ye ADR-001 olarak giriyor.

**Aynı hafta içinde.** Kod ajanı entegrasyonu uygularken, hız sınırının anahtar başına değil IP başına uygulandığını keşfediyor — kimsenin kontrol etmediği bir şey. Spec'te olmayan küçük bir cache katmanıyla bunu aşıyor. Bunu sessizce yapıp devam etmek yerine, `DECISIONS_DRAFT.md`'ye tek satır düşüyor: *"2026-07-14 — API IP başına hız sınırlıyor, anahtar başına değil, sınırın altında kalmak için 10 dakikalık bir cache katmanı eklendi."* Format yok, tören yok. Sadece bir not.

**2. Oturum**, birkaç gün sonra, yine planlama ajanında. Projenin briefi — bu projeyle ilgili herhangi bir oturumun başında yüklenen dosya — planlama tarafında kim okuyorsa onun için sabit bir kural taşıyor: önce inbox'a bak. O notu buluyor, gerçekten önemli olduğuna karar veriyor (sadece kozmetik bir detay değil, gelecekteki ölçeklenmeyi kısıtlıyor) ve düzgün formatta ADR-002'ye yükseltiyor. Inbox temizleniyor. Mekanizma konusunda dürüst olmakta fayda var: bu sihir değil, her oturumda yeniden yüklenen bir dosyadaki tek satırlık sabit bir talimat — ve ajanın onu gerçekten okuyup uyması kadar güvenilir.

**Bir ay sonra**, proje MVP fazından bir sonraki faza geçiyor. Bu geçiş yerleşik bir kontrol noktası: ilerlemeden önce, o ana kadar alınan ADR'lere hızlı bir bakış atıp varsayımların hâlâ geçerli olup olmadığını soruyoruz. ADR-002 Validated olarak işaretleniyor — cache katmanı işe yaradı, arkasındaki varsayım doğrulandı. ADR-001'in "hacim önemsiz kalacak kadar küçük" varsayımı işaretleniyor — 4 kat büyümüş, tekrar bakmaya değer — statüsü açık kalıyor, geri kalan her şey ilerliyor.

Bunların hiçbiri benim sormayı hatırlamamı ya da ajanın rastgele bir tarihte bunu kendiliğinden söylemeyi hatırlamasını gerektirmedi. İşi yapı yaptı.

![Precedent döngüsü](./assets/flow-diagram.svg)

## Rol, artefakt değil

Yukarıdakilerin hepsi, sektörün zaten bildiği şeylerin makul ölçüde doğrudan bir uygulaması — karar günlükleri yeni değil. Görmemin uzun sürdüğü şey, artefaktın hiçbir zaman asıl mesele olmadığıydı. Bütün bunu çalıştıran planlama ajanı sadece plan yapmıyor. Projenin tarihçisi olarak görev yapıyor.

Bir tarihçinin işi sadece "yazıya dök" değil. Kararları, sadece bir şeyin karara bağlandığı gerçeğini değil, gerekçesinin de hayatta kalacağı şekilde korumak (düz bir evet/hayır yerine gerçek bir statü yaşam döngüsü olan ADR formatı). Projenin kendisini, her oturumda yeniden anlatmadan, ucuza hatırlamak (her seferinde yeniden yüklenen brief). Temiz, bitmiş bir tarih yerine dağınık, süregelen bir tarihi yönetmek (inbox, ve reddedilen bir notun kaybolmak yerine iz bırakması). Review yapmak — eski bir kararın hâlâ geçerli olup olmadığını, kimsenin uymadığı bir takvime değil kararın kendisine bağlı bir zamanlamayla gerçekten geri dönüp kontrol etmek.

Ve sonra, beni bunu yazmaya asıl iten kısım geliyor, yukarıdakilerin hepsinden daha küçük ve bence daha ilginç olan kısım: **bütün bunu çalıştıran skill, kendi tasarımı hakkında bir karar günlüğü tutuyor.**

Inbox mekanizmasını eklediğimde, sadece talimatları güncellemedim — neden, alternatifin ne olduğunu ve hangi ödünleşimi kabul ettiğimi açıklayan bir ADR yazdım. Sonra bir isimlendirme hatası yakaladığımda (proje briefini `AGENT.md` diye adlandırmışım, gerçek standart olan `AGENTS.md` değil), o düzeltme de kendi kaydını aldı. "Neden karar verdiğini yaz" öğreten araç, bu kuralı kendi üzerine, kendinden istediği aynı disiplinle, özyinelemeli olarak uyguluyor.

Kulağa küçük bir gösteriş gibi geliyor ama gerçek bir sorunu çözüyor: metodolojiler zamanla kayar, ve genelde kimse ne zaman ya da neden kaydığını söyleyemez. Bir yıl sonra bu sistemdeki bir kural garip görünürse, cevap bir Slack thread'inde ya da yarım hatırlanan bir konuşmada gömülü değil. Aynı repoda, diğer her kararla aynı formatta, bir dosyada.

## Şüpheyle yaklaşman gereken yerler

Bunu olduğundan fazla değil, az göstermeyi tercih ederim, o yüzden henüz emin olmadığım yerler şunlar.

Belirli risklerden önce bir şey söylemem lazım: bu sistem bilerek hız yerine açıklanabilirliği optimize ediyor. Eğer senin bağlamın az sürtünmeyle hızlı ilerlemeyi, altı ay sonra bir seçimi açıklayabilmekten daha çok ödüllendiriyorsa, aşağıdakilerin bir kısmı güvenlik değil ek yük gibi okunacak.

**Karar günlükleri terk ediliyor. Bu son derece iyi dokümante edilmiş bir şey, varsayım değil.** Takımlar ADR'leri coşkuyla benimsiyor ve birkaç ay içinde sessizce bırakıyor — o kadar tutarlı bir şekilde ki literatürde bir istisna değil, bilinen bir örüntü. Sebepler arasında bakım yükünün en az bağlama sahip kişiye düşmesi, ya da önemsiz kararların önemli olanların yanına birikip kimsenin günlüğe güvenmemesi var. Tek-yönlü-kapı, kaydın gerçek iş akışının dışında yaşaması sorununu hedefliyor; her kayıttaki importance derecesi, önemsiz kararların önemlileri gölgelemesi sorununu hedefliyor. Bakım yükünün en az bağlama sahip kişiye düşmesi — buna hâlâ iyi bir cevabım yok.

**Inbox zehirlenebilir.** Bir seferlik, önemsiz bir sapma kalıcı bir karara yükseltilirse, ajan yanlış bir dersi öğrenir ve tekrar tekrar öğrenmeye devam eder. Küratörlük (bir insanın ya da planlama ajanının gerçekten neyin tutulmaya değer olduğuna karar vermesi) yardımcı oluyor ama bu bir hafifletme, garanti değil.

**Bunu küçük ölçeğin ötesinde test etmedim.** Bu, az sayıda projesi ve seyrek, bilinçli kararları olan bir kurulum için inşa edildi. Büyük bir ekip ya da yüksek hızlı, çok sayıda küçük kararlı bir ortama koyunca nerede kırılacağını gerçekten bilmiyorum. Tahminim bir yerde kırılacağı ve oradan sonra otomatik bir hafıza sisteminin devralması — ama bu bir tahmin, doğrulanmış bir şey değil.

## Eksik olan parçayı al, gerisini bırak

Açıkça söylemekte fayda var: bu parçaların hiçbiri birbirini gerektirmiyor. Buradan sadece tek yönlü kapıyı alsan bile — bir ajanın, arkasındaki karar kalıcı bir yerde var olmadan çalıştırılabilir bir işi devralmasına izin verme — bu, şu an çalıştırdığın herhangi bir sistemde tek başına işe yarar. Inbox için de aynı, ya da revalidasyonu takvime değil kendi doğal kontrol noktalarına bağlamak için de. Ya da sadece fikrin kendisi — tarihi ikisinin değil tek bir ajanın sahiplenmesi — burada anlatılan spesifik dosya formatlarının hiçbirini almadan bile, elindeki kuruluma taşımaya değer. Bu, bir isim ve manifestoyla toptan benimsenecek bir metodoloji değil. Küçük bir alışkanlık setine daha yakın. Elindekinde eksik olanı al, gerisini bırak.

## Neden yazıya döktüm

Bunların hiçbiri AI ajanlarını daha akıllı yapmıyor. *Kaydı* daha iyi yapıyor, bu daha sıkıcı ama daha kalıcı bir iyileştirme türü. Ajan, bir karar günlüğün olduğu için mimarinizi daha iyi düşünmeyecek. Ama altı ay sonra, biri — insan ya da ajan — sistemin neden garip bir şey yaptığını sorduğunda, "kimse hatırlamıyor" olmayan bir cevap olacak.

Bu düşük bir çıta. Aynı zamanda çoğu projenin, AI-destekli olsun olmasın, sessizce geçemediği bir çıta.

---

*Precedent açık kaynak — yazı, kurabileceğin skill, ve kendi öz-referanslı karar günlüğü GitHub'da.*
