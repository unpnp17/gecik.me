# Değişiklik kaydı

Biçim [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), sürümleme [Semantic Versioning](https://semver.org/) esas alır.

## [0.1.0] - 2026-09-27

### Eklendi
- Beş bağımsız ölçüm slotu; IP, FQDN veya tam URL girilebilir.
- İki ölçüm yöntemi: `https` (fetch + Resource Timing API) ve `görsel` (`<img>` yedeği).
- Canlı zaman serisi grafiği, imleçle anlık değer okuma, logaritmik ölçek seçeneği.
- Slot başına son / ort / min / maks / jitter / kayıp istatistikleri ve son 60 örnek şeridi.
- Ölçüm aralığı: 250 ms – 60 s.
- Zaman aşımı: 1, 2, 5, 15, 30 s.
- Pencere: 30 s – 6 saat. Uzun pencerelerde piksel sütunu başına ortalama çizgisi ve min–maks bandı.
- CSV dışa aktarma.

[0.1.0]: https://github.com/unpnp17/gecik.me/releases/tag/v0.1.0
