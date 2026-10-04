# Paydaş — Tasarım Belgesi

- **Tarih:** 2026-10-04
- **Durum:** Taslak, kullanıcı incelemesi bekliyor
- **Çalışma adı:** Paydaş (kod adı `paydas`). Kullanıcıya görünen ad sonra belirlenecek.
- **Karar günlüğü ve ertelenen konular:** [docs/open-questions.md](../../open-questions.md)

---

## 1. Problem ve amaç

Dört ortaklı bir grup tarla ya da arsa alıyor, hobi bahçesine dönüştürüp satıyor. Masraflar kâğıtta tutuluyor ve bu üç soruna yol açıyor:

- Kâğıtlar kayboluyor.
- Proje kârlılığı ve "kim ne kadar koydu, kim kime borçlu" sorusunun cevabı görülemiyor.
- Ortaklar aynı anda, senkron kayıt tutamıyor.

Paydaş, **ortaklı veya ortaksız alım-satım projelerinin maliyet, kâr ve ortak hesaplaşmasını** takip eden bir ürün. Önce bu grubun sorununu çözecek. Sonra aynı ihtiyacı olan küçük yatırımcılara, oto alım-satımcılara ve küçük müteahhitlere açılacak.

Temel tespit: masraf her an çıkabilir ve insanın yanında her an telefonu vardır. Bu yüzden mobilde kayıt girmek hızlı olmalı, fiş fotoğrafı eklenebilmeli ve bağlantı koptuğunda kayıt kaybolmamalı.

## 2. Kapsam

### 2.1 Çekirdek kavram: alım-satım projesi

Döngü şu: bir şey al → üzerine harca → (gerekirse parça parça) sat → kârı ortaklarla böl → kapat.

### 2.2 Kapsamda

- Arsa ve tarla (hobi bahçesi dahil), araç. İleride tadilat edilip satılan daire, iş makinesi gibi işler.
- Ortaksız işler: tek ortaklı (%100) proje olarak.
- Öncelik bireysel ortaklıklar ve küçük tüccar. Veri modeli kurumsal müşteriye kapıyı açık tutuyor.

### 2.3 Kapsam dışı

Sürekli işleyen bir işletmenin muhasebesi kapsam dışı: stok, bordro, e-fatura, KDV, resmi muhasebe ve vergi beyanı. İhtiyaç olursa muhasebeciye dışa aktarım verilir.

### 2.4 Pilot sektörler

Gayrimenkul ve araç. Her sektör sunucuda bir **proje tipi şablonu** olarak tanımlanıyor: varsayılan bir kategori listesi ve bir "birim" kavramı (parsel, araç, daire gibi). Ayrı bir uygulama ancak kullanıcı deneyimi ciddi biçimde ayrışırsa gündeme gelir.

## 3. Başarı ölçütü

**Faz 1 (MVP):** Şu an süren projenin açılış durumu tüm ortakların onayıyla uygulamaya girilmiş olacak. Sonraki 4 hafta boyunca projenin bütün masrafları sadece uygulamaya girilecek. Ortaklar "kim kime borçlu" sorusunun cevabını uygulamadan okuyacak.

## 4. Sözlük

| Terim | Tanım |
|---|---|
| Çalışma alanı (workspace) | Kiracı: bir kişi, bir ortak grubu ya da bir şirket. Her kayıt bir çalışma alanına aittir. |
| Kullanıcı (user) | Uygulamaya giriş yapan kişi. |
| Ortak (partner) | Bir projede hissesi olan kişi ya da şirket. Kullanıcı olmak zorunda değildir. |
| Ortak–kullanıcı bağı | Kişi olan bir ortağa en fazla bir kullanıcı bağlanır. Şirket olan bir ortağa birden fazla temsilci kullanıcı bağlanabilir. Finansal bilgileri görme yetkisi bu bağdan gelir. |
| Proje | Bir alım-satım işi. Tipi (`LAND`, `VEHICLE`, `GENERIC`), durumu ve ortaklık yapısı vardır. |
| Birim (unit) | Projenin ayrı satılabilen parçası: parsel, araç, daire. |
| Kayıt (entry) | Bir iş olayı: masraf, gelir, sermaye koyma, geri ödeme gibi. Kayıt değiştirilemez; düzeltme yeni bir revizyon olarak eklenir. |
| Hareket (movement) | Bir kaydın muhasebedeki etkisi: bir hesaptan diğerine giden tutar. Hareketleri sunucu üretir. |
| Sermaye hesabı | Her ortağın her projede bir sermaye hesabı vardır. Ortağın projeye koyduğu her şey bu hesaba artı, projeden aldığı her şey eksi olarak yazılır. |
| Kasa | Projenin ortak nakit hesabı, örneğin ortak bir banka hesabı. |
| Ortakta duran nakit | Bir ortağın elinde tutulan proje parası. Örnek: tahsil edip henüz kasaya koymadığı satış bedeli. |
| Sermaye oranı | Ortağın hissesi. Zarar paylaşımında ve "kim kime borçlu" hesabında kullanılır. |
| Kâr oranı | Kârın ortaklar arasında paylaştırıldığı oran. Varsayılan olarak sermaye oranına eşittir; emek payı kuralıyla değişir. |
| Onay seviyesi | 0: kayıt, 1: para el değiştirme, 2: anlaşma değişikliği (Bölüm 9). |
| Açılış durumu | Uygulamaya geçilmeden önce birikmiş olan durum. Tüm ortakların onayıyla kilitlenir. |

## 5. Mimari

### 5.1 Bileşenler

```
 [Flutter mobil] ─┐                       ┌─ [Keycloak (OIDC)]
                  ├── HTTPS ── [Caddy] ───┤
 [Vue web, Faz 2] ┘                       └─ [Spring Boot API] ── [PostgreSQL]
                                                    │
                                                    └─ [S3 uyumlu depolama, sunucu dışında]
                                                         (fiş fotoğrafları + veritabanı yedekleri)
```

| Bileşen | Teknoloji | Görevi |
|---|---|---|
| Mobil | Flutter; yerel veritabanı için drift (SQLite) | Hızlı kayıt, fiş fotoğrafı, offline kuyruk. İşlemlerin bilerek dar tutulmuş, sahaya yönelik bir alt kümesi. |
| Web (Faz 2) | Vue 3 + Vuetify + TypeScript + Vite | Tam ürün ve yönetim paneli: mobildeki tüm işlemler artı raporlar. Online çalışır. |
| API | Java 25 LTS, Spring Boot 4.x, Maven | Tüm iş kuralları, hesaplamalar ve yetki kontrolü. |
| Veritabanı | PostgreSQL | Tek şema; her tabloda `workspace_id`; satır düzeyi güvenlik (RLS). |
| Kimlik | Keycloak (OIDC) | Giriş, şifre, Google ve Apple ile giriş. İleride SMS ile giriş ve kurumsal tek oturum (SSO). |
| Dosya depolama | S3 uyumlu nesne depolama, sunucunun dışında | Fiş fotoğrafları ve yedekler. |
| Ters vekil (reverse proxy) | Caddy | HTTPS (sertifikayı otomatik alır) ve istekleri yönlendirme. |

