# HTTP SECURITY HEADERS

Hyper Text Transfer Protokol (HTTP), sunucu ve istemci arasında iletişim kurarak hipermedya belgelerinin (HTML) iletimini sağlayan bir protokoldür. İletişim sırasında birtakım veri alışverişleri yapılması gerekir. Sunucu/istemcinin detaylarını, mesajın boyutunu, türünü ve çerezleri içeren bu veriler HTTP başlıkları ile belirtilir.

Aşağıdaki örnekte, HTTP istek ve yanıtlarında yer alan bazı başlıkları görebiliriz.

![http-headers](http-headers.png)

Bu yazının devamında, bazı güvenlik zafiyetlerine kısaca değinilecek ve bunları engellemeye yardımcı olacak HTTP başlıkları anlatılacaktır.

### X-XSS-PROTECTION

Muhtemel XSS payloadlarını tespit edip filtreleyerek Reflected XSS saldırılarına karşı koruma sağlar. 
Stored XSS karşısında çaresizdir!

#### XSS nedir?
Cross Site Scripting (XSS), saldırganın kurbanın tarayıcısında keyfi JavaScript kodları çalıştırmasına izin veren bir güvenlik açığıdır.

**Parametreleri**
- **0:**
Filtrelemeyi devre dışı bırakır. 
- **1:**
Filtrelemeyi etkinleştirir.
- **mode=block:**
XSS tespiti halinde sayfanın yüklenmesini engeller. 
- **report=https://website.com/xss.log:**
XSS tespiti halinde saldırı girişimini belirtilen siteye raporlar. Sadece Chromium tabanlı tarayıcılarda kullanılır. 

`X-XSS-Protection: 1; mode=block; report=https://website.com/xss.log`


### X-Frame-Options

Sayfanın bir iframe içerisinde çağırılma durumunu kontrol eder. Clickjacking saldırılarına karşı koruma sağlar.

#### Clickjacking nedir?
Clickjacking, saldırganın zararsız gibi görünen bir siteye iframe elementi içerisinde başka bir web sitesi ekleyerek kullanıcıya istemediği işlemler yaptırmasıdır. 
Örneğin, kullanıcı bir hediye kazanacağını düşünerek ekranda gördüğü butona basar. Fakat saldırgan, iframe içerisine bir bankanın para transfer sayfasını koymuşsa kurban butona bastığı anda aslında ödemeyi onaylamış olur. Tarayıcı, bankaya gönderilen isteğe kurbanın çerezlerini de ekler ve saldırgana para transferi gerçekleşir.

**Parametreleri**
- **DENY:**
Hiçbir şekilde iframe içerisinde kullanılmasına izin verilmez.
- **SAMEORIGIN:**
Yalnızca aynı domain tarafından kullanılmasına izin verir.
- **ALLOW-FROM URL:**
Yalnızca belirtilen URL tarafından kullanılmasına izin verir. Güncel tarayıcılarda desteklenmemektedir!

`X-Frame-Options: SAMEORIGIN`


### X-Content-Type-Options

Tarayıcının, uygulamanın belirlediği MIME türüne uymasını sağlar. Mime Type Sniffing saldırılarına karşı koruma sağlar.

#### Mime Type Sniffing nedir?
Mime Type Sniffing, Content-Type belirtilmeyen durumlarda tarayıcının belgenin içeriğini analiz ederek belgenin türünü tespit etmeye çalışmasıdır. Bir zafiyet türü değildir fakat XSS gibi bazı saldırılara sebep olabilir. 

Mesela, HTML dosyası yüklemeye izin vermeyen ancak Content-Type belirtmeyen bir uygulamaya HTML ve JS kodları içeren herhangi bir dosya yüklendiğinde, tarayıcı bu dosyayı HTML dosyası olarak kabul eder ve XSS zafiyeti oluşur.

`X-Content-Type-Options: nosniff`

### HSTS (HTTP Strict Transport Security)

Web sitesi ve browser arasındaki iletişimin yalnızca HTTPS üzerinden gerçekleşmesini sağlar. MITM saldırılarına karşı koruma sağlar.

#### Man in The Middle nedir?
MITM saldırıları, bir saldırganın ağ üzerindeki iletişimi dinlemesiyle gerçekleşir. HTTP bağlantılarında paketler şifrelenmediği için, hassas veriler saldırganın eline geçebilir.

**Parametreleri**
- **max-age:** 
Özelliğin browser hafızasında tutulacağı süreyi saniye olarak belirtir.

- **includeSubDomains:** 
Özelliğin tüm subdomainler için geçerli olacağını belirtir.

- **preload:** 
SSL sertifikasının tarayıcıların default HSTS listesine eklenmesini sağlar. Böylece güvenli iletişimin başlaması için, başlığın belirleneceği ilk yanıt beklenmez.

`Strict-Transport-Security: max-age=10886400; includeSubDomains; preload`

### Content-Security-Policy (CSP)

CSP, web sitesi içeriklerinin (JS kodları, CSS dosyaları, görüntüler vs.) hangi kaynaklardan yüklenmesine izin verileceğini ayarlar. XSS, Clickjacking gibi saldırılara karşı koruma sağlar.

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
* ***.website.com:** Belirtilen URL ve subdomainlerinden yükleme yapmaya izin verir.

- Yukarıda sadece bazı özellikler açıklanmıştır. Tüm liste için [bkz.](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy)

`Content-Security-Policy: default-src 'self' https://website.com` 


### COOKIE FLAGS

HTTP çerezleri (cookies), kullanıcının tarayıcısında depolanan küçük verilerdir. Genellikle oturum takibi yapmak ve kullanıcının tercihlerini hatırlamak için kullanılır. Bu çerezlerin güvenliğini sağlamak için çerezlere eklenmesi gereken bazı parametreler vardır.

- **HttpOnly:** 
HttpOnly olarak ayarlanmış bir çerez yalnızca sunucuya gönderilir, JavaScript tarafından (document.cookie) erişilemez. Session_id gibi önemli verilerin XSS saldırısıyla ele geçirilmesini engeller.

- **Secure:** 
Çerezlerin yalnızca HTTPS taleplerinde sunucuya gönderilmesini sağlar. HTTPS taleplerinde veriler şifreli olarak gönderildiği için ağı dinleyen saldırganın çerezleri ele geçirmesi önlenir.

`Set-Cookie: sessionid=xAeeEjJyvOA0eCaD1qUFmCT3cfYxc9Tp8q1bonFtoVB10pDCx5GC6iw9I5nY8FXNAWZHEA; HttpOnly; Secure`


##### REFERANSLAR

- https://www.mehmetince.net/http-security-headerlari-neden-ve-nasil-kullanilmalidir/
- https://www.netsparker.com.tr/blog/web-guvenligi/http-guvenlik-headerlari/
- https://nullsweep.com/http-security-headers-a-complete-guide/
- https://developer.mozilla.org/en-US/docs/
