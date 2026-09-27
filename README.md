# gecik.me

Tarayıcıdan birden fazla hedefe (Google, Amazon, Apple, Cloudflare ya da kendi sunucun) HTTPS istekleri atıp gidiş-dönüş sürelerini canlı grafiğe döken, PingPlotter benzeri tek dosyalık bir gecikme izleyicisi.

Kurulum, sunucu ya da bağımlılık yok: `index.html` dosyasını tarayıcıda aç, **Ölçümü başlat**'a bas.

## Ne ölçer, ne ölçmez

Tarayıcı JavaScript'i ICMP paketi gönderemez, bu yüzden gecik.me `ping` değil **HTTPS RTT** ölçer:

```
ölçülen süre = DNS + TCP + TLS + HTTP isteği + sunucunun yanıt üretme süresi
```

- İlk örnek el sıkışma (DNS, TCP, TLS) yüzünden 2–4 kat yüksek çıkar. Sonraki istekler keep-alive bağlantı üzerinden gider ve saf ağ RTT'sine çok yaklaşır.
- Sunucunun işlem süresi de sonuca dahildir. Küçük, statik yanıt dönen uç noktalar (`/generate_204`, `/favicon.ico`, `/cdn-cgi/trace`) bu payı en aza indirir.
- Hop-by-hop (traceroute) görünümü tarayıcıdan mümkün değildir.

Gerçek ICMP ya da hop bazlı ölçüm gerekiyorsa bu araç yerine `mtr`, `ping` veya PingPlotter kullanılmalı.

## Hızlı başlangıç

| Yöntem | Nasıl | Not |
|---|---|---|
| Doğrudan dosya | `index.html`'e çift tıkla | En hızlısı; düz `http://` hedefler de çalışır |
| Yerel sunucu | `python3 -m http.server 8080` → `http://localhost:8080` | Dosya erişimini kısıtlayan tarayıcı politikaları varsa |
| GitHub Pages | Repo *Settings → Pages → Deploy from branch → `main` / root* | Sayfa HTTPS'ten gelir; düz `http://` hedefler mixed-content olarak engellenir |

## Kullanım

Beş slot var, her biri bağımsız ölçülür. Hedef alanına şunlardan birini yaz:

- **Tam URL** (`https://www.google.com/generate_204`): aynen kullanılır.
- **Sadece host veya IP** (`1.1.1.1`, `example.com`): `https://<host>/` denenir.

Değişiklik alandan çıkınca ya da Enter'a basınca uygulanır ve o slotun geçmişi sıfırlanır; ölçüm durmuşsa Enter ölçümü de başlatır. Soldaki kutu slotu ölçümden çıkarır.

### Yöntemler

| Yöntem | Mekanizma | Ne zaman |
|---|---|---|
| `https` | `fetch(mode:'no-cors', cache:'no-store')`; süre Resource Timing API'den okunur | Varsayılan, en doğru yöntem |
| `görsel` | `<img>` yükleme denemesi; `onload` ve `onerror` ikisi de yanıt sayılır | Hedefin CSP/CORS yapısı fetch'i engelliyorsa yedek. DNS hatası da yanıt gibi görünebileceği için dikkatli yorumla |

Her isteğe benzersiz bir `_lp=` query parametresi eklenir; tarayıcı ve ara katman önbellekleri devre dışı kalır.

### Kontroller

| Kontrol | Seçenekler | Açıklama |
|---|---|---|
| aralık | 250 ms, 500 ms, 1 s, 2 s, 5 s, 15 s, 60 s | İki ölçüm arası süre, slot başına |
| zaman aşımı | 1, 2, 5, 15, 30 s | Bu süre dolarsa örnek kayıp sayılır |
| pencere | 30 s, 60 s, 3 dk, 10 dk, 30 dk, 60 dk, 3 sa, 6 sa | Grafikte ve tabloda gösterilen zaman aralığı |
| log | açık / kapalı | Y eksenini logaritmik yapar; büyük spike'lar düşük değerleri ezmesin diye |
| CSV indir | | Bellekteki tüm örnekleri dışa aktarır |

Ölçüm sıralıdır: bir slotun yanıtı gelmeden (ya da zaman aşımı dolmadan) o slottan yeni istek çıkmaz. Aralığı zaman aşımının yarısından kısa tutarsan, kayıp anlarında efektif aralık uzar.