Sürüm numaraları plan aşamasında doğrulanıp sabitlenecek.

### 5.2 İlkeler

1. **İş kuralları sadece sunucuda.** İstemciler kayıt alır, kuyruğa koyar ve gösterir. Bakiye, kâr ya da dağıtım hesabını hiçbir zaman istemci yapmaz.
2. **Önce API sözleşmesi (OpenAPI).** Tek kaynak `api/openapi.yaml` dosyası. Dart ve TypeScript istemcileri bu dosyadan otomatik üretilir. Backend'in sözleşmeye uyduğu testlerle doğrulanır.
3. **Tek yazma yolu.** Web ve mobil aynı komut işleyicilerini çağırır. Mobilin toplu gönderimi (sync push) bu işleyicileri toplu çalıştıran bir sarmalayıcıdan ibarettir.
4. **Backend kimlik sunucusuna bağlı değil.** Backend sadece standart OIDC token'larını (JWT) doğrular (OAuth2 resource server). Kullanıcılar kimlik sunucusunun verdiği değişmeyen kimlikle (`sub`) eşleştirilir, e-postayla değil.
5. **Altyapı sağlayıcıya bağlı değil.** Docker, standart PostgreSQL ve S3 API kullanılır. Sunucu değiştirmek ucuz kalır.
6. **Modüler monolit.** Tek bir Spring Boot uygulaması olarak dağıtılır. İçinde özellik bazlı modüller var; modül sınırları testlerle korunur.

### 5.3 Backend modülleri

| Modül | Sorumluluğu |
|---|---|
| `identity` | Token'dan kullanıcıyı bulma, profil (telefon alanı dahil), hesap silme ve anonimleştirme (Faz 3) |
| `workspace` | Çalışma alanı, üyelik, roller, davet linkleri |
| `partner` | Ortaklar, ortak–kullanıcı bağı |
| `project` | Projeler, şablonlar, kategoriler, birimler, görünürlük, ortaklık yapısı versiyonları, açılış |
| `ledger` | Kayıtlar, revizyonlar, hareketler, bakiyeler, "kim kime borçlu" hesabı |
| `approval` | Onay talepleri, oylar, itirazlar |
| `sync` | Toplu gönderim, değişiklik akışı, kapanan erişimler |
| `files` | Yükleme linkleri, dosya bilgileri, indirme linkleri |
| `access` | Tek yetki kontrol noktası (`AccessPolicy`) |
| `notification` | Bildirim arayüzü. MVP'de sadece uygulama içi; Faz 2'de push ve e-posta; ileride SMS. |
| `settlement` (Faz 2) | Kapanış, dağıtım hesabı, ortaklık değişikliği hesapları |
| `fx` (Faz 2) | Kur ve TÜFE verisi |
| `reporting` (Faz 2) | Raporlar |

