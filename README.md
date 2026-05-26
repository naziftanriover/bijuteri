# Kuyumcu / Bijuteri POS Sistemi

Tarayıcıda çalışan, PWA destekli, Firebase ile gerçek zamanlı senkron çalışan bijuteri / kuyumcu satış noktası (POS) uygulaması. Firebase yapılandırması girilmezse otomatik olarak yerel depolama (localStorage) modunda çalışır.

## Özellikler

- Yönetim ve kasiyer rolleri (SHA-256 hash'li şifreler)
- Kasa (POS): barkod / arama, sepet, nakit / kart / veresiye
- Stok yönetimi: kategoriler, min stok uyarıları
- Müşteri & veresiye takibi, tahsilat
- Raporlar: 4 grafik (trend, ödeme tipi, kategori, kasiyer)
- Ayarlardan tema rengi, mağaza adı, şifreler, Firebase config
- PWA: offline çalışır, ana ekrana eklenebilir
- Dışa aktarım (JSON), demo veri, sıfırlama

## Kurulum (Yerel)

1. Bu klasördeki tüm dosyaları bir web sunucusunda yayınlayın. Test için:
   ```bash
   cd BIJUTERI
   python3 -m http.server 8080
   ```
2. Tarayıcıdan `http://localhost:8080` adresine girin.
3. İlk girişte **Yönetim** seçip varsayılan şifre `1234` ile girin.
4. Ayarlardan mağaza adı, tema rengi ve şifreleri değiştirin.

> Önemli: Service Worker ve PWA özellikleri `https://` veya `localhost` üzerinden çalışır, `file://` ile açtığınızda çalışmaz.

## Varsayılan Şifreler

| Rol | Şifre |
|-----|-------|
| Yönetim | `1234` |
| Kasiyer 1 | `1111` |
| Kasiyer 2 | `2222` |
| Kasiyer 3 | `3333` |

İlk girişten sonra Ayarlar > Şifre Yönetimi bölümünden mutlaka değiştirin.

---

## Firebase Kurulum Rehberi (Adım Adım)

Firebase bağlanırsa tüm cihazlarda **gerçek zamanlı** veri senkronu sağlanır. Bağlanmazsa uygulama yerel modda çalışmaya devam eder.

### 1. Firebase Projesi Oluşturma

1. https://console.firebase.google.com adresine girin.
2. Sağ üstten **"Proje ekle"** butonuna tıklayın.
3. Proje adınızı yazın (örn. `kuyumcu-pos`) ve **Devam**'a tıklayın.
4. Google Analytics istemiyorsanız kapatın, **Proje oluştur**'a tıklayın.
5. Proje hazırlanınca **Devam**'a basın.

### 2. Web Uygulaması Ekleme

1. Proje ana sayfasında **`</>`** (Web) ikonuna tıklayın.
2. Uygulamaya bir takma ad verin (örn. `pos-web`).
3. **Firebase Hosting**'i şimdilik seçmeyin, **Uygulamayı kaydet**'e basın.
4. Karşınıza `firebaseConfig` adında bir JavaScript objesi gelecek. Bu objenin **sadece süslü parantezler arasındaki kısmını** kopyalayın:

   ```javascript
   const firebaseConfig = {
     apiKey: "AIzaSy...",
     authDomain: "kuyumcu-pos.firebaseapp.com",
     projectId: "kuyumcu-pos",
     storageBucket: "kuyumcu-pos.appspot.com",
     messagingSenderId: "1234567890",
     appId: "1:1234567890:web:abcdef..."
   };
   ```

   Kopyalanacak kısım (kıvrımlı parantezler dahil):

   ```json
   {
     "apiKey": "AIzaSy...",
     "authDomain": "kuyumcu-pos.firebaseapp.com",
     "projectId": "kuyumcu-pos",
     "storageBucket": "kuyumcu-pos.appspot.com",
     "messagingSenderId": "1234567890",
     "appId": "1:1234567890:web:abcdef..."
   }
   ```

   > Not: Uygulama Ayarlar'a yapıştırırken hem JS objesini hem JSON'u kabul eder; tırnaklar düzelmeden çalışır.

### 3. Firestore Veritabanı Etkinleştirme

1. Sol menüden **Build > Firestore Database**'e gidin.
2. **Veritabanı oluştur**'a tıklayın.
3. **Üretim modunda başlat**'ı seçin (kuralları birazdan ayarlayacağız).
4. Konum olarak `eur3 (europe-west)` veya size yakın bir bölge seçin.
5. **Etkinleştir**'e basın.

### 4. Firestore Güvenlik Kuralları

Tek mağazada birden fazla cihazda kullanım için **basit kural** (geliştirme):

1. Firestore Database > **Kurallar** sekmesine girin.
2. İçeriği aşağıdakiyle değiştirin ve **Yayınla**'ya basın:

   ```
   rules_version = '2';
   service cloud.firestore {
     match /databases/{database}/documents {
       match /{document=**} {
         allow read, write: if true;
       }
     }
   }
   ```

> Üretim ortamında IP kısıtlaması veya Firebase Authentication ile koruma eklemeniz önerilir. Bu uygulamada şifre kontrolü uygulama içinde yapılır.

### 5. Uygulamada Config'i Girme

1. Uygulamayı açın, **Yönetim** olarak girin.
2. **Ayarlar > Firebase Config** bölümüne gelin.
3. 2. adımda kopyaladığınız JSON / JS objesini metin kutusuna yapıştırın.
4. **Kaydet ve Bağlan** butonuna basın.
5. Sayfa otomatik yenilenir. Üstte **"Firebase bağlı - bulut modu"** bildirimi görmelisiniz.
6. Mevcut yerel verileriniz otomatik olarak Firestore'a yüklenir.

### 6. Diğer Cihazlara Kurulum

1. Aynı URL'yi diğer cihazda açın.
2. Ayarlar > Firebase Config'e **aynı JSON**'u yapıştırın.
3. Veriler otomatik olarak gelir. Her satış, stok değişikliği vb. anlık yansır.

---

## PWA - Ana Ekrana Ekleme

### Android (Chrome)
1. Uygulamayı Chrome'da açın.
2. Üst menü (üç nokta) > **Ana ekrana ekle**.
3. Adı onaylayıp **Ekle**.

### iPhone / iPad (Safari)
1. Uygulamayı Safari'de açın.
2. Alt menüden **Paylaş** ikonuna basın.
3. **Ana Ekrana Ekle**'yi seçin.

### Windows / macOS (Chrome / Edge)
1. Adres çubuğunun sağındaki **yükle** ikonuna tıklayın.
2. Veya menüden **Uygulamayı yükle**.

---

## Veri Yedekleme

- **Ayarlar > Veri > JSON Yedek Al** ile tüm veritabanını indirebilirsiniz.
- Firebase kullanıyorsanız Firestore otomatik olarak yedek tutar (Firebase Console > Firestore > Yedeklemeler).

## Veri Sıfırlama

**Ayarlar > Veri > Tüm Veriyi Sıfırla** tüm ürün, satış, müşteri kayıtlarını siler. Geri alınamaz! Sıfırlamadan önce yedek alın.

## Demo Veri

İlk denemeler için **Ayarlar > Veri > Demo Veri Yükle** ile örnek kategori, ürün ve müşteri kayıtları oluşturulur.

---

## Teknolojiler

- Vanilla JavaScript (ES Modules)
- Tailwind CSS (CDN)
- Chart.js (CDN)
- Firebase v10 (modular SDK, CDN)
- Web Crypto API (SHA-256)
- Service Worker (offline / PWA)

## Tarayıcı Uyumluluğu

- Chrome / Edge / Brave 90+
- Safari 14+
- Firefox 90+
- Android Chrome, iOS Safari (PWA destekli)

## Sorun Giderme

**Firebase bağlanmıyor:**
- Config JSON'unun doğru kopyalandığını kontrol edin.
- Firestore'un etkinleştirildiğinden emin olun.
- Güvenlik kurallarının `allow read, write: if true;` olduğunu doğrulayın.
- Tarayıcı konsolunda (F12) hata mesajını inceleyin.

**Şifremi unuttum:**
- Tarayıcı konsolunda `localStorage.clear()` çalıştırın, sayfayı yenileyin. Tüm yerel ayarlar silinir; Firebase bağlıysa veriler korunur, sadece şifreler varsayılana döner.

**Service Worker güncellenmiyor:**
- Tarayıcı sekmesini kapatıp tüm pencereleri kapatın, yeniden açın.
- Veya DevTools > Application > Service Workers > Unregister.

---

## Lisans

Bu proje özel kullanım için hazırlanmıştır. Tüm hakları sahibine aittir.
