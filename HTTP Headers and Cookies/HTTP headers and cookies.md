# HTTP SECURITY HEADERS AND COOKIE FLAGS

*---Todo---*

Bunların neler olduğuna geçmeden önce açıklanacak bazı zafiyetler.
#### XSS 
Cross Site Scripting (XSS), saldırganın kurbanın tarayıcısında keyfi JavaScript kodları çalıştırmasına izin veren bir güvenlik açığıdır.

#### Clickjacking
*----------*

## HTTP SECURITY HEADERS

HTTP başlıkları, HTTP istek ve yanıtlarında yer alır ve bazı ek veriler içerir. Bu veriler sunucu/istemciye ait bilgiler, mesajın boyutu, türü gibi bilgiler veya çerezler olabilir. Burada anlatılanlar ise bazı zafiyetleri engellemeye yardımcı olacak güvenlik başlıklarıdır.

### X-XSS-PROTECTION

Muhtemel XSS payloadlarını tespit edip filtreleyerek Reflected XSS saldırılarına karşı koruma sağlar. 
Stored XSS karşısında çaresizdir!

**Parametreleri**
- 0:
Filtrelemeyi devre dışı bırakır. 
- 1:
Filtrelemeyi etkinleştirir.
- mode=block:
XSS tespiti halinde sayfanın yüklenmesini engeller. 
- report=https://website.com/xss.log:
XSS tespiti halinde saldırı girişimini belirtilen siteye raporlar. Sadece Chromium tabanlı tarayıcılarda kullanılır. 

`X-XSS-Protection: 1; mode=block; report=https://website.com/xss.log`


### X-Frame-Options

Sayfanın bir iframe içerisinde çağırılma durumunu kontrol eder. Clickjacking saldırılarına karşı koruma sağlar.

**Parametreleri**
- DENY:
Hiçbir şekilde iframe içerisinde kullanılmasına izin verilmez.
- SAMEORIGIN:
Yalnızca aynı domain tarafından kullanılmasına izin verir.
- ALLOW-FROM URL:
Yalnızca belirtilen URL tarafından kullanılmasına izin verir. Güncel tarayıcılarda desteklenmemektedir!

`X-Frame-Options: SAMEORIGIN`


### X-Content-Type-Options
### Content-Security-Policy (CSP)
### HSTS (HTTP Strict Transport Security)


## COOKIE FLAGS

HTTP çerezleri (cookies), kullanıcının tarayıcısında depolanan küçük verilerdir. Genellikle oturum takibi yapmak ve kullanıcının tercihlerini hatırlamak için kullanılır. Bu çerezlerin güvenliğini sağlamak için çerezlere eklenmesi gereken bazı özellikler vardır.

### HttpOnly

HttpOnly olarak ayarlanmış bir çerez yalnızca sunucuya gönderilir, JavaScript tarafından (document.cookie) erişilemez. Session_id gibi önemli verilerin XSS saldırısıyla ele geçirilmesini engeller.

### Secure

Çerezlerin yalnızca HTTPS taleplerinde sunucuya gönderilmesini sağlar. HTTPS taleplerinde veriler şifreli olarak gönderildiği için ağı dinleyen saldırganın çerezleri ele geçirmesi önlenir.

`Set-Cookie: sessionid=xAeeEjJyvOA0eCaD1qUFmCT3cfYxc9Tp8q1bonFtoVB10pDCx5GC6iw9I5nY8FXNAWZHEA; HttpOnly; Secure`


##### REFERANSLAR

- https://www.mehmetince.net/http-security-headerlari-neden-ve-nasil-kullanilmalidir/
- https://www.netsparker.com.tr/blog/web-guvenligi/http-guvenlik-headerlari/
- https://nullsweep.com/http-security-headers-a-complete-guide/
- https://developer.mozilla.org/en-US/docs/
