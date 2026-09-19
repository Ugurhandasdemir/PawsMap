# PawsMap — Kampüs Hayvan Yönetim Sistemi

Kampüsteki sokak hayvanları için kayıp/sahiplendirme/medikal yardım ilanlarını ve besleme istasyonlarını tek bir harita üzerinde toplayan Django web uygulaması. Öğrenci ve gönüllüler ilan açar, yönetici ekip ilanları onaylar ve besleme istasyonlarının doluluk durumunu izler.

**Canlı demo:** https://kampus-hayvan-yonetim-sistemi.vercel.app

## Özellikler

- **Etkileşimli harita:** Leaflet + OpenStreetMap üzerinde ilanlar ve besleme istasyonları. Konum, adres araması ile de bulunabilir.
- **Üç ilan türü:** Kayıp, Sahiplendirme, Medikal Yardım. Her ilanda fotoğraf, açıklama, iletişim bilgisi ve koordinat bulunur; yüklenen fotoğraflar sunucuda küçültülür.
- **Onay akışı:** Yeni ilanlar `Beklemede` durumunda açılır; yönetici onaylayınca yayına girer. İlan sahibi hayvan bulunduğunda ilanı "bulundu" olarak işaretleyebilir.
- **Besleme istasyonları:** Mama ve su seviyesi (0–100) tutulur; %25 ve altı "kritik" sayılır. İstasyonlar `/api/stations/` ucundan JSON olarak da alınabilir.
- **Gönüllü başvuruları:** Beceri ve müsaitlik bilgisiyle başvuru, yönetici onay/red akışı.
- **Rol tabanlı kullanıcılar:** Öğrenci, Personel, Gönüllü profilleri; e-posta ile giriş.
- **Özel yönetim paneli:** Dashboard, ilanlar, gönüllüler, istasyonlar ve kullanıcılar için ayrı sayfalar (`/admin-panel/`, yalnızca `is_staff`).

## Teknoloji Yığını

| Katman | Teknoloji |
| --- | --- |
| Backend | Python 3.12, Django 5.x |
| Veritabanı | SQLite (yerel), PostgreSQL (`DATABASE_URL` ile, `psycopg`) |
| Statik dosyalar | WhiteNoise |
| Medya | Yerel `media/`, `CLOUDINARY_URL` tanımlıysa Cloudinary |
| Harita | Leaflet 1.9.4, OpenStreetMap |
| Yayın | Vercel (`vercel.json`, `build_files.sh`) |

## Kurulum

```bash
git clone https://github.com/Ugurhandasdemir/PawsMap.git
cd PawsMap
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt

export SECRET_KEY="$(python -c 'import secrets; print(secrets.token_urlsafe(50))')"   # yerelde isteğe bağlı
python manage.py migrate
python manage.py createsuperuser
python manage.py runserver
```

Uygulama `http://127.0.0.1:8000` adresinde açılır.

### Ortam değişkenleri

Ayarlar doğrudan işletim sistemi ortam değişkenlerinden okunur (`.env` dosyası otomatik yüklenmez).

| Değişken | Açıklama | Zorunlu |
| --- | --- | --- |
| `SECRET_KEY` | Django gizli anahtarı. Üretimde mutlaka kendi değerinizi verin | Üretimde evet |
| `DEBUG` | `True` / `False`. Vercel'de varsayılan `False` | Hayır |
| `ALLOWED_HOSTS` | Virgülle ayrılmış ek host listesi | Hayır |
| `CSRF_TRUSTED_ORIGINS` | Virgülle ayrılmış güvenilir origin listesi | Hayır |
| `DATABASE_URL` | PostgreSQL bağlantı adresi; yoksa SQLite kullanılır | Hayır |
| `CLOUDINARY_URL` | Tanımlıysa yüklenen fotoğraflar Cloudinary'ye gider | Hayır |

## Proje Yapısı

```
PawsMap/
├── config/          # Django ayarları, kök URL, WSGI/ASGI
├── core/            # models (AnimalReport, FeedingStation, VolunteerApplication, UserProfile), views, urls
├── templates/       # sayfa şablonları; admin_panel/ altında yönetim ekranları
├── static/css/      # sayfa bazlı stiller
├── build_files.sh   # Vercel build: collectstatic + migrate
└── vercel.json
```

## Yayına Alma (Vercel)

1. `SECRET_KEY`, `DATABASE_URL` (PostgreSQL) ve `CLOUDINARY_URL` değişkenlerini Vercel proje ayarlarına ekleyin. Vercel dosya sistemi kalıcı olmadığı için SQLite ve yerel `media/` üretimde kullanılamaz.
2. Repoyu Vercel'e bağlayın; `vercel.json` ve `build_files.sh` derleme ve yönlendirmeyi yapar.

## Bilinen Eksikler

- Otomatik test yok (`core/tests.py` boş).
- `src/` altındaki React/Vite dosyaları şablon kalıntısıdır; uygulama Django şablonlarıyla çalışır.
