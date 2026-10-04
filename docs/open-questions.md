# Açık Sorular

_Son güncelleme: 2026-10-04_

## Karara bağlananlar

- **Geliştirme:** Birlikte yazılacak. Geliştirici Java + Vue/Vuetify tecrübeli, mobilde tecrübesi az.
- **Ürün yaklaşımı:** Sıfırdan gerçek ürün gibi kurgulanacak (çok kiracılı, denetlenebilir, testli).
- **Kapsam:** Çekirdek kavram "alım-satım projesi" (al → harca → sat → böl → kapat). Sürekli işletme, stok, bordro, resmi muhasebe kapsam dışı.
- **Pilot dikeyler:** Gayrimenkul + araç. Tek sunucu platformu; dikeyler şablon olarak eklenir; ayrı uygulama yalnızca UX ciddi ayrışırsa.
- **Hedef:** Öncelik kurumsal olmayanlar (bireysel ortaklıklar, küçük tüccar). Kurumsal için model kapıyı açık tutar.
- **Çekirdek model:** Kullanıcı ≠ Ortak; tenant olarak çalışma alanı; değiştirilemez hareket defteri (bakiyeler türetilir); kayıt anında para birimi + kur.
- **Backend:** Spring Boot + PostgreSQL (kesin).
- **Web:** Vue 3 + Vuetify. Tam ürün + yönetim paneli; mobildeki tüm işlemler webde de var (web ⊇ mobil).
- **Ortaklık:** Hem projeden projeye hem de proje sürerken değişebilir. Devir (pazarlıklı fiyat) ve sermaye iadesiyle çıkış ikisi de desteklenecek.
- **Kâr:** Varsayılan olarak sermaye katkısına bağlı. Emek payı: kârda yüzde / sabit ücret / yüksek hisse; zararda sabit ücret.
- **Masraf girişi:** Bireysel ortaklıklarda tüm ortaklar masraf girebilir.
- **Ödeme kaynağı:** Çoğunlukla ortakların kendi cebinden; model ortak kasayı da destekler.
- **Fazla ödeme:** Kasada para varsa kasadan, yoksa ortaklar elden öder; ortaklar isterse masraf hisseye dahil edilir ve oranlar yeniden hesaplanır.
- **Onay mekanizması:** Seviye 0 kayıt (hemen işlenir + itiraz, tutar eşiği yok), Seviye 1 para el değiştirme (alan onaylar), Seviye 2 anlaşma değişikliği (onaysız işlenmez).
- **Saha çalışanı:** Kâr ve ortak bakiyelerini görmez. Saha rolü MVP'de yok (model kapıyı açık tutar).
- **Ortak olunmayan proje görünürlüğü:** Opsiyonel.
- **Offline:** Nadir; sahada yalnızca kayıt, onay ve ortaklık işlemleri internet bekler.
- **Giriş:** MVP'de e-posta + Google + Apple; davet linki. SMS ile giriş ileride — kod buna hazır yazılacak.
- **Sunucu:** Pilotta bir arkadaştan alınacak sunucu.
- **Git:** Claude git işlemi yapmaz (init/commit yok); depo ve commit kullanıcıda.
- **Mağaza hesabı:** İlk etapta bireysel geliştirici hesabı.
- **Süren proje:** Var ve uzun sürecek; kesin hesap 2. fazda kalabilir.
- **Taksit / senet:** Taksit neredeyse yok; bir kez senetle satış yapıldı. Vadeli satış + tahsilat 2. faz, MVP peşin.
- **Sunucu detayı:** Yeterli düzeyde; ilk etapta yalnızca 4 kişi test edecek.
- **9. bölüm (MVP fazları):** İtirazsız geçti.
- **Süren projenin geçmişi:** Her ortağın koyduğu toplam kâğıtta belli; masraf detayı kayıp. Projede satış yapıldı, hiç pay dağıtımı yapılmadı.
- **Çalışma adı:** Paydaş (kod adı `paydas`); kullanıcıya görünen ad sonra belirlenecek.
- **8. bölüm (güvenlik ve altyapı):** İtirazsız geçti — Keycloak, tek şema + `workspace_id` + RLS, sağlayıcıdan bağımsız mimari, kişisel verilerin defterden ayrılması, güvenlik temelleri.

## Onay bekleyen öneriler

