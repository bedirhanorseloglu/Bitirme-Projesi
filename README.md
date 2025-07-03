# GNSS Veri Seti İle Konum Optimizasyonu

## 📋 Proje Özeti

Bu proje, **GNSS (Global Navigation Satellite System)** verilerini kullanarak hassas konum tahmini yapmak amacıyla geliştirilmiş bir hibrit sistem sunmaktadır. Proje, **Kalman Filtresi** ve **LSTM (Long Short-Term Memory)** modellerini birleştirerek, geleneksel GPS sistemlerinden %50.3'e varan iyileştirme sağlamaktadır.

## 🎯 Projenin Amacı

Modern navigasyon sistemlerinde GNSS verileri, atmosferik etkiler, çoklu yol etkisi ve sinyal gürültüsü gibi faktörler nedeniyle hata içerebilir. Bu proje, bu hataları minimize ederek daha hassas konum tahmini yapmayı amaçlamaktadır.

## 📊 Veri Seti Bilgileri

- **98 sürüş oturumu** (Kaliforniya, ABD)
- Her oturumda ortalama **4 telefon** ile eş zamanlı kayıt
- Oturum başına ortalama **17,200 satır, 47 sütun**
- Toplam **~6,742,400 gözlem**
- **Test bölgeleri:** MTV (Mountain View), SJC (San Jose), SFO (San Francisco), SVL (Silicon Valley), LAX (Los Angeles)

## 🔧 Metodoloji

### 1. Uydu Sinyali Filtreleme
Kaliteli GNSS verisi elde etmek için aşağıdaki filtreler uygulanmıştır:

- **Taşıyıcı Frekans Hatası:** < 2 MHz
- **Uydu Eğim Açısı:** > 10°
- **Sinyal-Gürültü Oranı (CNR):** > 15 dB-Hz
- **Çoklu Yol Etkisi:** Yok

### 2. WLS (Weighted Least Squares) Konum Tahmini
```python
# Belirsizlik değerlerine göre ağırlıklandırılmış tahmin
weights = 1 / uncertainty_values
position_estimate = WLS(measurements, weights)
```

### 3. Taşıyıcı Faz Düzeltme
- Pseudorange ve taşıyıcı faz hatalarının tespiti
- Ortalama değer ile düzeltme işlemi
- Sagnac etkisinin kompensasyonu

### 4. Anomali Tespiti ve Düzeltme
- **Z-ekseni hız eşiği:** 2 m/s
- **Lineer interpolasyon** ile eksik veri tamamlama
- **Koordinat dönüşümleri:** ECEF → Geodezik → ENU

### 5. Kalman Filtresi
```python
# Mahalanobis mesafesi ile anomali tespiti
mahalanobis_threshold = 30.0
if mahalanobis_distance > threshold:
    reject_measurement()
```

### 6. LSTM Modeli
```python
model = Sequential([
    LSTM(20, return_sequences=False),
    Dense(128, activation='relu'),
    Dense(64, activation='relu'),
    Dense(16, activation='relu'),
    Dense(4, activation='relu'),
    Dense(2)  # latitude, longitude
])
```

## 📈 Sonuçlar

### Performans Metrikleri (Google Pixel 4)
| Yöntem | Ortalama Hata (m) | İyileştirme |
|--------|------------------|-------------|
| Baseline | 2.1929 | - |
| WLS | 1.9769 | 9.8% |
| Kalman Filtresi | 1.1443 | 47.8% |
| **LSTM (Hibrit)** | **1.1253** | **50.3%** |

### Model Performansı
- **LSTM Doğruluk:** 84.44%
- **Trace Length:** 4 (geçmiş 4 zaman adımı)
- **Optimizasyon:** Adam
- **Kayıp Fonksiyonu:** Mean Squared Error

## 🚀 Kullanım Alanları

### 🚗 Otonom Araçlar
Trafik ışıklarına yaklaşırken hassas duruş konumu ayarlama

### 🛸 Drone Navigasyonu
Zorlu hava koşullarında sinyal kaybı durumunda konum tahmini

