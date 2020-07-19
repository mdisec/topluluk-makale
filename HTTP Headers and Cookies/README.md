# HTTP SECURITY HEADERS

Hyper Text Transfer Protokol (HTTP), sunucu ile istemci arasında iletişim kurarak hipermedya belgelerinin (HTML) iletimini sağlayan bir protokoldür. İletişim sırasında birtakım veri alışverişleri yapılması gerekir. Sunucu ve istemciye ait detayları, mesajın boyutunu, türünü ve çerezleri içeren bu veriler HTTP başlıkları ile belirtilir.

Aşağıdaki örnekte, Twitter web sitesinin kullandığı bazı HTTP başlıklarını görebiliriz.

![http-headers](images/http-headers.png)

Yazının devamında, bazı güvenlik zafiyetleri ve bunları engellemeye yardımcı olacak HTTP başlıkları anlatılacaktır.

### X-XSS-PROTECTION

Muhtemel XSS payloadlarını tespit edip filtreleyerek Reflected XSS saldırılarına karşı koruma sağlama amacıyla geliştirildi. Ancak frame buster mekanizmalarını devre dışı bırakmak, sayfadan veri çıkarabilmek gibi bazı güvenlik zafiyetlerine sebep oluyordu (Ayrıntılı bilgi için [bkz](https://medium.com/bugbountywriteup/xss-auditor-the-protector-of-unprotected-f900a5e15b7b)). Güncel tarayıcıların çoğunda desteklenmemektedir.

#### XSS nedir?
Cross Site Scripting (XSS), saldırganın kurbanın tarayıcısında keyfi JavaScript kodları çalıştırmasına izin veren bir güvenlik açığıdır.

**Kullanımı**
- **0:**
Filtrelemeyi devre dışı bırakır. 
- **1:**
Filtrelemeyi etkinleştirir.
- **mode=block:**
XSS tespiti halinde sayfanın yüklenmesini engeller. 
- **report=https://example.com/xss.log:**
XSS tespiti halinde saldırı girişimini belirtilen siteye raporlar. Sadece Chromium tabanlı tarayıcılarda kullanılır. 

`X-XSS-Protection: 1; mode=block; report=https://example.com/xss.log`

![x-xss-protection](images/x-xss-protection.png)

### X-Frame-Options

Sayfanın bir iframe içerisinde çağırılma durumunu kontrol eder. Clickjacking ve postMessage metodunun sebep olduğu XSS saldırılarına karşı koruma sağlar.

#### Clickjacking nedir?
Clickjacking, saldırganın zararsız gibi görünen bir siteye iframe elementi içerisinde başka bir web sitesi ekleyerek kullanıcıya istemediği işlemler yaptırmasıdır. 
Örneğin, kullanıcı bir hediye kazanacağını düşünerek ekranda gördüğü butona basar. Fakat saldırgan, iframe içerisine bir bankanın para transfer sayfasını koymuşsa kurban butona bastığı anda aslında ödemeyi onaylamış olur. Tarayıcı, bankaya gönderilen isteğe kurbanın çerezlerini de ekler ve saldırgana para transferi gerçekleşir.

*PoC:* 
- Birçok tarayıcıda desteklenmeyen ALLOW-FROM parametresinin kullanımı, Periscope'da clickjacking zafiyetine sebep olmuştur. 
- https://hackerone.com/reports/591432

#### postMessage XSS nedir?
postMessage; bir web sitesinin, içerisindeki iframe ile güvenli bir şekilde iletişim kurmasını sağlayan JavaScript metodudur. Eğer metodun uygulanmasında XSS zafiyetine sebep olacak bir kod yazılmış ise, saldırgan zafiyetli sayfayı kendi web sitesinde iframe etiketi içinde açarak XSS saldırısını gerçekleştirebilir.

*PoC:*
- Bu örnekte, https://platform.twitter.com sayfasının başka kaynaklar tarafından iframe içerisinde yüklenmesine izin verilmeseydi, kod zafiyetli olduğu halde XSS saldırısı gerçekleşemeyecekti. 
- https://hackerone.com/reports/29328

**Kullanımı**
- **DENY:**
Hiçbir şekilde iframe içerisinde kullanılmasına izin verilmez.
- **SAMEORIGIN:**
Yalnızca aynı domain tarafından kullanılmasına izin verir.
- **ALLOW-FROM URL:**
Yalnızca belirtilen URL tarafından kullanılmasına izin verir. Güncel tarayıcılarda desteklenmemektedir!

`X-Frame-Options: SAMEORIGIN`

![x-frame-options](images/x-frame-options.png)

### X-Content-Type-Options

Tarayıcının, uygulamanın belirlediği MIME türüne uymasını sağlar. Mime Type Sniffing saldırılarına karşı koruma sağlar.

#### Mime Type Sniffing nedir?
Mime Type Sniffing, Content-Type belirtilmeyen durumlarda tarayıcının belgenin içeriğini analiz ederek belgenin türünü tespit etmeye çalışmasıdır. Bir zafiyet türü değildir fakat XSS gibi bazı saldırılara sebep olabilir. 

Mesela, HTML dosyası yüklemeye izin vermeyen ancak Content-Type belirtmeyen bir uygulamaya HTML ve JS kodları içeren herhangi bir dosya yüklendiğinde, tarayıcı bu dosyayı HTML dosyası olarak kabul eder ve XSS zafiyeti oluşur.

*PoC:*
- Uber'in bu başlığı kullanmaması, yalnızca `.tar.gz` uzantılı dosyaların yüklenebildiği mirror sayfasında XSS zafiyetine sebep olmuştur. 
- https://hackerone.com/reports/126197

**Kullanımı**

`X-Content-Type-Options: nosniff`

![x-content-type-options](images/x-content-type-options.png)

### HSTS (HTTP Strict Transport Security)

Web sitesi ile tarayıcı arasındaki iletişimin yalnızca HTTPS üzerinden gerçekleşmesini sağlar. MITM saldırılarına karşı koruma sağlar.

#### Man in The Middle nedir?
MITM saldırıları, bir saldırganın ağ üzerindeki iletişimi dinlemesiyle gerçekleşir. HTTP bağlantılarında paketler şifrelenmediği için, hassas veriler saldırganın eline geçebilir.

**Kullanımı**
- **max-age:** 
Özelliğin tarayıcı hafızasında tutulacağı süreyi saniye olarak belirtir.
- **includeSubDomains:** 
Özelliğin tüm subdomainler için geçerli olacağını belirtir.
- **preload:** 
SSL sertifikasının tarayıcıların default HSTS listesine eklenmesini sağlar. Böylece güvenli iletişimin başlaması için, başlığın belirleneceği ilk yanıt beklenmez.

`Strict-Transport-Security: max-age=10886400; includeSubDomains; preload`

![strict-transport-security](images/strict-transport-security.png)

### Content-Security-Policy (CSP)

CSP, web sitesi içeriklerinin (JS kodları, CSS dosyaları, görüntüler vs.) hangi kaynaklardan yüklenmesine izin verileceğini ayarlar. XSS, Clickjacking gibi saldırılara karşı koruma sağlar. Google'ın geliştirdiği CSP Evaluator uygulaması ile, bir web sitesinin uyguladığı CSP kurallarını ve bunların olası tehlikelerini görebiliriz.

![csp-evaluator](images/csp-evaluator.png)

*PoC:*
- HackerOne'ın CSP kurallarını düzgün uygulaması, web sitesinde XSS zafiyeti olmasına rağmen -IE dışındaki tarayıcılarda- bu saldırıyı önlemektedir.
- https://hackerone.com/reports/474656

**Kullanımı**
- **base-uri:** base elementinde kullanılabilecek URL'leri kısıtlar.
- **default-src:** Default değer belirtir.
- **font-src:** @font-face kullanarak yüklenecek kaynakları belirtir.
- **form-action:** Form action olarak kullanılabilecek URL'leri kısıtlar.
- **frame-ancestors:** Sayfayı iframe elementi içerisinde *yükleyebilecek* URL'leri belirtir.
- **frame-src:** Sayfanın içerisinde iframe elementiyle *yüklenebilecek* URL'leri belirtir.
- **img-src:** Resimlerin yüklenebileceği kaynakları belirtir.
- **media-src:** audio, video gibi elementler kullanılarak yüklenecek medyaların kaynaklarını belirtir.
- **object-src:** object, embed, applet gibi elementler kullanılarak yüklenecek objelerin kaynaklarını belirtir. 
- **report-uri:** Belirtilen kuralları ihlal teşebbüsü oluştuğunda, raporun gönderileceği web sitesini belirtir.
- **script-src:** JavaScript kodları için kullanılabilecek kaynakları kısıtlar.
- **style-src:** Stil dosyaları için kullanılabilecek kaynakları kısıtlar.
- **upgrade-insecure-requests:** HTTP isteklerini HTTPS isteklerine çevirir.

* **self:** Yalnızca aynı site üzerinden yüklenmesine izin verir.
* **none:** Hiçbir kaynak üzerinden yüklenmesine izin verilmez.
* ***.example.com:** Belirtilen URL ve subdomainlerinden yükleme yapmaya izin verir.

- Yukarıda sadece bazı özellikler açıklanmıştır. Tüm liste için [bkz.](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy#Directives)

`Content-Security-Policy: default-src 'self' https://example.com` 

![content-security-policy](images/content-security-policy.png)

### Feature Policy

Kamera, mikrofon, sensörlere erişim ve aşağıda belirtilmiş birtakım tarayıcı özelliklerinin, sayfanın kendisi ve sayfa içerisindeki iframe elementi tarafından kullanım izinlerini ayarlar.

**Kullanımı** 
- **accelerometer:** İvmeölçer sensörünün kullanım iznini belirtir.
- **autoplay:** Medyaların otomatik oynatma özelliğini belirtir.
- **camera:** Kameraya erişim yetkisini belirtir.
- **fullscreen:** Element.requestFullScreen() ögesinin kullanımını denetler.
- **geolocation:** Kullanıcının konumuna erişim yetkisini denetler.
- **microphone:** Mikrofona erişim yetkisini belirtir.
- **picture-in-picture:** Videoların, resim içinde resim özelliği kullanılarak oynatılma iznini denetler.

* **self:** Belirtilen özelliği yalnızca sitenin kendisinin kullanmasına izin verir.
* **none:** Özelliğin kullanımına izin vermez.
* **\***: Özelliğin sitenin kendisi ve yüklediği iframe pencereleri tarafından kullanılmasına izin verir.
* **example.com:** Özelliğin belirtilen web sitesi tarafından kullanılmasına izin verir.

Tüm liste için [bkz.](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Feature-Policy#Directives)

`Feature-Policy: camera 'self'; microphone 'none'; autoplay * `

![feature-policy](images/feature-policy.png)

### Cross-Origin Resource Sharing (CORS) ve Access-Control-Allow-Origin

Kökenler arası kaynak paylaşımı (CORS), bir web sitesi üzerindeki bazı kaynakların, başka bir kökene sahip (farklı domain, protokol veya port) web sitesi tarafından kullanılabilmesini sağlayan mekanizmadır. Ajax çağrıları (XMLHttpRequest ve fetch API), drawImage() metoduyla çizilen canvas elementleri, CSS içerisinde @font-face ile çağrılan yazı tipleri ve [WebGL texture](https://developer.mozilla.org/en-US/docs/Web/API/WebGL_API/Tutorial/Using_textures_in_WebGL) istekleri için kullanılır. Kullanıcı profili gibi hassas veriler içeren sayfaların içeriğinin görüntülenmesine engel olarak bu verilerin ele geçirilmesini önler. Hangi kökenlere izin verileceği Access-Control-Allow-Origin başlığı ile belirlenir.

*PoC:*
- Twitter'a ait niche.co sitesinin bu başlığı yanlış kullanımı; kullanıcı bilgilerinin ifşalanması, değiştirilmesi, silinmesi ve CSRF tokeninin çalınmasına yol açmıştır.
- https://hackerone.com/reports/426147

**Kullanımı**
- ***:** Kimlik bilgisi gerektirmeyen istekler için, bütün kaynaklara izin verir.
- **example.com:** Yalnızca belirtilen kökenden gelen isteklere izin verir.
- **null:** Hiçbir kökene izin verilmeyeceğini belirtmek için kullanılsa da, "data://" ve "file://" olarak belirtilen kaynakların origin değeri null olabildiği için, null kullanımından kaçınılmalıdır.

`Access-Control-Allow-Origin: https://example.com`

![cors](images/cors.png)

### Http Public Key Pinning

Şifrelenmiş anahtar ile web sitesini eşleştiren bir güvenlik başlığıdır. Kullanıcı ve sunucu arasındaki ilk bağlantıda anahtar belirlenir ve sonraki isteklerin bu anahtarı içermesi beklenir. Yanlış anahtar gönderildiğinde kullanıcı uyarılır ve "report-uri" özelliği aktifse, belirtilen URL'ye rapor edilir. Sahte sertifikalar yoluyla yapılan MiTM saldırılarını engellemek için geliştirilmiştir. Şu anda güncel tarayıcılar tarafından desteklenmemektedir. ([Neden kaldırıldı?](https://groups.google.com/a/chromium.org/forum/#!msg/blink-dev/he9tr7p3rZ8/eNMwKPmUBAAJ))

**Kullanımı**
- **pin-sha256:**
Base64 olarak kodlanmış [SPKI](https://ldapwiki.com/wiki/Subject%20Public%20Key%20Info) parmak izidir.
- **max-age:**
Anahtarın tarayıcı hafızasında tutulacağı süreyi saniye olarak belirtir.
- **includeSubDomains:**
Anahtarın tüm subdomainler için geçerli olacağını belirtir.
- **report-uri:**
Pin doğrulama hataları belirtilen web sitesine bildirilir.

`Public-Key-Pins: pin-sha256="Yjk0ZDI3Yjk5MzRkM2UwOGE1MmU1MmQ3ZGE3ZGFiZmFjNDg0ZWZlMzdhNTM4MGVlOTA4OGY3YWNlMmVmY2RlOQ=="; max-age=5184000; includeSubDomains; report-uri="https://example.com/hpkp-report"`

![http-public-key-pinning](images/public-key-pinning.png)

### COOKIE FLAGS

HTTP çerezleri (cookies), kullanıcının tarayıcısında depolanan küçük verilerdir. Genellikle oturum takibi yapmak ve kullanıcının tercihlerini hatırlamak için kullanılır. Bu çerezlerin güvenliğini sağlamak için çerezlere eklenmesi gereken bazı parametreler vardır.

- **HttpOnly:** 
HttpOnly olarak ayarlanmış bir çerez yalnızca sunucuya gönderilir, JavaScript tarafından (document.cookie) erişilemez. Kimlik doğrulama çerezlerinin XSS saldırısıyla ele geçirilmesini engeller.
  - Amazon'un bu parametreyi kullanmaması, account takeover zafiyetine sebep olmuştur.
  - https://medium.com/@adam.adreleve/advanced-javascript-injections-amazon-xss-to-full-account-takeover-c1559e1c43ad

- **Secure:** 
Çerezlerin yalnızca HTTPS taleplerinde sunucuya gönderilmesini sağlar. HTTPS taleplerinde veriler şifreli olarak gönderildiği için ağı dinleyen saldırganın çerezleri ele geçirmesi önlenir.

- **SameSite:**
Çerezlerin üçüncü taraf isteklerine eklenip eklenmeyeceğini ayarlar.
  - **None:** Çerezlerin bütün üçüncü taraf isteklerine eklenmesine izin verir. `secure` özelliği ile birlikte kullanılmak zorundadır.
  - **Strict:** Çerezlerin hiçbir üçüncü taraf isteğine eklenmesine izin vermez.
  - **Lax:** Çerezleri yalnızca top level bir değişikliğe sebep olacak isteklere ekler. Yani gönderilen istek, sizi bulunduğunuz sayfadan başka bir sayfaya götürüyorsa, çerezler isteğe eklenir. `<a href>` etiketi ile gönderilen istekler buna örnektir. Ajax metodu veya `<img> <iframe>` etiketleri kullanılarak gönderilen isteklere ise çerezler eklenmez.

`Set-Cookie: sessionid=xAeeEjJyvOA0eCaD1qUFmCT3cfYxc9Tp8q1bonFtoVB10pDCx5GC6iw9I5nY8FXNAWZHEA; HttpOnly; Secure; SameSite=Strict`

![samesite](images/samesite.png)

**Sonuç olarak,** bu küçük ama etkili önlemleri almak web sitemizin güvenliğini artırabilir, bizi ve kullanıcılarımızı zafiyetlerden koruyabilir.

##### REFERANSLAR

- https://www.mehmetince.net/http-security-headerlari-neden-ve-nasil-kullanilmalidir/
- https://developer.mozilla.org/en-US/docs/*
- https://www.netsparker.com.tr/blog/web-guvenligi/*
- https://nullsweep.com/http-security-headers-a-complete-guide/
- https://tr.wikipedia.org/wiki/K%C3%B6kler_Aras%C4%B1_Kaynak_Payla%C5%9F%C4%B1m%C4%B1#%C4%B0stek_ba%C5%9Fl%C4%B1klar%C4%B1
- https://caniuse.com/*