**Uzun süreli izleme önerisi:** 6 saatlik pencere için aralık 5–15 s yeterli. 1 s aralıkla 6 saat, hedef başına 21.600 istek demektir.

## Metrikler

Tablodaki tüm değerler **seçili pencerenin tamamı** üzerinden hesaplanır, milisaniye cinsindendir.

| Sütun | Tanım |
|---|---|
| son | En son örnek; zaman aşımına uğradıysa `kayıp` |
| ort | Başarılı örneklerin aritmetik ortalaması |
| min / maks | Başarılı örneklerin en düşük ve en yüksek değeri |
| jitter | Ardışık başarılı örnekler arasındaki farkların mutlak ortalaması |
| kayıp | Zaman aşımına uğrayan örneklerin yüzdesi |
| son 60 örnek | Bar şeridi; kırmızı çubuk kayıp |

### Grafiği okumak

- 10 dakikadan uzun pencerelerde bir piksel sütununa birden fazla örnek düşer. **Çizgi** o sütunun ortalamasını, arkasındaki **soluk bant** min–maks aralığını gösterir. Böylece ortalamada kaybolacak tekil spike'lar bandın tepesinde görünür kalır.
- Alt kenardaki **kırmızı çizikler** kayıp örnekleri işaretler.
- İmleci grafiğin üzerinde gezdirince o anın saati ve her hedefin değeri (uzun pencerede `ortalama (min–maks)` ve kayıp sayısı) görünür.

## Önerilen hedefler

| Servis | Uç nokta | Neden |
|---|---|---|
| Google | `https://www.google.com/generate_204` | Gövdesiz 204 yanıtı, en düşük sunucu payı |
| Cloudflare | `https://1.1.1.1/cdn-cgi/trace` | Anycast, küçük düz metin yanıt |
| Apple | `https://captive.apple.com/hotspot-detect.html` | Captive portal kontrolü için tasarlanmış, çok küçük |
| Amazon | `https://www.amazon.com/favicon.ico` | Statik, CDN'den |
| Kendi sunucun | `https://sunucu.ornek.com/health` | En küçük yanıtı dönen endpoint'i seç |

Yönlendirme (redirect) yapan adresler iki gidiş-dönüş ölçer; `apple.com` gibi alan adı kökleri yerine doğrudan uç noktayı kullan.

## CSV formatı

```csv
timestamp_iso,slot,target,rtt_ms
2026-09-27T14:03:11.482Z,1,"https://www.google.com/generate_204",18.40
2026-09-27T14:03:12.481Z,1,"https://www.google.com/generate_204",
```

Boş `rtt_ms` kayıp demektir. Zaman damgası UTC'dir.

## Veri ve gizlilik

- Tüm veriler yalnızca tarayıcı belleğinde tutulur; sayfa yenilenince silinir. Hiçbir yere gönderilmez.
- Tarayıcı sekmesi arka plana alındığında zamanlayıcılar kısıtlanabilir; uzun ölçümlerde sekmeyi ön planda ya da ayrı bir pencerede tut.
- Bellekte hedef başına en fazla 6,5 saatlik (üst sınır 45.000) örnek tutulur.

## Geliştirme

Tüm uygulama tek dosyadır: `index.html` (HTML + CSS + JavaScript, harici bağımlılık yok).

Her push ve pull request'te GitHub Actions, dosyadaki `<script>` bloğunu ayıklayıp `node --check` ile sözdizimi kontrolü yapar ([`.github/workflows/check.yml`](.github/workflows/check.yml)). Aynı kontrolü yerelde çalıştırmak için:

```bash
python3 - <<'PY' > /tmp/gecik-check.js
import re; s = open('index.html', encoding='utf-8').read()
print(re.search(r'<script>(.*)</script>', s, re.S).group(1))
PY
node --check /tmp/gecik-check.js && echo OK
```

Sürümler `vMAJOR.MINOR.PATCH` biçiminde etiketlenir; sürüm numarası sayfa başlığında da görünür. Değişiklikler [`CHANGELOG.md`](CHANGELOG.md) dosyasında tutulur.

## Lisans

[MIT](LICENSE) © 2026 Mustafa Öztürk
