# aurora-crypto-core
<div align="center">

```
 █████╗ ██╗   ██╗██████╗  ██████╗ ██████╗  █████╗
██╔══██╗██║   ██║██╔══██╗██╔═══██╗██╔══██╗██╔══██╗
███████║██║   ██║██████╔╝██║   ██║██████╔╝███████║
██╔══██║██║   ██║██╔══██╗██║   ██║██╔══██╗██╔══██║
██║  ██║╚██████╔╝██║  ██║╚██████╔╝██║  ██║██║  ██║
╚═╝  ╚═╝ ╚═════╝ ╚═╝  ╚═╝ ╚═════╝ ╚═╝  ╚═╝╚═╝  ╚═╝
```

**BİLİŞSEL KRİPTOGRAFİK SİSTEM — v4.1**

*Kuantum Dirençli · Sıfır Sunucu · Tam İstemci Tarafı*

---

[![License](https://img.shields.io/badge/Lisans-MIT-00ffcc?style=flat-square&labelColor=0a0a0f)](LICENSE)
[![Standard](https://img.shields.io/badge/Standart-AES--256--GCM-00ffcc?style=flat-square&labelColor=0a0a0f)](https://csrc.nist.gov/publications/detail/fips/197/final)
[![KDF](https://img.shields.io/badge/KDF-PBKDF2--SHA256-ff2d78?style=flat-square&labelColor=0a0a0f)](https://www.rfc-editor.org/rfc/rfc2898)
[![Server](https://img.shields.io/badge/Sunucu-SIFIR-00ffcc?style=flat-square&labelColor=0a0a0f)](#)
[![API](https://img.shields.io/badge/Web%20Crypto%20API-W3C%20Standart-f5c842?style=flat-square&labelColor=0a0a0f)](https://www.w3.org/TR/WebCryptoAPI/)
[![Status](https://img.shields.io/badge/Durum-AKTİF-00ff99?style=flat-square&labelColor=0a0a0f)](#)

</div>

---

## `> GENEL BAKIŞ`

**Aurora**, sıfır sunucu mimarisine dayanan, tamamen tarayıcı tarafında çalışan açık kaynaklı bir kriptografik şifreleme sistemidir. Verileriniz hiçbir zaman bir ağ paketine girmez, hiçbir zaman uzak bir sunucuya dokunmaz, hiçbir zaman üçüncü bir tarafın erişim alanına girmez.

Tüm kriptografik işlemler **W3C Web Crypto API** aracılığıyla işletim sisteminin alt seviyeli güvenli ortamında yürütülür. Oturum tamamlanıp pencere kapandığında, kullanılan anahtarlar ve tüm ara veriler kalıcı olarak RAM'den silinir — geriye sıfır iz kalır.

```
┌─────────────────────────────────────────────────────────────────┐
│  PLAINTEXT  ──►  PBKDF2 (1M iter)  ──►  AES-256-GCM  ──►  BASE64 │
│                      │                        │                  │
│                   128-bit SALT            96-bit IV              │
│                   (CSPRNG)              (CSPRNG)                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## `> ÖZELLİKLER`

| Katman | Teknoloji | Detay |
|--------|-----------|-------|
| **Şifreleme** | AES-256-GCM | 256-bit anahtar, 128-bit GCM kimlik doğrulama etiketi |
| **Anahtar Türetme** | PBKDF2-SHA256 | 1.000.000 iterasyon, çarpraz donanım yavaşlatması |
| **Tuzlama** | CSPRNG | 128-bit kriptografik rastgele tuz, her işlemde yeniden üretilir |
| **Başlangıç Vektörü** | CSPRNG | 96-bit IV, her şifreli paketle benzersiz |
| **API** | Web Crypto API | W3C standardı, donanım destekli sandbox ortamı |
| **Mimari** | Stateless | Sunucu yok, veritabanı yok, bulut yok |
| **Paket Formatı** | BASE64 | `SALT (16B) ‖ IV (12B) ‖ CIPHERTEXT ‖ AUTH_TAG (16B)` |

---

## `> GÜVENLİK MİMARİSİ`

### AES-256-GCM — Şifreleme Çekirdeği

AES-256-GCM, ABD Ulusal Standartlar ve Teknoloji Enstitüsü (NIST FIPS 197) tarafından tanımlanan ve NATO, NSA Suite B gibi kurumsal savunma altyapılarında kullanılan endüstri standardıdır. 256-bit anahtar uzunluğu, teorik kaba kuvvet saldırısı için **2²⁵⁶** kombinasyon gerektirir.

> Gözlemlenebilir evrendeki atom sayısı yaklaşık **2⁸⁰** olarak tahmin edilmektedir. 2²⁵⁶, bunun milyarlarca kez milyarlarca katıdır.

GCM (Galois/Counter Mode), şifrelemenin yanı sıra 128-bit bir kimlik doğrulama etiketi üretir. Bu etiket, şifre çözme anında paketin değiştirilip değiştirilmediğini kriptografik olarak doğrular — aktif ağ saldırıları altında bile veri bütünlüğü garanti altındadır.

### PBKDF2 — Parola Kalkanı

İnsan tarafından seçilen parolalar, doğası gereği entropi açısından zayıftır. PBKDF2 (RFC 2898, NIST SP 800-132), bu zayıflığı telafi etmek için parolayı **1.000.000 kriptografik döngüden** geçirerek türetme maliyetini yapay olarak artırır.

```
Saldırgan kapasitesi (GPU):    ~10,000,000,000 SHA-256/saniye
PBKDF2 iterasyon yükü:         ×1,000,000
────────────────────────────────────────────
Efektif deneme hızı:           ~10,000 parola/saniye

8 karakterli alfanümerik uzay: ~218,340,105,584,896 kombinasyon
Kırma süresi (kaba kuvvet):    ~690 YIL (GPU kümesi ile)
```

### Kriptografik Tuzlama

Her şifreleme işlemi öncesinde `crypto.getRandomValues()` ile 128-bit rastgele bir tuz üretilir. Aynı mesaj, aynı parola ile iki kez şifrelense bile çıktı tamamen farklıdır. Bu yapı, önceden hesaplanmış sözlük saldırılarını (rainbow table) matematiksel olarak geçersiz kılar.

### Sıfır İz Mimarisi

```
┌──────────────────────────────────────────────┐
│  Kullanıcı Tarayıcısı                         │
│  ┌────────────────────────────────────────┐  │
│  │  Web Crypto API (OS Sandbox)           │  │
│  │  • Anahtar türetme                     │  │
│  │  • Şifreleme / Çözme                   │  │
│  │  • non-extractable CryptoKey nesnesi   │  │
│  └────────────────────────────────────────┘  │
│                                               │
│  ← Ağ çağrısı YOK ← Disk yazma YOK ←        │
└──────────────────────────────────────────────┘
                      │
              Sunucu / Bulut
                   [BOŞ]
```

`CryptoKey` nesneleri `extractable: false` ile oluşturulduğundan türetilmiş anahtarlar hiçbir zaman ham byte dizisi olarak JavaScript ortamına sızdırılmaz. Pencere kapatıldığında tüm kriptografik material kalıcı olarak yok edilir.

---

## `> KERCKHOFFs PRENSİBİ`

> *"Bir kriptografik sistemin güvenliği, algoritmanın gizliliğine değil; anahtarın matematiksel gücüne dayanmalıdır."*
> — Auguste Kerckhoffs, 1883

Aurora'nın tüm kaynak kodu bu depoda açık şekilde yayınlanmaktadır. Herhangi bir "gizli mantık" veya "kapalı kutu" barındırmaz. Sistemi inceleyin, denetleyin, doğrulayın. Tek sır — ve tek gerçek güvenlik unsuru — yalnızca sizin zihninizde olan **paroladır**.

---

## `> SALDIRI DİRENÇ MATRİSİ`

| Saldırı Vektörü | Durum | Gerekçe |
|-----------------|-------|---------|
| Kaba Kuvvet (CPU) | ✅ Dirençli | PBKDF2 × 1M iterasyon maliyeti |
| Kaba Kuvvet (GPU) | ✅ Dirençli | Efektif hız ~10K parola/sn'ye düşer |
| Rainbow Table | ✅ İmmün | 128-bit CSPRNG tuz; her işlem benzersiz |
| Man-in-the-Middle | ✅ Dirençli | GCM kimlik doğrulama etiketi bütünlüğü korur |
| Sunucu İhlali | ✅ İmmün | Sunucu yoktur; şifrelenmiş veri sunucuya ulaşmaz |
| Paket Manipülasyonu | ✅ İmmün | GCM auth tag değişimi tespit eder, çözüm başarısız olur |
| Replay Saldırısı | ✅ Dirençli | Her pakette benzersiz IV; tekrar oynatma işe yaramaz |
| Frekans Analizi | ✅ İmmün | Stream şifresi, blok sınırlarını gizler |
| Metadata Sızıntısı | ✅ Sıfır | Ağ çağrısı, log, sunucu kaydı bulunmaz |

---

## `> KULLANIM`

Aurora sıfır bağımlılıkla çalışır. Herhangi bir paket yöneticisi, framework veya kurulum gerektirmez.

### Hızlı Başlangıç

```bash
# Depoyu klonlayın
git clone https://github.com/kullanici-adi/aurora.git

# Proje dizinine girin
cd aurora

# Doğrudan tarayıcıda açın — sunucu gerektirmez
open aurora.html
# veya
start aurora.html   # Windows
xdg-open aurora.html  # Linux
```

> **Not:** Tüm kriptografik işlemler yerel tarayıcınızda gerçekleşir. İnternet bağlantısı yalnızca Google Fonts için gereklidir; gerekirse CSS'ten font importu kaldırılarak tamamen çevrimdışı çalıştırılabilir.

### Şifreleme

```
1. "ŞİFRELE" sekmesini açın
2. Şifrelemek istediğiniz metni girin
3. Güçlü bir parola belirleyin
4. "ŞİFRELE VE AĞA AKTAR" butonuna tıklayın
5. BASE64 paketi otomatik olarak panoya kopyalanır
```

### Deşifre

```
1. "DEŞİFRE ET" sekmesini açın
2. BASE64 paketini yapıştırın
3. Şifreleme sırasında kullanılan parolayı girin
4. "PAKETİ ÇÖZ VE DOĞRULA" butonuna tıklayın
```

---

## `> TEKNİK SPESIFIKASYONLAR`

```yaml
Şifreleme Algoritması:   AES-256-GCM
Anahtar Uzunluğu:        256 bit
GCM Kimlik Doğrulama:    128 bit auth tag
KDF:                     PBKDF2
KDF Hash:                SHA-256
KDF İterasyon:           1,000,000
Tuz Uzunluğu:            128 bit (16 byte) — CSPRNG
IV Uzunluğu:             96 bit (12 byte) — CSPRNG
Paket Formatı:           BASE64(SALT[16] || IV[12] || CIPHER || TAG[16])
API:                     Web Crypto API (W3C)
Anahtar Tipi:            non-extractable CryptoKey
Çalışma Ortamı:          Client-Side Only (Stateless)
Bağımlılık:              Sıfır (Zero Dependencies)
Sunucu Çağrısı:          Sıfır (Zero Server Calls)
Veri Kalıcılığı:         Sıfır (RAM-Only, Session-Scoped)
```

---

## `> TARAYICI DESTEĞİ`

Web Crypto API, modern tüm tarayıcılarda yerleşik olarak desteklenmektedir.

| Tarayıcı | Minimum Sürüm | Durum |
|----------|---------------|-------|
| Chrome | 37+ | ✅ Tam Destek |
| Firefox | 34+ | ✅ Tam Destek |
| Safari | 10.1+ | ✅ Tam Destek |
| Edge | 12+ | ✅ Tam Destek |
| Opera | 24+ | ✅ Tam Destek |

> ⚠️ Web Crypto API yalnızca **HTTPS** veya `localhost` ortamında çalışır. `http://` üzerinde deploy etmeyin.

---

## `> PROJE YAPISI`

```
aurora/
├── aurora.html          # Tek dosya — tüm sistem burada
├── README.md            # Bu dosya
└── LICENSE              # MIT Lisansı
```

Aurora kasıtlı olarak **tek dosya** mimarisinde tasarlanmıştır. Harici CSS, JS veya asset dosyası yoktur. Tüm mantık, stil ve kriptografi `aurora.html` içinde konumlandırılmıştır; böylece kaynak kodu denetimi tek bir bakışta tamamlanabilir.

---

## `> ÖNEMLI UYARILAR`

```
⚠  PAROLA YÖNETİMİ
   Aurora, parola kurtarma mekanizması içermez.
   Parolayı kaybetmek = veriyi kalıcı olarak kaybetmek.
   Güçlü, hatırlanabilir bir parola seçin.

⚠  ORTAM GÜVENLİĞİ
   Kötü amaçlı yazılım bulaşmış bir tarayıcı bu sistemi de
   tehlikeye atabilir. Güvenilir bir cihaz kullanın.

⚠  YAN KANAL SALDIRISI
   Aurora ağ saldırılarına karşı dirençlidir ancak fiziksel
   erişim (keylogger, ekran izleme) kapsamı dışındadır.

⚠  KUANTUM HESAPLAMA
   AES-256, Grover algoritması göz önüne alındığında etkin
   güvenlik seviyesi 128-bit'e düşer. Bu, öngörülebilir
   gelecek için hâlâ güvenli kabul edilmektedir.
```

---

## `> KAYNAKLAR VE STANDARTLAR`

- [NIST FIPS 197](https://csrc.nist.gov/publications/detail/fips/197/final) — AES Standardı
- [NIST SP 800-38D](https://csrc.nist.gov/publications/detail/sp/800-38d/final) — GCM Modu
- [NIST SP 800-132](https://csrc.nist.gov/publications/detail/sp/800-132/final) — PBKDF Önerisi
- [RFC 2898](https://www.rfc-editor.org/rfc/rfc2898) — PBKDF2 Spesifikasyonu
- [W3C Web Crypto API](https://www.w3.org/TR/WebCryptoAPI/) — Tarayıcı Kriptografi Standardı
- [Kerckhoffs Prensibi](https://en.wikipedia.org/wiki/Kerckhoffs%27s_principle) — Güvenlik Felsefesi

---

## `> LİSANS`

```
MIT License — Copyright (c) 2025 Aurora Project

Kaynak kodu inceleme, kopyalama, değiştirme ve dağıtma
hakları MIT lisansı çerçevesinde serbesttir.
```

---

<div align="center">

```
// AURORA — BİLİŞSEL KRİPTOGRAFİK SİSTEM //
// CLIENT-SIDE ONLY — ZERO TRACE — ZERO SERVER //
```

*Gizlilik bir ayrıcalık değil, temel bir haktır.*

</div>
