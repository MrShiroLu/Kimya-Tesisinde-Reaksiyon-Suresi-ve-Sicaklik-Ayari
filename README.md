# Senaryo 5: Kimya Tesisinde Reaksiyon Süresi ve Sıcaklık Ayarı Optimizasyonu

## İçindekiler
- [Ödev Bilgileri](#ödev-bilgileri)
- [Proje Açıklaması](#proje-açıklaması)
- [Problem Tanımı](#problem-tanımı)
- [Notebook İçeriği](#notebook-içeriği)
---

## Proje Açıklaması

Bu proje, kimyasal üretim süreçlerinde **reaksiyon verimi optimizasyonu** için tasarlanmış kapsamlı bir matematiksel optimizasyon çalışmasıdır. 

### Problem Bağlamı

Kimya endüstrisinde, üretim süreçlerinin verimliliği doğrudan reaksiyon parametrelerinin optimizasyonuna bağlıdır. Reaksiyon süresi ve sıcaklık, kimyasal reaksiyonların hızını ve verimini etkileyen iki kritik parametredir. Bu parametrelerin yanlış ayarlanması:
- Düşük ürün verimi
- Enerji israfı
- Zaman kaybı
- Güvenlik riskleri

gibi sonuçlara yol açabilir.

### Çalışmanın Kapsamı

Bu çalışma, kısıtlı optimizasyon teknikleri kullanarak kimyasal bir tesiste optimal çalışma koşullarının belirlenmesini amaçlamaktadır. İki farklı optimizasyon algoritması (SLSQP ve Differential Evolution) kullanılarak sonuçlar karşılaştırılmış ve en iyi parametreler belirlenmiştir.

### Ana Amaç
Verilen güvenlik ve operasyonel kısıtlar altında reaksiyon verimini maksimize eden optimal reaksiyon süresi ve sıcaklık değerlerini bulmak ve bu süreçte farklı optimizasyon algoritmalarının performansını karşılaştırmak.

---

## Problem Tanımı

### Matematiksel Formülasyon

Bu problem, **kısıtlı nonlineer optimizasyon** problemi olarak modellenmiştir.

#### Amaç Fonksiyonu

Maksimize edilecek reaksiyon verimi fonksiyonu:

```
maximize: y = 8x₁ + 3x₂ - x₁x₂ + x₁²
```

**Fonksiyon Bileşenleri:**
- `8x₁`: Reaksiyon süresinin doğrudan katkısı (pozitif etki)
- `3x₂`: Sıcaklığın doğrudan katkısı (pozitif etki)
- `-x₁x₂`: Etkileşim terimi (negatif etki - aşırı değerler verimlilik kaybı)
- `x₁²`: Sürenin quadratik etkisi (uzun süre daha fazla dönüşüm)

#### Karar Değişkenleri

- **x₁ (Reaksiyon Süresi)**: 
  - Birim: Dakika
  - Aralık: [10, 60]
  - Fiziksel Anlamı: Reaktörlerde kimyasal reaksiyonun devam ettiği süre
  
- **x₂ (Sıcaklık)**:
  - Birim: Derece Celsius
  - Aralık: [40, 120]
  - Fiziksel Anlamı: Reaksiyon ortamının sıcaklığı

#### Kısıtlar

**1. Toplam Süre-Sıcaklık Kısıtı:**
```
x₁ + x₂ ≤ 140
```
- **Gerekçe**: Yüksek sıcaklık ve uzun sürenin birleşimi güvenlik riski oluşturur
- **Pratik Anlam**: Reaktör termal kapasitesi ve güvenlik standartları

**2. Minimum Sıcaklık Kısıtı:**
```
x₂ ≥ 60
```
- **Gerekçe**: Reaksiyonun başlaması için minimum aktivasyon enerjisi gerekli
- **Pratik Anlam**: 60°C altında reaksiyon hızı çok düşük

**3. Sınır Kısıtları:**
```
10 ≤ x₁ ≤ 60
40 ≤ x₂ ≤ 120
```
- **Gerekçe**: Ekipman kapasitesi, güvenlik ve operasyonel kısıtlar

### Problem Karakteristiği

- **Tip**: Kısıtlı nonlineer optimizasyon
- **Değişken Sayısı**: 2
- **Kısıt Sayısı**: 2 eşitsizlik + 4 sınır kısıtı
- **Fonksiyon Türü**: Kuadratik (convex olmayan)
- **Çözüm Uzayı**: 2 boyutlu sürekli alan

---

## Notebook İçeriği

### 1. Kütüphane İçe Aktarımı

```python
import numpy as np
import matplotlib.pyplot as plt
from scipy.optimize import minimize, differential_evolution
from mpl_toolkits.mplot3d import Axes3D
import pandas as pd
import warnings
warnings.filterwarnings('ignore')

plt.rcParams['font.family'] = 'DejaVu Sans'
plt.rcParams['axes.unicode_minus'] = False
```

**Kullanılan Kütüphaneler:**
- **NumPy**: Sayısal hesaplamalar ve matris işlemleri için
- **Matplotlib**: 2D ve 3D görselleştirmeler için
- **SciPy**: Optimizasyon algoritmaları (minimize, differential_evolution)
- **Pandas**: Veri analizi ve tablo gösterimleri için
- **mpl_toolkits**: 3D grafik desteği için

Ayrıca Türkçe karakter desteği ve uyarı mesajlarının bastırılması yapılandırılır.

---

### 2. Amaç Fonksiyonlarının Tanımı

#### Ana Fonksiyon: `objective_function(x)`

Reaksiyon verimi fonksiyonunu hesaplar:

```python
def objective_function(x):
    """
    Reaksiyon verimi fonksiyonu
    y = 8x1 + 3x2 - x1*x2 + x1^2
    
    Parametreler:
    x[0] = x1 : Reaksiyon süresi (dk)
    x[1] = x2 : Sıcaklık (derece C)
    
    Dönüş:
    float: Reaksiyon verimi
    """
    x1, x2 = x[0], x[1]
    return 8*x1 + 3*x2 - x1*x2 + x1**2
```

**Fonksiyon Analizi:**
- **Girdi**: İki elemanlı liste/array [x1, x2]
- **Çıktı**: Tek sayısal değer (verim)
- **Özellik**: Nonlineer, quadratik terimler içerir

#### Minimizasyon Adaptörü: `objective_function_minimization(x)`

SciPy'nin optimize fonksiyonları minimize etmek üzere tasarlandığından, maksimizasyon problemini minimizasyona dönüştürür:

```python
def objective_function_minimization(x):
    """
    Maksimizasyonu minimizasyona çevirir
    minimize -f(x) = maximize f(x)
    """
    return -objective_function(x)
```

**Matematiksel Dönüşüm:**
```
Orijinal: maximize f(x)
Dönüştürülmüş: minimize -f(x)
```

Bu yaklaşım, tüm optimizasyon algoritmalarında standart bir tekniktir.
---

### 3. Kısıtların Tanımlanması

Optimizasyon probleminde iki tip kısıt vardır: sınır kısıtları ve eşitsizlik kısıtları.

#### Sınır Kısıtları (Bounds)

```python
bounds = [
    (10, 60),   # x1: Reaksiyon süresi (dk)
    (40, 120)   # x2: Sıcaklık (derece C)
]
```
**Açıklama:**
- Her değişken için (min, max) tuple formatında tanımlanır
- Bu kısıtlar "box constraints" olarak da bilinir
- Fiziksel ve operasyonel limitlerden kaynaklanır

#### Eşitsizlik Kısıtları (Inequality Constraints)

```python
constraints = [
    {'type': 'ineq', 'fun': lambda x: 140 - x[0] - x[1]},  # x1 + x2 ≤ 140
    {'type': 'ineq', 'fun': lambda x: x[1] - 60}           # x2 ≥ 60
]
```
**SciPy Kısıt Formatı:**
- `'type': 'ineq'`: Eşitsizlik kısıtı
- `'fun'`: Kısıt fonksiyonu (≥ 0 formatında olmalı)

**Matematiksel Dönüşüm:**
```
Orijinal: x1 + x2 ≤ 140
SciPy için: 140 - x1 - x2 ≥ 0

Orijinal: x2 ≥ 60
SciPy için: x2 - 60 ≥ 0
```

**Kısıt Sınırları:**
- **Birinci kısıt**: Güvenlik limiti - toplam "yük" 140'ı geçmemeli
- **İkinci kısıt**: Aktivasyon eşiği - minimum reaksiyon sıcaklığı

**Fizibilite Bölgesi:**
Tüm kısıtları sağlayan (x1, x2) noktalarının oluşturduğu alan 2D uzayda bir poligondur.

---

### 4. **Optimizasyon Yöntemleri**

#### 4.1. SLSQP (Sequential Least Squares Programming)

**Özellikler:**
- Gradient tabanlı yerel optimizasyon algoritması
- Kısıtlı optimizasyon problemleri için etkili
- Hızlı yakınsama
- Başlangıç noktasına duyarlı

**Kullanım:**
```python
result_slsqp = minimize(
    objective_function_minimization,
    x0=[35, 80],  # Başlangıç noktası
    method='SLSQP',
    bounds=bounds,
    constraints=constraints
)
```
#### 4.2. Differential Evolution

**Özellikler:**
- Genetik algoritma tabanlı global optimizasyon
- Başlangıç noktasına duyarsız
- Global optimumu bulma olasılığı yüksek
- Daha fazla hesaplama süresi gerektirir

**Kullanım:**
```python
result_de = differential_evolution(
    objective_with_penalty,
    bounds,
    seed=42,
    maxiter=1000
)
```
**Penalty Yaklaşımı:**
Kısıt ihlallerini büyük ceza değerleriyle fonksiyona ekler:
```python
def constraint_penalty(x):
    penalty = 0
    if x[0] + x[1] > 140:
        penalty += 1e6 * (x[0] + x[1] - 140)
    if x[1] < 60:
        penalty += 1e6 * (60 - x[1])
    return penalty
```
---

### 5. **Sonuç Karşılaştırması**

Her iki yöntemin sonuçları karşılaştırılır ve en iyi sonuç seçilir:

```python
best_result = result_slsqp if -result_slsqp.fun > -result_de.fun else result_de
```
Pandas DataFrame kullanılarak görsel bir karşılaştırma tablosu oluşturulur.

---

### 6. **Görselleştirmeler**

#### 6.1. 3D Yüzey Grafiği
Amaç fonksiyonunun 3 boyutlu görselleştirmesi:
- **X ekseni**: Reaksiyon süresi (x1)
- **Y ekseni**: Sıcaklık (x2)
- **Z ekseni**: Verim
- Kısıt ihlali yapan bölgeler maskelenir

```python
mask = (X1 + X2 > 140) | (X2 < 60)
Y_masked = np.ma.array(Y, mask=mask)
```
#### 6.2. Kontur Grafiği
- 2D kontour plot ile verim düzeyleri
- Kısıt çizgileri (x1 + x2 = 140 ve x2 = 60)
- Optimal nokta işaretlemesi

#### 6.3. Kesit Grafikleri
Bir değişken sabit tutulurken diğerinin etkisi:
- x2 sabit → x1 değişimi
- x1 sabit → x2 değişimi

---

### 7. **Duyarlılık Analizi**

Optimal noktanın ±10 birim çevresinde parametrelerin etkisi incelenir:

**Analiz Edilen Durumlar:**
- x1'de +5 dk artış/azalış
- x2'de +5 derece C artış/azalış

**Çıktı:** Yüzdesel verim değişimi

```python
change_percent = ((new_yield - optimal_yield) / optimal_yield) * 100
```
---

### 8. **Çözüm Doğrulaması**

Bulunan optimumun güvenilirliğini test etmek için:
- 20 farklı rastgele başlangıç noktası
- Her noktadan SLSQP ile optimizasyon
- Sonuçların istatistiksel analizi (ortalama, std, min, max)
- Histogram grafikleri ile dağılım görselleştirmesi

---

### 9. **Pratik Öneriler ve Rapor**

Kimya tesisi operatörleri için pratik öneriler içeren detaylı rapor:

1. **Optimal Çalışma Koşulları**: En iyi x1 ve x2 değerleri
2. **Kısıt Durumu**: Tüm kısıtların sağlandığının doğrulanması
3. **Pratik Öneriler**: Tolerans aralıkları ve operasyonel tavsiyeler
4. **Risk Analizi**: Duyarlılık seviyesi ve kritik parametreler
5. **Ekonomik Değerlendirme**: Verim, süre ve enerji maliyeti

---