# Play Integrity API Kurulum Rehberi

Bu dokümanda Fingofy uygulamasına **Play Integrity API**'nin nasıl entegre edildiği ve Google Play Console'da nasıl aktif edileceği açıklanmaktadır.

## 🚀 Kurulum Tamamlanan Adımlar

### ✅ 1. NPM Paketi Kurulumu
```bash
npm install react-native-google-play-integrity --save
```

### ✅ 2. Android Dependencies
`android/app/build.gradle` dosyasına eklendi:
```gradle
dependencies {
    // Play Integrity API (Latest version)
    implementation 'com.google.android.play:integrity:1.5.0'
    // ... other dependencies
}
```

### ✅ 3. TypeScript Service Dosyası
`services/playIntegrityService.ts` oluşturuldu ve şu özellikler eklendi:
- ✅ Token oluşturma ve doğrulama
- ✅ Development/Production mod ayarı
- ✅ Kritik işlem kontrolleri
- ✅ Hata yönetimi
- ✅ Analytics entegrasyonu

### ✅ 4. React Hook
`hooks/usePlayIntegrity.ts` oluşturuldu:
- ✅ Güvenlik kontrolü UI state'i
- ✅ Loading ve error handling
- ✅ Kritik işlem kontrolleri

### ✅ 5. App Başlangıç Kontrolü
`App.tsx` içerisine eklendi:
- ✅ Kullanıcı giriş yaptığında otomatik güvenlik kontrolü
- ✅ Development modunda devre dışı bırakma
- ✅ Analytics logging

### ✅ 6. Finansal İşlem Koruması
`AddTransactionScreen.tsx` güncellendi:
- ✅ Para birimi bazında dinamik limit kontrolü
- ✅ Konfigürasyon tabanlı güvenlik seviyeleri
- ✅ Hata mesajları (TR/EN)
- ✅ Loading state yönetimi

## 📋 Google Play Console'da Yapılması Gerekenler

### ⏳ 1. Google Play Console Kurulumu
1. **Google Play Console**'a girin
2. **Uygulamanızı** seçin
3. **App integrity** bölümüne gidin
4. **Play Integrity API**'yi **etkinleştirin**

### ⏳ 2. Google Cloud Project Bağlantısı
1. Google Cloud Console'da yeni proje oluşturun veya mevcut projeyi seçin
2. **Play Integrity API**'yi etkinleştirin
3. **Service Account** oluşturun
4. **JSON key** indirin

### ⏳ 3. Sunucu Tarafı Token Doğrulaması
```javascript
// Backend'de token doğrulama örneği
const { GoogleAuth } = require('google-auth-library');

async function verifyIntegrityToken(token) {
  const auth = new GoogleAuth({
    keyFile: 'path/to/service-account-key.json',
    scopes: ['https://www.googleapis.com/auth/playintegrity']
  });

  const response = await auth.request({
    url: `https://playintegrity.googleapis.com/v1/packages/com.your.package/decodeIntegrityToken`,
    method: 'POST',
    data: { integrityToken: token }
  });

  return response.data;
}
```

## 🔒 Güvenlik Seviyesi Ayarları

### Development Mode
- ❌ Play Integrity **devre dışı**
- ✅ Tüm işlemler **izin veriliyor**
- 📊 Analytics **sadece log**

### Production Mode
- ✅ Play Integrity **aktif**
- 🚫 Güvenlik kontrolü geçemeyen cihazlar **engelleniyor**
- 💰 **Para birimi bazında dinamik limitler**

## 💰 Güvenlik Limitleri Konfigürasyonu

`constants/securityConfig.ts` dosyasında tamamen özelleştirilebilir:

```typescript
export const SECURITY_CONFIG = {
  criticalOperationThresholds: {
    'TRY': 5000,    // 5000₺ ve üzeri (optimize edilmiş)
    'USD': 500,     // 500$ ve üzeri (optimize edilmiş)
    'EUR': 450,     // 450€ ve üzeri (optimize edilmiş)
    'GBP': 400,     // 400£ ve üzeri (optimize edilmiş)
  },
  alwaysCheckIntegrity: false, // Tüm işlemler için zorunlu
  skipInDevelopment: true,     // Development modunda atla
};
```

### Güvenlik Seviyeleri

**💡 Mantık:** Düşük güvenlik = Yüksek limitler (az kontrol), Yüksek güvenlik = Düşük limitler (sık kontrol)

- **LOW SECURITY**: Yüksek limitler (10.000₺, 1000$) - Rahat kullanım, az güvenlik kontrolü
- **MEDIUM SECURITY**: Orta limitler (5.000₺, 500$) - **Varsayılan** ✅ Dengeli güvenlik
- **HIGH SECURITY**: Düşük limitler (2.000₺, 200$) - Sık kontrol, yüksek güvenlik
- **CRITICAL SECURITY**: Tüm işlemler kontrol edilir - Maksimum güvenlik, her işlem

## 🛡️ Kontrol Edilenler

### Cihaz Kontrolü
- ✅ **Root detection**
- ✅ **Custom ROM detection**
- ✅ **Emulator detection**
- ✅ **Sideload detection**

### Uygulama Kontrolü
- ✅ **APK tampering**
- ✅ **Play Store authenticity**
- ✅ **Debug detection**

## 🚨 Hata Durumları

### Güvenlik Kontrolü Başarısız
```
⚠️ "Yüksek miktarlı işlemler için güvenlik kontrolü başarısız.
    Lütfen orijinal uygulamayı kullandığınızdan emin olun."
```

### Servis Kullanılamıyor
```
⚠️ "Güvenlik kontrolü yapılamadı, işlem devam ediyor."
```

## 📊 Analytics Events

### Başlangıç Kontrolü
```javascript
analyticsEventService.logEvent('security_check_completed', {
  success: boolean,
  user_id: string,
  is_development: boolean
});
```

### Kritik İşlem Kontrolü
```javascript
analyticsEventService.logEvent('critical_operation_check', {
  success: boolean,
  timestamp: number
});
```

## 🔧 Test Etme

### Development Test
```javascript
// Development modunda test için
const playIntegrityService = PlayIntegrityService.getInstance();
playIntegrityService.setEnabled(true); // Force enable
```

### Production Test
1. **Release APK** oluşturun
2. **Google Play Internal Testing**'e yükleyin
3. **Fiziksel cihazda** test edin
4. **Play Console** logs'larını kontrol edin

## ⚡ Performans

- **Başlangıç kontrolü**: ~500-1000ms
- **Kritik işlem kontrolü**: ~300-500ms
- **Network latency**: Google servers'a bağlı
- **Caching**: Token'lar 5 dakika önbelleklenir

## 🔄 Güncelleme Süreci

1. **Package update**: `npm update react-native-google-play-integrity`
2. **Android dependency**: `build.gradle`'deki versiyon numarasını güncelle
3. **Clean build**: `cd android && ./gradlew clean`
4. **Rebuild**: `npm run android`

---

## 📞 Destek

Herhangi bir sorun durumunda:
1. **Android Logs**: `adb logcat | grep -i integrity`
2. **React Native Logs**: Metro bundler console
3. **Google Play Console**: App integrity dashboard