- **Kasa düzeltmesi:** Ali 200 bin ödediyse kasadan 200 bin almalı (150 bin değil). Kasa boşsa diğer üçü 50'şer bin borçlanır.
- **Geri ödeme akışı:** Masraf kaydından sonra "nasıl ödensin?" → kasadan / ortaklar elden / hisseye dahil et / kapanışa bırak. Tutarları sistem hesaplar.
- **Mobil:** Flutter (offline için SQLite katmanı, aday: drift).
- **Web offline değil (varsayım):** Tam işlevli ama online.
- **İstemci ilkesi:** İş kuralları yalnızca sunucuda; OpenAPI sözleşmesinden Dart + TS istemcileri üretilir.
- **Dağıtım modeli:** Emek payı yüzdesi = kâr oranı ayarı (tek mekanizma). Sabit emek ücreti = ortağa yapılan masraf. Sıra: net sonuç → sermaye iadesi → kâr oranında kâr / sermaye oranında zarar.
- **Ortaklık yapısı değişikliği:** Tek olay tipi; UI'da 4 sihirbaz.
- **Seviye 2 varsayılanı:** Oybirliği; uygulamayı kullanmayan ortak için zorunlu notla "adına onay".
- **Roller ve görünürlük:** Yönetici yönetim yetkisidir, finansal görme yetkisi vermez. Proje görünürlüğü Gizli (varsayılan) / Açık; değiştirmek Seviye 2. Ortak ↔ kullanıcı bağı 0..n.
- **Offline tasarımı:** İstemcide üretilen ID + bekleyen kayıtlar kuyruğu + sıra numarasıyla çekme; çakışma kullanıcıya gösterilir; kur sunucuda doldurulur.
- **SMS'e hazırlık:** Kullanıcı kimliği IdP `sub` ile bağlanır (e-posta değil); profilde baştan telefon alanı (E.164 + doğrulandı bilgisi); davet linki kanaldan bağımsız; bildirimler tek arayüzden (bugün e-posta + push, yarın SMS).
- **Sunucu kurulumu:** Docker Compose (Caddy + Spring Boot + Keycloak + PostgreSQL). Yedekler ve fişler sunucu dışında. Tahmini minimum 4 GB RAM / 2 vCPU / 50 GB SSD.
- **Arkadaş sunucusu sınırı:** Pilot (kendi verileriniz) için kabul edilebilir; dış müşteriden önce kendi kontrolünüzdeki altyapıya taşıma.
- **MVP fazları:** Faz 1 kâğıdı bırakma (mobil + sunucu çekirdeği), Faz 2 gerçek tablo (web v1, kesin hesap, Seviye 2, reel getiri, push), Faz 3 ürün (KVKK, mağaza, SMS...), Kurumsal sonra.
- **MVP başarı ölçütü:** Bir projenin tüm masrafları 4 hafta boyunca yalnızca uygulamaya girildi; ortaklar "kim kime borçlu" cevabını uygulamadan okuyor.
- **Satış ≠ tahsilat:** Modelde ayrı; kâr dağıtımı yalnızca tahsil edilen paradan. MVP arayüzü peşin satış.

- **Açılış sihirbazı (MVP):** Ortaklar + güncel hisseler → her ortağın koyduğu toplam → bilinen büyük kalemler (opsiyonel) → geçmiş satış tahsilatları + açık alacak (senet vb.) → paranın şu an nerede olduğu (kasa / ortakların elinde). Geçmiş masraf toplamını sistem denklemden hesaplar (konan + tahsilat − bilinen kalemler − eldeki nakit); negatif ya da tutarsızsa uyarır. Sonunda tüm ortakların onayıyla (oybirliği) kilitlenir.
- **MVP'ye eklenenler:** Geçmiş tarihli kayıt, başka ortağın ödediği kaydı girme, açılış sihirbazı + açılış onayı.

## Kritik

Tasarımı değiştirecek açık soru yok. Spec yazıldı: [2026-10-04-paydas-design.md](superpowers/specs/2026-10-04-paydas-design.md) — kullanıcı incelemesi bekleniyor.

## Sonraya kalanlar

- **Şirket kurulumu:** Ücretli müşteri ve faturalamadan önce. Apple bireysel → şirket hesabı dönüşümü (uygulamayı taşımak yerine hesabı dönüştürmek; Apple ile giriş kullanıcı taşıma yükünü önler — doğrulanmalı).
- **Google Play bireysel hesap:** Üretime çıkmadan önce 12 test kullanıcısıyla 14 günlük kapalı test şartı (son bilinen kural — doğrulanmalı).
- **Dosya depolama seçimi:** Harici S3 uyumlu servis mi, sunucuda S3 uyumlu servis mi — plan aşamasında.
- **Kurumsal faz:** Saha rolü, hiyerarşik masraf onayı, çalışan avansı ve kapanışı, şube/departman, özel roller, SSO, muhasebeye dışa aktarım, belirli üyelere proje açma.
- **Link ile onay:** Hesabı olmayan ortağa SMS/e-posta ile tek seferlik onay linki.
- **Yerel veritabanı şifreleme + biyometrik kilit:** Halka açılıştan önce.
- **Fiş fotoğrafları ve silme hakkı:** Hukuki soru.
- **Apple giriş kuralı:** Üçüncü taraf giriş sunuluyorsa Apple ile giriş gerekebilir (doğrulanmalı).
- **Çıkışta değer:** Nominal sermaye mi, güncel değer mi? (Ticari karar.)
- **Mükerrer kayıt:** Sunucu senkronda "olası mükerrer" işaretler.
- **Enflasyon ve ortak katkısı:** Farklı zamanlarda konan eşit tutarlar hesaplaşmada eşit mi?
- **Kur ve TÜFE veri kaynağı:** Aday TCMB EVDS (doğrulanmalı).
- **Ürün hedef kitlesi:** İlk pazarlama hangi segmente?
- **Gelir modeli:** Şimdi değil.
- **Ürün adı:** Sonra.