Dış sistemlerin her biri (Keycloak yönetim API'si, nesne depolama, bildirim kanalları, kur kaynağı) ayrı bir arayüzün (port) arkasında durur.

## 6. Veri modeli

### 6.1 Tablolar (özet)

Kiracıya ait her tabloda `workspace_id` (RLS anahtarı) ve `created_at` bulunur. İstemcinin ürettiği kimlikler UUIDv7'dir.

| Tablo | Önemli alanlar | Not |
|---|---|---|
| `app_user` | `id`, `idp_subject` (benzersiz), `display_name`, `email`, `phone_e164`, `phone_verified`, `status` | Kişisel veri içerir. Belirli bir çalışma alanına ait değildir; kullanıcı sadece kendi kaydına erişir. |
| `workspace` | `id`, `name` | |
| `workspace_member` | `workspace_id`, `user_id`, `role` (`ADMIN`/`MEMBER`), `status` | |
| `invitation` | `token_hash`, `created_by`, `expires_at`, `max_uses`, `used_count`, `partner_id?` | Link belirli bir kanala bağlı değildir. Davet bir ortağa bağlıysa katılan kullanıcı o ortağa bağlanır. |
| `partner` | `id`, `kind` (`PERSON`/`ORGANIZATION`), `display_name`, `status` | Kişisel veri içerir. |
| `partner_user_link` | `partner_id`, `user_id` | Kişi olan ortakta en fazla bir bağ olabilir (veritabanı kısıtı). |
| `project` | `id`, `type`, `name`, `status` (`OPENING`/`ACTIVE`/`CLOSED`), `visibility` (`PRIVATE`/`WORKSPACE`), `base_currency` (`TRY`) | |
| `project_unit` | `id`, `project_id`, `label`, `status` | Faz 1'de modelde var; ekranı opsiyonel. |
| `category` | `id`, `project_type`, `name`, `system`, `archived` | Şablondan gelen kategoriler ve çalışma alanına özel kategoriler. |
| `ownership_version` | `id`, `project_id`, `version_no`, `change_kind`, `effective_on`, `state` | `change_kind`: `INITIAL`/`TRANSFER`/`EXIT`/`ENTRY`/`CAPITALIZE`. Faz 1'de sadece `INITIAL`. |
| `ownership_share` | `version_id`, `partner_id`, `capital_ratio` | Oranların toplamı tam olarak 1 olmalı. |
| `profit_rule` (Faz 2) | `project_id`, `version_no`, `partner_id`, `labor_share` | Emek payı yüzdesi. |
| `account` | `id`, `project_id`, `type`, `partner_id?`, `category_id?` | Bölüm 6.2. Gerektiğinde oluşturulur. |
| `entry` | `id` (istemci üretir), `project_id`, `created_by`, `client_created_at` | Mantıksal kayıt. |
| `entry_revision` | `entry_id`, `revision_no`, `kind`, `occurred_on`, `amount`, `currency`, `fx_rate`, `category_id?`, `source`, `target`, `unit_id?`, `note`, `status` (`ACTIVE`/`VOID`), `approval_state`, `author_id`, `origin` (`NORMAL`/`OPENING`) | Satır hiç değiştirilmez; geçerli olan en yüksek revizyondur. `source` ödeyen taraf, `target` alan taraftır (kayıt türüne göre değişir). |
| `movement` | `revision_id`, `debit_account_id`, `credit_account_id`, `amount` | Sunucu üretir; tutar her zaman sıfırdan büyüktür. |
| `attachment` | `id` (istemci üretir), `entry_id`, `object_key`, `content_type`, `size_bytes`, `state` (`PENDING`/`UPLOADED`) | |
| `approval_request` | `id`, `subject_type`, `subject_id`, `level`, `state` | |
| `approval_vote` | `request_id`, `partner_id`, `decision`, `decided_by`, `on_behalf`, `note` | Başkası adına verilen onayda not zorunlu. |
| `dispute` | `id`, `entry_id`, `raised_by`, `reason`, `state` | |
| `processed_command` | `command_id`, `result` | Toplu gönderimde aynı komutun iki kez işlenmemesi için. |
| `change_log` | `tx` (xid8), `seq`, `project_id?`, `entity_type`, `entity_id` | Senkronizasyon akışı (Bölüm 11.3). |

### 6.2 Hareket defteri (çift taraflı kayıt)

Her projenin hesapları:

| Hesap tipi | Anlamı | Doğal bakiye tarafı |
|---|---|---|
| `CAPITAL` (ortak başına) | Ortağın sermaye hesabı | Alacak |
| `CASH` | Kasa | Borç |
| `HELD` (ortak başına) | Ortağın elinde duran proje parası | Borç |
| `EXPENSE` (kategori başına) | Masraflar | Borç |
| `REVENUE` | Satış gelirleri | Alacak |
| `RECEIVABLE` | Henüz tahsil edilmemiş alacak (senet gibi) | Borç |
| `PAYABLE` (taraf başına) | Bir tarafa olan borç: ortağa sabit emek ücreti (Faz 2), ortak olmayan birine masraf iadesi (kurumsal) | Alacak |
| `OPENING` | Açılışta geçici olarak kullanılan denge hesabı; açılış bittiğinde bakiyesi sıfırdır | — |

Kayıt türleri ve muhasebe etkileri:

| Kayıt türü | Örnek | Borç | Alacak | Seviye | Faz |
|---|---|---|---|---|---|
| `EXPENSE` (ortak öder) | Ali çit için 200 bin öder | `EXPENSE:çit` | `CAPITAL:Ali` | 0 | 1 |
| `EXPENSE` (kasadan) | Kasadan elektrik faturası | `EXPENSE:elektrik` | `CASH` | 0 | 1 |
| `EXPENSE` (ortaktaki nakitten) | Ali elindeki proje parasıyla öder | `EXPENSE:x` | `HELD:Ali` | 0 | 1 |
| `INCOME` (kasaya) | Parsel satışı kasaya girer | `CASH` | `REVENUE` | 0 | 1 |
| `INCOME` (ortak tahsil eder) | Satış parasını Ali alır | `HELD:Ali` | `REVENUE` | 0 | 1 |
| `CONTRIBUTION` | Veli kasaya 100 bin koyar | `CASH` | `CAPITAL:Veli` | 0 | 1 |
| `DEPOSIT` | Ali elindeki proje parasını kasaya koyar | `CASH` | `HELD:Ali` | 0 | 1 |
| `COLLECTION` | Açılıştaki senet tahsil edilir | `CASH` veya `HELD:x` | `RECEIVABLE` | 0 | 1 |
| `REIMBURSEMENT` (kasadan) | Kasadan Ali'ye 200 bin ödenir | `CAPITAL:Ali` | `CASH` | 1 (Ali onaylar) | 1 |
| `REIMBURSEMENT` (ortaktaki nakitten) | Ali elindeki paradan Veli'ye 50 bin verir | `CAPITAL:Veli` | `HELD:Ali` | 1 (Veli onaylar) | 1 |
| `SETTLEMENT` | Veli, Ali'ye elden 50 bin öder | `CAPITAL:Ali` | `CAPITAL:Veli` | 1 (Ali onaylar) | 1 |
| `HANDOVER` | Ali elindeki proje parasını Veli'ye devreder | `HELD:Veli` | `HELD:Ali` | 1 (Veli onaylar) | 1 |
| `OPENING_*` | Açılış kayıtları (Bölüm 6.4) | — | — | 2 (hep birlikte) | 1 |
| `SALE_ON_CREDIT` | Senetli satış | `RECEIVABLE` | `REVENUE` | 0 | 2 |
| `PARTNER_FEE` | Mehmet'e sabit emek ücreti | `EXPENSE:emek ücreti` | `PAYABLE:Mehmet` | 2 | 2 |
| `DISTRIBUTION` | Ara ya da kesin dağıtım | `CAPITAL:x` veya `PAYABLE:x` | `CASH` veya `HELD:y` | 1 (alan onaylar) | 2 |
| `CLOSING` | Kapanışta net sonucun sermaye hesaplarına dağıtılması | Bölüm 7.3 | Bölüm 7.3 | 2 | 2 |

**Bakiye tanımı:** Borç doğallı bir hesabın bakiyesi = borçlar toplamı − alacaklar toplamı. Alacak doğallı hesapta tersi. Hesaba sadece geçerli revizyonlardan, `status = ACTIVE` ve `approval_state = APPROVED` olanların hareketleri katılır. 0. seviye kayıtlar oluşturuldukları anda onaylı sayılır.

**Değişmez kural:** Her hareketin borç ve alacak tarafı aynı tutardır. Bu yüzden defterin toplamı her zaman sıfırdır.

### 6.3 Revizyon, iptal ve düzeltme yetkisi

- Bir kaydı sadece onu giren kullanıcı düzeltebilir ya da iptal edebilir. Diğer ortaklar itiraz eder. Bu kural sayesinde senkronizasyon çakışmaları sadece aynı kullanıcının iki farklı cihazından gelebilir.
- Düzeltme, `revision_no + 1` numaralı yeni bir satır olarak eklenir. İptal, `status = VOID` olan yeni bir revizyondur. Eski revizyonlar silinmez ve geçmiş herkese açıktır.
- Düzeltme isteği hangi revizyonun üzerine yapıldığını (`base_revision`) belirtir. Sunucudaki geçerli revizyon farklıysa istek çakışma (409) ile döner.
- 1. seviye bir kayıt onay beklerken onu giren kişi düzeltebilir ya da iptal edebilir. Onaylandıktan sonra kilitlenir; düzeltmek için ters yönde yeni bir kayıt girilir.
- Açılış kayıtları açılış onayıyla birlikte kilitlenir.

### 6.4 Proje oluşturma ve açılış durumu

Yeni bir proje de, zaten devam eden bir proje de aynı sihirbazla oluşturulur:

1. Proje bilgileri ve tipi girilir.
2. Ortaklar ve sermaye oranları girilir (`ownership_version`, türü `INITIAL`). Projeyi oluşturan kullanıcı ortaklardan birine bağlı olmak zorundadır.
3. Devam eden projelerde geçmiş durum girilir (bu adım opsiyonel):
   - **Her ortağın bugüne kadar koyduğu toplam:** `CAPITAL:i` alacaklanır, `OPENING` borçlanır. İsteğe bağlı olarak tarihli birden fazla satır girilebilir; bu tarihler Faz 2'deki reel getiri hesabında kullanılır.
   - **Bilinen büyük kalemler** (opsiyonel), örneğin arsanın alış bedeli: `EXPENSE` borçlanır, `OPENING` alacaklanır.
   - **Geçmiş satışlardan tahsil edilen tutar:** `OPENING` borçlanır, `REVENUE` alacaklanır.
   - **Tahsil edilmemiş alacak** (senet gibi): `RECEIVABLE` borçlanır, `REVENUE` alacaklanır.
   - **Paranın şu an nerede olduğu:** kasadaki tutar için `CASH` borçlanır, `OPENING` alacaklanır. Ortağın elinde duran tutar için `HELD:i` borçlanır, `OPENING` alacaklanır.
   - **Kalan geçmiş masraf toplamını sistem hesaplar:** koyulan toplam + tahsil edilen satış − bilinen kalemler − eldeki nakit. Bu tutar "Aktarım öncesi masraflar" kategorisine yazılır (`EXPENSE` borçlanır, `OPENING` alacaklanır). Sonuç negatif çıkarsa sihirbaz "girilen rakamlar tutarsız" uyarısı verir ve onaya göndermeye izin vermez.
4. Durum onaya gönderilir. Bu bir 2. seviye işlemdir, yani projenin tüm ortaklarının onayı gerekir. Onay gelene kadar proje `OPENING` durumunda kalır. Bu sürede 0. seviye kayıtlar girilebilir, çünkü sermaye hesapları sermaye oranlarından bağımsızdır. "Kim kime borçlu" ekranı ise onay beklendiğini gösterir.
5. Onay gelince proje `ACTIVE` olur ve açılış kayıtları kilitlenir. Onay reddedilirse sihirbaz düzenlenebilir hale döner.

Açılış tamamlandığında `OPENING` hesabının bakiyesi sıfırdır (değişmez kural).

### 6.5 Para tutarları

- Tutarlar Java'da `BigDecimal`, veritabanında `numeric(19,4)` olarak tutulur. Arayüzde 2 ondalık hane gösterilir. Para için `double` ya da `float` kullanmak yasak; bu kural mimari testlerle denetlenir.
- Bir tutar paylaştırılırken (örneğin 100 TL'yi üç ortağa bölerken) sonuç kuruşa yuvarlanır. Yuvarlamadan artan kuruşlar **en büyük kalan yöntemiyle** her seferinde aynı şekilde dağıtılır. Parçaların toplamı her zaman tam tutara eşittir.
- Faz 1'de tek para birimi TRY. Yine de `currency` ve `fx_rate` alanları baştan modelde var. Faz 2'de kur, kaydın tarihine göre sunucu tarafından doldurulacak; eski kayıtlar da geriye dönük doldurulacak.

### 6.6 Kişisel verilerin defterden ayrılması

Kişisel veriler sadece `app_user` ve `partner` tablolarında durur. Defter tabloları (`entry_revision`, `movement`) yalnızca kimlik referansı tutar. Bir hesap silindiğinde kişisel alanlar anonimleştirilir (örneğin "Silinmiş ortak #3"). Finansal kayıtlar ise diğer ortakların hesaplaşması için yerinde kalır. Bu iş Faz 3'te, hukuki incelemeyle birlikte yapılacak.

## 7. Hesaplamalar

### 7.1 Bakiyeler (Faz 1)

- `capital_i` = ortak i'nin `CAPITAL` hesabının bakiyesi, yani net katkısı.
- `C` = tüm ortakların sermaye bakiyelerinin toplamı (Σ capital_i). `r_i` = ortak i'nin onaylı sermaye oranı.
- Her ortağın koymuş olması gereken tutar: `target_i = C × r_i`.
- Fark: `d_i = capital_i − target_i`. Pozitif çıkarsa ortak alacaklı, negatif çıkarsa borçludur. Farkların toplamı her zaman sıfırdır (Σ d_i = 0).
- Proje nakdi = kasa (`CASH`) + ortaklarda duran nakit (Σ `HELD:i`).

### 7.2 "Kim kime borçlu" önerileri (Faz 1)

Sistem dengeyi sağlamak için seçenekler önerir. Kullanıcı birini seçer ve ilgili kayıtlar 1. seviye olarak oluşturulur.

1. **Proje nakdinden geri ödeme.**
   - Herkesin dengede olacağı katkı düzeyi: `K* = min_i (capital_i / r_i)`.
   - Ortak i'ye ödenecek tutar: `p_i = capital_i − r_i × K*`.
   - Toplam ödeme (Σ p_i) proje nakdinden fazlaysa kısmi öneri yapılır: eldeki nakit, alacaklılara `p_i` oranında dağıtılır.
   - Örnek: Ali'nin sermayesi 300 bin, diğerlerinin 100'er bin, oranlar %25. K* = 400 bin çıkar; Ali'ye 200 bin ödenir, diğerlerine bir şey ödenmez.
2. **Elden ödeşme.** Borçlular alacaklılara öder. Transfer sayısını azaltmak için en çok borcu olan, en çok alacağı olanla eşleştirilir (açgözlü eşleştirme).
3. **Kapanışa bırakmak.** Hiçbir şey yapılmaz; fark proje kapanırken sermaye iadesiyle kapanır.
4. **Hisseye dahil etmek** (Faz 2). Bölüm 7.4.

### 7.3 Kapanış ve dağıtım (Faz 2)

1. **Önkoşullar:** bütün alacaklar tahsil edilmiş ya da silinmiş olmalı; onay bekleyen kayıt kalmamalı.
2. **Net sonuç:** `N = REVENUE − Σ EXPENSE`. Sabit emek ücretleri de masrafa dahildir.
3. **Paylaştırma oranları:**
   - `N ≥ 0` ise kâr, kâr oranlarıyla bölünür: `k_i = e_i + (1 − Σe) × r_i`. Burada `e_i`, ortak i'nin emek payı yüzdesidir; emek payı yoksa 0'dır.
   - `N < 0` ise zarar, sermaye oranlarıyla (`r_i`) paylaşılır.
4. **Kapanış kayıtları** (`CLOSING`): `REVENUE` ve `EXPENSE` hesapları kapatılır, `N` sermaye hesaplarına dağıtılır. Yuvarlamada en büyük kalan yöntemi kullanılır.
5. Bu noktada `capital_i + payable_i`, ortak i'ye ödenecek tutardır. Bu tutarların toplamı proje nakdine eşittir (değişmez kural).
6. **Dağıtım kayıtları** (`DISTRIBUTION`, 1. seviye) ödemeleri kaydeder. Bütün hesaplar sıfırlanınca proje `CLOSED` olur.

Kapanış önerisi tek parça halinde 2. seviye onaya gider.

**Doğrulama örneği:** Dört ortak her biri 500 bin koyarak arsayı 2 milyona alıyor. Ali ayrıca tek başına 200 bin masraf ödüyor. Arsa 4 milyona satılıyor ve para kasaya giriyor. N = 1,8 milyon. Oranlar eşit (%25), dolayısıyla her ortağa 450 bin kâr düşer. Ali'ye 700 + 450 = 1.150 bin, diğerlerine 500 + 450 = 950'şer bin ödenir. Toplam 4 milyon, kasadaki paraya eşit.

**Emek payı eşdeğerliği:** "Kârın %20'si önce Mehmet'e, kalanı eşit bölünsün" kuralı, Mehmet'e %40, diğerlerine %20'şer kâr oranı vermekle aynı sonucu verir. Bu yüzden sistemde tek mekanizma var: kâr oranı. Kâr oranları her seferinde kuraldan yeniden hesaplanır; böylece hisseler değiştiğinde kendiliğinden güncellenir.

### 7.4 Ortaklık yapısı değişikliği (Faz 2)

Tek bir olay tipidir ve 2. seviye onay gerektirir. Her değişiklik yeni bir `ownership_version` ve buna bağlı kayıtlar üretir. Kullanıcı arayüzde dört ayrı adım adım akıştan birini seçer:

| Akış | Hareketler | Yeni oranlar |
|---|---|---|
| Hisse devri (pazarlıkla belirlenen fiyat) | Satan ortağın sermaye bakiyesi, devrettiği pay oranında alana aktarılır (`CAPITAL:satan` borçlanır, `CAPITAL:alan` alacaklanır). Ortaklar arasında ödenen fiyat proje dışı bir ödemedir; bilgi notu olarak kaydedilir. | Kullanıcı girer |
| Sermaye iadesiyle çıkış | `CAPITAL:çıkan` borçlanır, `CASH` ya da `HELD` alacaklanır. Proje nakdi yetmezse ödemeyi fiilen kalan ortaklar yapar; bu ekonomik olarak bir devirdir ve devir olarak kaydedilir. | Kalan ortaklar arasında oranları korunarak dağıtılır; kullanıcı değiştirebilir |
| Sermaye ekleyerek yeni ortak girişi | `CASH` borçlanır, `CAPITAL:yeni` alacaklanır | Kullanıcı girer |
| Masrafı hisseye dahil etme | Hareket yok | Sermaye bakiyeleri oranında yeniden hesaplanır |

Kâr, kapanış anındaki oranlarla hesaplanır. Geçmişteki masraflar o günkü oranlara göre tek tek yeniden bölünmez.

## 8. Mobil deneyim hedefleri (Faz 1)

- **Hızlı kayıt:** uygulamayı açtıktan sonra 10 saniye içinde kayıt tamamlanabilmeli. Akış: tutar (önce sayı klavyesi) → kategori → kaydet. Ödeyen varsayılan olarak kullanıcının kendisi, tarih varsayılan olarak bugün. Fotoğraf tek dokunuşla eklenir.
- **Sıralama:** son kullanılan proje ve kategoriler en üstte gösterilir.
- **Senkron durumu:** her kaydın üzerinde görünür: bekliyor, gönderildi, reddedildi ya da çakışma.
- **Dil ve biçim:** arayüz Türkçe; para `1.250.000,00 ₺` biçiminde gösterilir. Metinler kodun dışında tutulur (Flutter intl), böylece ileride başka dil eklemek kolay olur.
- **Bildirimler:** MVP'de uygulama içinde bir hareket akışı ve "onay bekleyenler" rozeti var. Telefona gelen anlık bildirim (push) Faz 2'de.

## 9. Onay modeli

| Seviye | Kapsadığı işlemler | Kural |
|---|---|---|
| 0: Kayıt | `EXPENSE`, `INCOME`, `CONTRIBUTION`, `DEPOSIT`, `COLLECTION` (Faz 2'de ayrıca `SALE_ON_CREDIT`) | Hemen işlenir. Projenin ortaklarına uygulama içi bildirim gider. Her ortak itiraz edebilir. |
| 1: Para el değiştirme | `REIMBURSEMENT`, `SETTLEMENT`, `HANDOVER` (Faz 2'de ayrıca `DISTRIBUTION`) | Parayı alan ortak onaylayana kadar bakiyelere yansımaz. |
| 2: Anlaşma değişikliği | Proje oluşturma ve açılış (ilk ortaklık yapısı), görünürlük değişikliği. Faz 2'de ayrıca: ortaklık yapısı değişikliği, emek payı ve sabit ücret, kapanış. | Projenin tüm ortaklarının onayı gerekir (oybirliği). |

- **Başkası adına onay:** bağlı kullanıcısı olmayan, yani uygulamayı kullanmayan bir ortak için başka bir ortak not yazmak şartıyla onay verebilir. Bu onay geçmişte ayrıca işaretlenir. Bağlı kullanıcısı olan bir ortak adına onay verilemez.
- **Red:** tek bir red, talebin tamamını reddeder. Öneren kişi yeni bir talep açar.
- **İtiraz:** itiraz edilen kayıt bakiyelerde kalmaya devam eder, ama "tartışmalı" olarak işaretlenir ve raporlarda ayrı gösterilir. İtiraz iki şekilde kapanır: kaydı giren kişi kaydı düzeltir ya da iptal eder, veya itiraz eden itirazını geri çeker.
- Onaylar ve redler de değiştirilemeyen kayıtlar olarak saklanır.
- Onay vermek internet bağlantısı gerektirir.

## 10. Roller, yetkiler ve görünürlük

- **Çalışma alanı rolleri:** `ADMIN` kullanıcı davet eder, çıkarır ve çalışma alanı ayarlarını yönetir. `MEMBER` normal üyedir. **Yönetici rolü finansal bilgileri görme yetkisi vermez.**
- **Projeye erişim:**
  - Projede ortak olan birine bağlı kullanıcı tam erişime sahiptir: kayıt girer, itiraz eder, onay verir ve bütün finansal bilgileri görür.
  - Görünürlüğü `WORKSPACE` olan bir projeyi çalışma alanının diğer üyeleri salt okunur görür.
  - Görünürlüğü `PRIVATE` olan bir projeyi (varsayılan) diğer üyeler hiç görmez.
- Görünürlüğü değiştirmek 2. seviye onay gerektirir.
- **Proje oluşturma:** çalışma alanının her üyesi proje oluşturabilir. Projeyi oluşturan kişi o projenin ortaklarından birine bağlı olmak zorundadır.
- **Ortak kayıtları** çalışma alanı düzeyindedir ve her üye ortak ekleyebilir. Ortak eklemek o kişiye projeye erişim vermez; erişim proje ortaklığından ve ortak–kullanıcı bağından gelir.
- Bir kullanıcıyı çalışma alanından çıkarmak onu ortaklıktan çıkarmaz. Kullanıcının erişimi kapanır, ama ortak kaydı ve bakiyeleri yerinde kalır.
- **Tek yetki kontrol noktası:** `AccessPolicy.check(user, action, resource)`. Bütün komutlar ve sorgular buradan geçer. Senkronizasyonda veri çekilirken de aynı kontrol uygulanır.
- **Saha rolü** (`FIELD`) Faz 1'de yok. Kurumsal fazda proje rolü olarak eklenecek. Saha çalışanı sadece kendi kayıtlarını görür; kârı ve bakiyeleri göremez.

**Faz 1 yetki tablosu:**

| İşlem | Ortağa bağlı kullanıcı | Diğer üye (görünür proje) | Diğer üye (gizli proje) |
|---|---|---|---|
| Projeyi görmek | ✓ | Salt okunur | ✗ |
| Kayıt girmek | ✓ | ✗ | ✗ |
| Kendi kaydını düzeltmek ya da iptal etmek | ✓ | ✗ | ✗ |
| İtiraz etmek | ✓ | ✗ | ✗ |
| Onay vermek | ✓ | ✗ | ✗ |

Kullanıcı davet etmek ve çıkarmak sadece `ADMIN` yetkisindedir. Bu yetki projelere erişim sağlamaz.

## 11. Offline çalışma ve senkronizasyon

### 11.1 Mobilde yerel veri

- Telefonda drift (SQLite) ile tutulan bir yerel veritabanı var. İçinde üç şey bulunur: kullanıcının görebildiği projelerin bir kopyası, `outbox` (gönderilmeyi bekleyen komutlar) ve `upload_queue` (yüklenmeyi bekleyen fotoğraflar).
- Kullanıcının yaptığı her işlem önce bu yerel veritabanına yazılır ve ekranda hemen "senkron bekliyor" etiketiyle görünür.
- Offline iken sadece 0. seviye işlemler yapılabilir: kayıt girmek, kendi kaydını düzeltmek ya da iptal etmek, itiraz etmek. Onaylar, 1. seviye kayıtlar, açılış ve ortaklık işlemleri internet gerektirir.
- Bakiyeler telefonda hesaplanmaz. Ekranda son senkronizasyonda sunucudan alınan bakiyeler gösterilir. Henüz gönderilmemiş yerel kayıtlar ayrı bir listede durur.

### 11.2 Gönderim (push)

`POST /api/v1/workspaces/{workspaceId}/sync/push`

```json
{ "deviceId": "…", "commands": [ { "commandId": "…", "type": "CreateEntry", "entryId": "…", "baseRevision": null, "payload": { } } ] }
```

Her komut için şu sonuçlardan biri döner:

- `APPLIED`: komut işlendi.
- `DUPLICATE`: aynı `commandId` daha önce işlenmişti; önceki sonuç aynen döner.
- `CONFLICT`: kayıt bu arada başka bir cihazdan değişmiş; geçerli revizyon da gönderilir.
- `REJECTED`: doğrulama ya da yetki hatası; sebebiyle birlikte döner.

Kurallar:

- Bu uçtan gönderilebilen komut tipleri: `CreateEntry` (sadece 0. seviye kayıt türleri), `ReviseEntry`, `VoidEntry`, `RaiseDispute`. Diğer işlemler normal API adreslerinden, online olarak yapılır.
- Komutlar sırayla işlenir ve her biri kendi veritabanı işleminde çalışır. Birinin reddedilmesi diğerlerini durdurmaz.
- Aynı komutun iki kez işlenmesini `processed_command` tablosu engeller. Böylece telefon aynı komutu tekrar gönderse bile sorun olmaz.
- Yetki gönderim anında yeniden kontrol edilir. Kullanıcı offline'dayken erişimi kapatıldıysa komutu reddedilir ve bu ona gösterilir.

### 11.3 Çekme (pull)

`GET /api/v1/workspaces/{workspaceId}/sync/pull?cursor=…&limit=…` → `{ changes, nextCursor, hasMore, revokedProjects }`

- Veritabanına yapılan her yazma, aynı veritabanı işlemi içinde `change_log` tablosuna da bir satır ekler. Bu satırda işlemin kimliği (`tx = pg_current_xact_id()`) ve bir sıra numarası (`seq`) bulunur.
- **Commit sırası tuzağı:** Sıra numarası, işlemlerin commit olma sırasına göre değil, numarayı alma sırasına göre artar. Küçük numaralı bir satır daha geç commit olabilir. Telefon bu arada daha büyük bir numaradan devam etmişse o satırı hiç görmez ve kayıt kaybolur.
- **Çözüm:**
  - Telefonun kaldığı yeri gösteren imleç `(tx, seq)` çiftidir.
  - Çekme sadece `tx < pg_snapshot_xmin(pg_current_snapshot())` koşulunu sağlayan satırları `(tx, seq)` sırasıyla döndürür. Yani yalnızca kesin olarak bitmiş işlemlerin satırlarını verir.
  - Henüz bitmemiş bir işlemin satırı, imleç onu geçmeden önce mutlaka döndürülür; böylece hiçbir satır atlanmaz.
  - Bu davranış, aynı anda yazan işlemlerle bir test üzerinden doğrulanacak.
- Telefon gelen değişiklikleri aynı değişiklik iki kez gelse de sonuç değişmeyecek şekilde uygular (varlık kimliği ve revizyon numarasına bakarak).
- Kullanıcının erişimi kapanan projeler `revokedProjects` listesinde bildirilir ve telefon o projenin yerel kopyasını siler.

### 11.4 Fotoğraflar

1. Telefon fotoğraf için bir kimlik üretir ve kayda ekler. Kayıt fotoğraftan önce senkronize olabilir; o arada ekranda "fiş yükleniyor" yazar.
2. `POST /api/v1/workspaces/{workspaceId}/files` çağrısı, kısa süre geçerli imzalı bir yükleme linki döndürür.
3. Telefon fotoğrafı bu linkle doğrudan nesne depolamaya yükler. Yükleme yarıda kalırsa yeniden dener.
4. `POST /api/v1/workspaces/{workspaceId}/files/{id}/complete` çağrısıyla sunucu dosyanın gerçekten yüklendiğini ve boyutunu doğrular, durumunu `UPLOADED` yapar.

- **Okuma:** fotoğraf kısa süre geçerli imzalı bir indirme linkiyle açılır. Her istek yetki kontrolünden geçer.
- **Boyut:** telefon fotoğrafı yüklemeden önce küçültür (uzun kenar 2000 piksel, JPEG). Üst sınır 10 MB.

### 11.5 Diğer kurallar

- **Mükerrer kontrolü sunucuda yapılır.** Aynı projede, aynı tutarda, aynı kategoride ve tarihi ±3 gün içinde olan kayıtlar "olası mükerrer" olarak işaretlenir. Kayıt reddedilmez.
- **Tarihler:** `occurred_on`, kullanıcının girdiği tarihtir (telefonun yerel tarihi). Sunucu ayrıca kendi saatiyle `created_at` ve telefondan gelen `client_created_at` alanlarını saklar.
- **Kur** (Faz 2) sunucuda doldurulur.

## 12. API

- REST + JSON. Tüm adresler `/api/v1` altında. Sözleşme `api/openapi.yaml` dosyasında.
- **Kimlik doğrulama:** `Authorization: Bearer <OIDC access token>`. Çalışma alanı adreste yer alır: `/api/v1/workspaces/{workspaceId}/…`.
- **Tekrar güvenliği:** yazma işlemleri istemcinin ürettiği kimliği (UUIDv7) taşır, bu yüzden aynı istek tekrar gönderilse de sorun olmaz.
- **Hatalar** RFC 9457 Problem Details biçiminde döner: `type`, `title`, `status`, `detail`; bunlara ek olarak makinenin okuyabileceği bir `code` ve doğrulama hataları için bir `errors[]` listesi.
- **Çakışma:** 409 ve geçerli revizyon.
- **Faz 1'deki ana adres grupları:** `me`, `workspaces`, `invitations`, `partners`, `projects` (`opening` dahil), `entries`, `approvals`, `disputes`, `balances`, `files`, `sync`.

## 13. Güvenlik

- **Giriş yöntemleri:** Keycloak üzerinden e-posta ve şifre, Google ve Apple (Faz 1). SMS ile giriş Faz 3'te eklenecek; ya bir Keycloak eklentisiyle ya da kimlik sunucusunu değiştirerek. İki durumda da backend etkilenmez.
- **Mobilde giriş:** OIDC Authorization Code + PKCE akışı, telefonun sistem tarayıcısı üzerinden. Token'lar iOS Keychain'de ve Android Keystore'da saklanır.
- **Backend:** Spring Security OAuth2 resource server. JWT imzası, `iss` ve `aud` alanları doğrulanır.
- **Kiracıların verilerini ayırma:**
  - Her veritabanı işleminin başında aktif çalışma alanı bağlantıya bildirilir: `SET LOCAL app.workspace_id`.
  - Kiracıya ait bütün tablolarda RLS politikası açıktır.
  - Uygulamanın veritabanı kullanıcısı tabloların sahibi değildir, bu yüzden RLS'yi atlayamaz.
  - Bir kiracının başka bir kiracının verisine ulaşamadığı entegrasyon testleriyle denenir.
- **Proje düzeyindeki yetkiler** uygulamada `AccessPolicy` ile kontrol edilir.
- **Fiş fotoğrafları** herkese kapalı bir depoda durur; sadece kısa süre geçerli imzalı linklerle erişilir.
- **Davet linki:** rastgele üretilmiş bir token içerir. Veritabanında token'ın kendisi değil, hash'i tutulur. Linkin süresi ve kullanım sayısı sınırlıdır.
- **Gizli anahtarlar** kod deposunda tutulmaz; ortam değişkenleri ya da sunucudaki dosyalar kullanılır.
- **Yedekleme:**
  - PostgreSQL için günlük tam yedek ve WAL arşivi alınır. WAL arşivi, istenen herhangi bir ana geri dönebilmeyi sağlar (PITR).
  - Yedekler sunucunun dışındaki S3 uyumlu depoya gider.
  - Ayda bir geri yükleme testi yapılır.
  - Araç olarak pgBackRest düşünülüyor; plan aşamasında kesinleşecek.
- **Telefondaki veritabanını şifreleme ve biyometrik kilit:** Faz 3.
- **KVKK:** pilot sizin kendi verilerinizle yürütülecek. Dışarıdan müşteri almadan önce hukuki inceleme yapılacak (Faz 3).

## 14. Altyapı ve dağıtım

- **Pilot sunucusu:** arkadaşından alınacak tek bir sunucu. Üzerinde Docker Compose ile dört servis çalışır: `caddy`, `api`, `keycloak`, `postgres`. Keycloak kendi verisini aynı PostgreSQL sunucusunda ayrı bir veritabanında tutar.
- **Nesne depolama:** sunucunun dışında S3 uyumlu bir servis; fiş fotoğrafları ve yedekler burada durur. Sağlayıcı plan aşamasında seçilecek.
- **Ortamlar:** geliştirme (yerel Docker Compose) ve pilot (sunucu). Dışarıdan müşteri almadan önce sizin kontrolünüzdeki bir altyapıya taşınılacak.
- **Mobil dağıtım:** iOS için TestFlight (bireysel Apple geliştirici hesabıyla), Android için dahili test kanalı.
- **CI/CD:** plan aşamasında ayrıca konuşulacak ve onayınla kurulacak.

## 15. Kod deposu yapısı

```
api/openapi.yaml      API sözleşmesi (tek kaynak)
backend/              Spring Boot (Maven)
mobile/               Flutter
web/                  Vue 3 + Vuetify (Faz 2)
infra/                docker-compose, Caddyfile, Keycloak ayar dışa aktarımı
docs/                 tasarım belgeleri, açık sorular
```

## 16. Hata yönetimi

- **Sunucu tarafı:**
  - Doğrulama hataları 400 ya da 422 döner, Problem Details biçiminde.
  - Yetki hatası 403 döner.
  - Bulunamayan kaynak 404 döner. Başka bir kiracıya ait kaynak istendiğinde de 404 döner, böylece o kaynağın var olduğu bile sızmaz.
  - Çakışma 409 döner.
  - Beklenmeyen hatalar 500 döner ve bir korelasyon kimliğiyle loglanır.
- **Defter kuralları veritabanında da korunur.** Örneğin tutarın sıfırdan büyük olması ve kayıtların değiştirilemezliği veritabanı kısıtlarıyla da sağlanır. Bu kurallardan biri ihlal edilirse bu bir programlama hatasıdır ve işlem geri alınır.
- **Mobil tarafı:**
  - Gönderim başarısız olursa, denemeler arasındaki süre giderek uzatılarak yeniden denenir.
  - `REJECTED` ve `CONFLICT` sonuçları kaydın üzerinde görünür ve kullanıcıdan bir karar ister.
  - Hiçbir yerel kayıt kullanıcı onaylamadan silinmez.
- Fotoğraf yüklemesi başarısız olsa bile kayıt engellenmez; kuyruk yüklemeyi tekrar dener.

## 17. Test stratejisi

- **Defter ve hesaplama kuralları (en kritik kısım):** jqwik ile özellik tabanlı testler yazılacak. Rastgele üretilen çok sayıda senaryoda şu kuralların hep geçerli olduğu kontrol edilecek:
  - Her hareket dengelidir; defterin toplamı sıfırdır.
  - "Kim kime borçlu" önerileri uygulandığında bütün farklar (`d_i`) sıfıra iner.
  - Yuvarlamada parçaların toplamı tam tutara eşittir.
  - Açılış sonrasında `OPENING` hesabı sıfırdır.
  - (Faz 2) Kapanış sonrasında bütün hesaplar sıfırdır; dağıtılan toplam proje nakdine eşittir.
- **Örneklerle testler:** bu belgedeki örnekler birebir test olarak yazılır: Ali'nin kasadan 200 bin geri ödeme senaryosu, 4 milyonluk satış örneği, emek payı eşdeğerliği.
- **Entegrasyon testleri:** Testcontainers ile gerçek bir PostgreSQL üzerinde çalışır. Kapsamı:
  - kiracılar arası veri sızıntısını deneyen RLS testleri,
  - aynı anda yazan işlemlerle `change_log` testi,
  - aynı komutun tekrar gönderildiği push testleri.
- **Sözleşme testleri:** backend yanıtlarının OpenAPI sözleşmesine uyduğu doğrulanır.
- **Mimari testler** (ArchUnit): modül sınırlarına uyuluyor mu, para için `double` ya da `float` kullanılmış mı.
- **Mobil:** outbox ve senkronizasyon mantığı için birim testleri. Sahte bir API ile çakışma, tekrar gönderme ve red senaryoları denenir. Kritik ekranlar için widget testleri yazılır.
- **Web (Faz 2):** Vitest ve bileşen testleri.

## 18. Fazlar

### Faz 1: MVP (kâğıdı bırakmak)

Sunucu ve mobil uygulama:

1. Giriş (e-posta ve şifre, Google, Apple) ve profil.
2. Çalışma alanı oluşturma, davet linki (WhatsApp gibi kanallardan paylaşılır) ve katılma.
3. Ortaklar: uygulamayı kullanan ya da kullanmayan.
4. Proje oluşturma ve açılış sihirbazı (`LAND`, `VEHICLE`, `GENERIC` şablonları ve varsayılan kategorileriyle) ve açılış onayı.
5. Kayıtlar: masraf, peşin gelir, sermaye koyma, kasaya yatırma, açılıştaki alacağın tahsilatı. Fiş fotoğrafı, geçmiş tarihli kayıt, başka bir ortağın ödediği kaydı girme.
6. Düzeltme ve iptal (sadece kaydı giren kişi), revizyon geçmişini görme.
7. Hareket akışı, itiraz, "olası mükerrer" işareti.
8. Bakiyeler, "kim kime borçlu" ekranı, geri ödeme önerileri (proje nakdinden ya da elden), 1. seviye onaylar.
9. Onay kutusu (1. ve 2. seviye) ve başkası adına onay.
10. Proje görünürlüğü (değiştirmek 2. seviye onay ister).
11. Offline kuyruk, senkronizasyon ve fotoğraf kuyruğu.
12. Altyapı: Docker Compose, Keycloak, yedekleme, TestFlight ve Android dahili test kanalı.

Kabul ölçütü: Bölüm 3.

### Faz 2: Gerçek tablo

- Web v1: mobildeki bütün işlemler, yönetim ekranları ve raporlar.
- Kapanış ve kesin hesap.
- Emek payı ve sabit emek ücreti.
- Ortaklık yapısı değişikliği akışları (masrafı hisseye dahil etme dahil).
- Senetli satış ve tahsilat takibi.
- Ara dağıtım.
- Kur ve TÜFE ile enflasyondan arındırılmış getiri.
- Birim (parsel, araç) bazında maliyet ve başabaş fiyatı.
- Anlık bildirim (push) ve e-posta.

### Faz 3: Ürün

- KVKK hukuki incelemesi ve gerekli metinler.
- Hesap silme ve anonimleştirme.
- Mağazalarda yayın.
- Telefondaki veritabanını şifreleme ve biyometrik kilit.
- İlk kullanım akışı.
- Araç şablonunun geliştirilmesi.
- SMS ile giriş.
- Şirket kurulumu ve mağaza hesabının şirket hesabına dönüştürülmesi.

### Kurumsal (talep geldiğinde)

- Saha rolü.
- Yöneticinin çalışan masrafını onayladığı akış.
- Çalışan avansı.
- Şube ve departman yapısı.
- Özel roller.
- Kurumsal tek oturum (SSO).
- Muhasebe programına dışa aktarım.
- Projeyi sadece belirli üyelere açma.

**Uygulama planı sadece Faz 1'i kapsayacak.** Diğer fazlar kendi planlarıyla gelecek.

## 19. Varsayılan kategoriler

Bu kategoriler şablonun bir parçasıdır. Her çalışma alanı kendi kategorilerini ekleyebilir ve kullanmadıklarını arşivleyebilir.

- **`LAND` (arsa ve tarla):** Alım bedeli; Tapu harcı ve döner sermaye; Emlakçı komisyonu; Noter; Harita, ifraz ve kadastro; Avukat ve danışmanlık; Çit ve tel örgü; Su ve sondaj; Elektrik; Yol ve hafriyat; Fidan ve peyzaj; İşçilik; Malzeme; İlan ve reklam; Emlak vergisi; Ulaşım ve yakıt; Diğer.
- **`VEHICLE` (araç):** Alım bedeli; Noter; Ekspertiz; Muayene; Trafik sigortası ve kasko; MTV; Bakım ve onarım; Boya ve kaporta; Yedek parça; Temizlik ve detaylı bakım; İlan; Komisyon; Otopark; Ulaşım ve yakıt; Diğer.
- **`GENERIC` (genel):** Alım bedeli; İşçilik; Malzeme; Komisyon; Vergi ve harç; Ulaşım; İlan; Diğer.
- **Sistem kategorisi** (her şablonda bulunur, kullanıcı seçemez): Aktarım öncesi masraflar.

## 20. Riskler

| Risk | Etkisi | Önlem |
|---|---|---|
| Senkronizasyon hatası yüzünden kayıt kaybolması | Güven kaybı; ürün kullanılmaz hale gelir | Outbox; tekrar gönderilmeye dayanıklı push; işlem kimliğine dayalı çekme; aynı anda yazma testleri; yerel kayıt kullanıcı onaylamadan silinmez |
| Hesaplama hatası (yanlış bakiye) | Ortaklar arasında anlaşmazlık | Değiştirilemez defter; değişmez kural testleri; bu belgedeki örneklerin test olarak yazılması |
| Bir kiracının verisinin başkasına görünmesi | Ciddi güvenlik ve hukuk sorunu | RLS + uygulamadaki yetki kontrolü + sızıntı testleri |
| Tek sunucunun arızalanması | Kesinti ve veri kaybı | Sunucu dışında yedek ve WAL arşivi; ayda bir geri yükleme testi |
| Arkadaşın sunucusunda verilere erişebilmesi | Gizlilik | Sadece pilot dönemde; dış müşteriden önce taşınma |
| İki ayrı arayüzün (Flutter + Vue) bakım yükü | Geliştirme yavaşlar, platformlar arasında özellik farkı oluşur | İş kuralları sunucuda; istemciler OpenAPI'den üretiliyor; mobil bilerek dar tutuluyor |
| Kayıt girmek zahmetli gelirse ortakların WhatsApp'a dönmesi | Başarı ölçütü tutmaz | 10 saniyede kayıt hedefi; 0. seviyede onay beklenmiyor |
| Mobil geliştirme tecrübesinin az olması | Mobildeki hatalar geç fark edilir | Dart'ın Java'ya yakınlığı; mobil istemcinin ince tutulması; outbox için birim testleri |

## 21. Ertelenen konular

Ertelenen konular ve karar günlüğü [docs/open-questions.md](../../open-questions.md) dosyasında. Oradaki konuların hiçbiri Faz 1 tasarımını değiştirmiyor.
