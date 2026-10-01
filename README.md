# Nuclei Şablonları — Güvenlik Tespit Koleksiyonu

⚠️ **Sorumluluk Reddi**: Bu depodaki içerik yalnızca eğitim ve bilgilendirme amaçlıdır; yazarlar kötüye kullanımdan sorumlu değildir. Kullanmadan önce gerekli yetkiyi aldığınızdan emin olun, kendi sorumluluğunuzda sorumlu davranın ve tüm yasal/etik kurallara uyun. 🚀

Bu koleksiyon, yetkili sızma testleri ve savunma amaçlı güvenlik denetimleri için hazırlanmış [Nuclei](https://github.com/projectdiscovery/nuclei) şablonlarından oluşur. Kök dizindeki **36 kürlenmiş şablonun tümü** `nuclei -validate` ile doğrulanmıştır, çift `id` içermez ve **hiçbiri `-dast` bayrağı gerektirmez** (varsayılan taramada çalışır). Ayrıca resmî ProjectDiscovery seti (**14.000+ şablon**) `official/` alt dizinine klonlanmıştır — bkz. **Resmî Template Seti**.

> **Kalite**: Tüm koleksiyon, yerel bir "kırılgan mock" sunucuya karşı test edilip her tespitin **ateşlendiği** doğrulanmış; ayrıca tamamen temiz/sertleştirilmiş bir sunucuda **sıfır false-positive** ürettiği ölçülmüştür. Bu, her yola `200` + temiz HTML dönen (SPA/özel 404) sitelerde bile gürültüsüz tarama demektir. Ayrıntılar için aşağıdaki **Test & Doğrulama** bölümüne bakın.

## Kullanım

```bash
# Tek bir şablon
nuclei -t credentials-disclosure-all.yaml -u https://hedef.example.com

# SADECE kürlenmiş hızlı set (resmî seti hariç tut) — önerilen toplu tarama
cat siteler.txt | nuclei -t . -et official/ -nc -silent

# SADECE resmî set (14.000+ şablon)
cat siteler.txt | nuclei -t official/ -nc -silent

# HER ŞEY birlikte (kürlenmiş + resmî) — en kapsamlı, en yavaş
cat siteler.txt | nuclei -t . -nc -silent
```

> **Önemli:** `official/` alt dizini olduğundan, `nuclei -t .` artık 14.000+ resmî şablonu da çalıştırır (çok yavaş). Alışık olduğun hızlı toplu taramayı sürdürmek için **`-et official/`** (exclude) ekle.

## Şablonlar

| Dosya | Tespit | Önem |
|-------|--------|------|
| `credentials-disclosure-all.yaml` | Sızdırılmış kimlik bilgisi/gizli anahtar/token — İngilizce **ve Türkçe** alan adları (kullanıcı, şifre, parola…) + sağlayıcı token'ları (GitHub, GitLab, OpenAI, Google, Stripe, AWS, JWT…) | orta |
| `pii-disclosure.yaml` | Kişisel veri (KVKK/GDPR): **T.C. Kimlik No**, TR/genel IBAN, telefon, kredi kartı, e-posta | yüksek |
| `backup-files.yaml` | **Birleşik yedek tespiti**: arşiv/sıkıştırılmış (zip/tar/rar/7z/sql) + PHP kaynak/config yedekleri + **Türkçe adlar** (yedek, veritabani…) | orta |
| `env-file-exposure.yaml` | `.env` / ortam değişkeni dosyası ifşası | yüksek |
| `git-exposure.yaml` | `.git` dizini ifşası (kaynak kod sızıntısı) | orta |
| `aws-access-secret-key.yaml` | AWS erişim/gizli anahtar ifşası (**değer biçimi doğrulamalı**, `EXAMPLE` elenir) | yüksek |
| `s3-detect.yaml` | Amazon S3 bucket tespiti | bilgi |
| `errsqli.yaml` | Hata tabanlı SQL enjeksiyonu (40+ veritabanı, 31 parametre + Türkçe parametreler, **regex hataları düzeltildi**, `<EOF>` FP'si kaldırıldı, Türkçe DB hata imzaları) | yüksek |
| `response-ssrf.yaml` | Tam yanıt (in-band) SSRF + dosya okuma (38 parametre + Türkçe, AWS/GCP/Azure/Aliyun/Tencent metadata, OOB interactsh) | yüksek |
| `openRedirect.yaml` | GET tabanlı açık yönlendirme (**regex matcher** — `//bing.com`, `\bing.com`, unicode, meta-refresh/JS dahil) | orta |
| `cRlf.yaml` | CRLF enjeksiyonu / yanıt bölme (parametre tabanlı vektörler eklendi, `%%0a0a` hatası düzeltildi) | yüksek |
| `lfi-path-traversal.yaml` | **YENİ** — LFI / dizin aşımı (/etc/passwd, win.ini, php://filter) + Türkçe parametreler | yüksek |
| `xss-reflected.yaml` | **YENİ** — Yansıyan XSS (benzersiz kanarya, yalnızca kodlanmamış yansımada ateşler — düşük FP) | yüksek |
| `ssti-detection.yaml` | **YENİ** — Server-Side Template Injection (çoklu motor; 1337×1337 değerlendirme doğrulaması) | yüksek |
| `sensitive-files.yaml` | **YENİ** — Hassas dosya ifşası (.svn/.hg, .htpasswd, .npmrc, .aws/credentials, Dockerfile, CI, IDE) — içerik imzası doğrulamalı | yüksek |
| `wordpress-user-enum.yaml` | **YENİ** — WordPress kullanıcı sayımı (REST `/wp-json/wp/v2/users` + `/?author=1`) | orta |
| `cors.yaml` | CORS yanlış yapılandırması (çoklu teknik: arbitrary origin, null, pre/post-domain, protokol düşürme, credentials'sız yansıma) | yüksek |
| `x-forwarded.yaml` | X-Forwarded başlık yansıması (benzersiz kanarya ile) | orta |
| `put-method-enabled.yaml` | HTTP PUT metodu etkin | yüksek |
| `CVE-2025-29927.yaml` | Next.js middleware atlatma (CVE-2025-29927) | kritik |
| `CVE-2025-11072.yaml` | **YENİ** — WordPress Download Counter Button ≤1.8.6.7 kimlik doğrulamasız keyfi dosya okuma (gerçek, doğrulanmış CVE) | yüksek |
| `next-js.yaml` | Next.js middleware atlatma (payload varyantı) | kritik |
| `nextjs-middleware-cache.yaml` | Next.js önbellek zehirlenmesi | yüksek |
| `detect-all-takeovers.yaml` | Alt alan adı ele geçirme | yüksek |
| `wordpress-takeover.yaml` | WordPress ele geçirme | yüksek |
| `wp-setup-config.yaml` | WordPress kurulum yapılandırması açığa çıkması | kritik |
| `admin-login-panel.yaml` | Yönetici/giriş paneli tespiti (İngilizce **+ Türkçe** yollar) | bilgi |
| `application-error-disclosure.yaml` | Uygulama hatası / yığın izi ifşası (PHP/.NET/Java/Python + **Türkçe**) | düşük |
| `phpinfo-exposure.yaml` | phpinfo() bilgi sayfası ifşası | düşük |
| `directory-listing.yaml` | Dizin listeleme (Index of /) açık | düşük |
| `security-headers-missing.yaml` | Eksik güvenlik başlıkları (CSP, HSTS, X-Frame-Options, Permissions-Policy…) — **tek bulgu + eksik liste**, yalnızca gerçek HTML 200'de | bilgi |
| `api_endpoints.yaml` | Genişletilmiş API keşfi: REST/GraphQL, OAuth, Swagger/OpenAPI, Spring Actuator, /.well-known + **Türkçe uçlar** | bilgi |
| `graphql_get.yaml` | GraphQL uç noktası tespiti | bilgi |
| `Swagger.yaml` | Swagger UI configUrl enjeksiyonu | düşük |
| `iis.yaml` | IIS kısa ad (8.3) numaralandırması | düşük |
| `cloudflare-rocketloader-htmli.yaml` | Cloudflare Rocket Loader HTML enjeksiyonu | — |

## Resmî Template Seti (`official/`)

`official/` dizini, [ProjectDiscovery nuclei-templates](https://github.com/projectdiscovery/nuclei-templates) deposunun sığ (shallow) klonudur — 14.000+ bakımlı, test edilmiş şablon (binlerce CVE dahil). Güncellemek için:

```bash
cd official && git pull --depth 1    # veya: nuclei -update-templates (ayrı dizine)
```

Kürlenmiş hızlı setini korumak için toplu taramalarda `-et official/` kullanmayı unutma (yukarıdaki **Kullanım**).

### Güncel CVE'ler ve doğruluk notu

Elle CVE şablonu eklerken **gerçek/doğrulanmış** teknik detay şarttır; uydurma yol/payload çalışmayan (ve yanıltıcı) bir tespit üretir. Örnek: kullanıcı tarafından "WordPress CVE-2026-41940" olarak anılan zafiyet aslında bir **cPanel/WHM kimlik doğrulama atlatmasıdır** (CVSS 9.8) — WordPress ile ilgisi yoktur ve güvenilir kamuya açık bir HTTP imzası bulunmadığından bu sete eklenmemiştir. Buna karşılık `CVE-2025-11072.yaml`, resmî PoC'tan doğrulanmış gerçek bir WordPress CVE'si olarak eklenmiş ve test edilmiştir. Binlerce güncel CVE için `official/` setini kullan.

## Türkçe Destek

Türkçe tespitler ayrı dosyalar hâlinde değil, ilgili genel şablonların **içine gömülüdür**:

- **Kimlik bilgileri** (`credentials-disclosure-all.yaml`): `şifre`/`sifre`, `parola`, `kullanıcı adı`, `veritabanı şifresi`, `api anahtarı`, `erişim anahtarı`, `oturum anahtarı` — hem Türkçe karakterli hem ASCII yazımıyla.
- **Kişisel veri** (`pii-disclosure.yaml`): T.C. Kimlik No, TR IBAN, Türk telefon numaraları.
- **Yedek dosyalar** (`backup-files.yaml`): `yedek`, `yedekleme`, `siteyedek`, `veritabani`, `db_yedek` adları.
- **Paneller** (`admin-login-panel.yaml`): `/yonetim`, `/yonetici`, `/giris`, `/kontrol-paneli`, `/uye-girisi` yolları.
- **Hata sayfaları** (`application-error-disclosure.yaml`): `Ölümcül hata`, `Sunucu Hatası`, `Veritabanı bağlantı hatası`.
- **API uçları** (`api_endpoints.yaml`): `/api/kullanicilar`, `/api/siparisler`, `/api/uyeler`.
- **SQL enjeksiyonu** (`errsqli.yaml`): `urun`, `kategori`, `haber`, `sayfa`, `kullanici`, `ara`, `aranan` parametreleri + Türkçe DB hata metinleri (`SQL sözdizimi hatası`, `veritabanı sorgusu başarısız`).
- **LFI** (`lfi-path-traversal.yaml`): `dosya`, `sayfa`, `belge`, `icerik`, `indir` parametreleri.
- **XSS** (`xss-reflected.yaml`): `ara`, `aranan`, `anahtar`, `mesaj` parametreleri.
- **SSTI** (`ssti-detection.yaml`): `ara`, `aranan`, `isim`, `mesaj` parametreleri.
- **SSRF** (`response-ssrf.yaml`): `adres`, `baglanti`, `kaynak`, `yonlendir` parametreleri.

> Not: Go `regexp` motoru `(?i)` ile Türkçe'ye özgü harf katlamayı (ı/İ) yapmaz; bu yüzden tespitler hem Türkçe karakterli (`şifre`) hem ASCII (`sifre`) yazımı **ayrı ayrı** içerir.

## Test & Doğrulama

Tüm koleksiyon iki yönlü test edilmiştir:

1. **Pozitif test (kırılgan mock sunucu)** — Her template'in, ilgili zafiyeti barındıran bir yanıta karşı **ateşlendiği** doğrulandı (SQLi, LFI, XSS, SSTI, CRLF, CORS, SSRF göstergeleri, açığa çıkan dosyalar vb.).
2. **False-positive testi (temiz/sertleştirilmiş sunucu)** — Her yola `200` + temiz HTML dönen, tüm güvenlik başlıklarını gönderen ve yansımaları HTML-encode eden bir sunucuda tüm koleksiyon çalıştırıldı; sonuç: **0 false-positive**. Bu, SPA/özel-404 nedeniyle her yola 200 dönen sitelerde bile gürültüsüz tarama anlamına gelir.

Yeniden üretmek için (`scratchpad` altındaki test sunucuları örnektir):

```bash
nuclei -t . -validate                       # sözdizimi/motor doğrulaması
nuclei -t . -u http://127.0.0.1:8899 -nc    # pozitif (kırılgan hedef)
nuclei -t . -u http://127.0.0.1:8898 -nc    # FP kontrolü (temiz hedef → boş olmalı)
```