### 📱 Mobil Harita Servisleri
Yoğun şehir trafiğinde daha net konum gösterimi

### 🌪️ Arama-Kurtarma
Doğal afetlerde enkaz altındaki cihazların hassas konumu

### 🌾 Akıllı Tarım
Santimetre hassasiyetinde otonom tarım makineleri

### ✈️ Havacılık
İHA ve SİHA'larda hassas konum tespiti

## 🔍 Teknik Detaylar

### Koordinat Sistemleri
- **ECEF (Earth-Centered, Earth-Fixed)**: Dünya merkezli koordinatlar
- **Geodezik**: Enlem, boylam, yükseklik
- **ENU (East, North, Up)**: Yerel koordinat sistemi
- **UTM**: Hassas mesafe hesaplama için

### Özellik Çıkarımı
- **Mesafe**: Vincenty formülü ile hesaplanan
- **Hız**: Euclidean mesafe türevi
- **İvme**: Hız türevi
- **Eğrilik**: Rota virajlılığı
- **Median filtreleme**: Ani değişimlerin düzeltilmesi



## 📊 Veri Analizi

### Hata Dağılımı
- **3.5-4 metre aralığında:** 130 ölçüm hatası
- **Ortalama iyileştirme:** Google Pixel 4 ile %50.3

### Koordinat Dönüşüm Hassasiyeti
- **WGS84**: Küresel referans
- **UTM**: Düzlemsel bölgesel hesaplama
- **Vincenty**: Elipsoid yüzey mesafe hesabı

## 🔮 Gelecek Geliştirmeler

### Teknik İyileştirmeler
- **GRU/Transformer** modelleri ile karşılaştırma
- **Çevresel veriler** (hava durumu, yol durumu) entegrasyonu
- **Gerçek zamanlı sistem** optimizasyonu
- **TinyML** çözümleri ile hafif model geliştirme

### Genelleme
- Farklı coğrafyalarda test ve eğitim
- Çoklu cihaz desteği
- Farklı GNSS konstelasyonları (GPS, GLONASS, Galileo, BeiDou)

## ❓ Sıkça Sorulan Sorular

### **Q: Neden LSTM tercih ettiniz?**
**A:** LSTM, zaman serisi verileriyle çalışmak için idealdir çünkü geçmiş bilgileri uzun süre hafızasında tutabilir. GNSS verileri zamana bağlı olduğu için LSTM bu sinyallerdeki örüntüleri yakalayabilir.

### **Q: Kalman filtresi neden yeterli olmadı?**
**A:** Kalman filtresi lineer sistemlerde başarılıdır ancak GNSS sinyalleri çevresel faktörlere maruz kaldığında doğruluğu sınırlı kalır. LSTM ise doğrusal olmayan ilişkilere daha iyi adapte olabilir.

### **Q: Bu sistem gerçek zamanlı çalışabilir mi?**
**A:** Şu anki sistem offline çalışmaktadır. Model GRU veya TinyML çözümleriyle optimize edilirse gerçek zamanlı kullanım mümkündür.

### **Q: Farklı şehirlerde işe yarar mı?**
**A:** Bu çalışma Silikon Vadisi verisiyle test edilmiştir. Farklı coğrafyalarda genelleme için o bölgelere özel veri ile yeniden eğitim gereklidir.

## 📞 İletişim

**Geliştirici:** Bedirhan Örseloğlu  
**GitHub:** [@bedirhanorseloglu/githhub](https://github.com/bedirhanorseloglu)  
**LinkedIn:** [@bedirhanorseloglu/linkedin](https://www.linkedin.com/in/bedirhanorseloglu/)


## 🙏 Teşekkürler

Bu proje, **TÜBİTAK** işbirliğiyle gerçekleştirilmiştir. Maddi ve manevi destekleri için teşekkür ederim.

---

> **Not:** Bu README, projenin teknik detaylarını kapsamaktadır. Daha detaylı bilgi için notebook dosyasını inceleyebilirsiniz.
