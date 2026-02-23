# Give Me Some Credit — Kredi Riski Modeli

Bu proje, bir bankanın kredi başvurularını değerlendirirken "önümüzdeki 2 yıl içinde borcunu 90+ gün geciktirme" (temerrüt) riskini tahmin etmeyi amaçlar. Gerçek dünya hikâyesi üzerinden ilerleyen bu çalışma; veri keşfi (EDA), temizlik, modelleme ve yarışma formatında `submission.csv` oluşturma adımlarını içerir.

## Hikâye
Bankada kredi onay sorumlususunuz. Başvuran müşterinin profiline (yaş, gelir, borç oranı, gecikme sayıları vb.) bakarak iki yıl içinde temerrüde düşüp düşmeyeceğini tahmin eden bir model istiyorsunuz. Bu notebook, veriyi anlama, makine öğrenmesi modellerini deneme ve en iyi bulduğunuz modeli test setine uygulayıp olasılık tabanlı bir çıkış (`submission.csv`) üretme akışını gösterir.

## Dosya Yapısı
- `data/cs-training.csv` — Eğitim verisi
- `data/cs-test.csv` — Test verisi (ID ile birlikte)
- `notebook/eda-model.ipynb` — Veri keşfi, ön işleme, modelleme ve submission üretimi

## Kolon (Değişken) Anlamları
- `SeriousDlqin2yrs`: Hedef değişken (1 = 2 yıl içinde 90+ gün gecikme / temerrüt, 0 = yok)
- `RevolvingUtilizationOfUnsecuredLines`: Kredi kullanım oranı (kredi kartı vb. limit doluluk oranı)
- `age`: Yaş (sheet içinde 18'den küçükler temizlendi, 90 üzeri clipping ile sınırlandırıldı)
- `NumberOfTime30-59DaysPastDueNotWorse`: 30–59 gün gecikme sayısı
- `DebtRatio`: Gelire oranla toplam borç ödemeleri (yüksekse risk artar)
- `MonthlyIncome`: Aylık gelir (eksik değerler medyan ile dolduruldu)
- `NumberOfOpenCreditLinesAndLoans`: Açık kredi ve kredi kartı sayısı
- `NumberOfTimes90DaysLate`: 90+ gün gecikme sayısı (en önemli risk göstergesi)
- `NumberRealEstateLoansOrLines`: Gayrimenkul kredisi/limit sayısı
- `NumberOfTime60-89DaysPastDueNotWorse`: 60–89 gün gecikme sayısı
- `NumberOfDependents`: Bakmakla yükümlü olunan kişi sayısı (eksikler mod ile dolduruldu)

## Özet EDA ve Ön İşleme (Notebook'tan çıkan özet)
- Sınıf dengesizliği mevcut: `SeriousDlqin2yrs=1` oranı oldukça düşük (dengesizlik için ağırlıklandırma/örnekleme düşünüldü).
- `age` değerleri 18 altı satırlar silindi; 90 üzeri değerler clipping ile sınırlandı.
- `MonthlyIncome` eksikleri medyan ile dolduruldu; `NumberOfDependents` eksikleri mod ile dolduruldu.
- Aykırı değer tespiti IQR ile yapıldı; modelleme öncesi mantıklı clipping/transformasyonlar uygulandı.

## Modeller ve Denemeler
- Notebook'ta LightGBM ve XGBoost başta olmak üzere ağaç tabanlı modeller denenmiştir.
- XGBoost için `RandomizedSearchCV` ile hiperparametre araması yapılmış; en iyi parametrelerle model eğitilmiş ve `submission.csv` üretilmiştir.

## Model Performansı — Yeniden Üretme ve Raporlama
Notebook hücreleri model eğitimi ve değerlendirmesi için gerekli kodu içerir. Performansı hesaplamak için kullanılan metrikler:
- Accuracy, Precision, Recall, F1-score, ROC AUC ve Confusion Matrix

Çalıştırmak ve metrikleri görmek için (notebook içinde):
1. `notebook/eda-model.ipynb` dosyasını açın.
2. Hücreleri sırayla çalıştırın (özellikle eğitim ve değerlendirme hücreleri).
3. Test setinde elde edilen sonuçlar konsola yazdırılacaktır; sonuçları `README` içine ya da rapora yapıştırabilirsiniz.

Örnek rapor tablosu (kendinizin doldurması için şablon):

| Model | Accuracy | Precision | Recall | F1 | ROC AUC |
|---|---:|---:|---:|---:|---:|
| XGBoost (tuned) | 0.8218 | 0.2333 | 0.7253 | 0.3531 | 0.7770 |
| LightGBM | — | — | — | — | — |

Not: Aşağıda notebook'tan elde edilen XGBoost (tuned) test metrikleri eklendi; diğer modeller için siz çalıştırıp değerleri ekleyebilirsiniz.

Confusion Matrix (Test, XGBoost tuned):

```
[[23089  4772]
 [  550  1452]]
```

Kısa yorum: Yüksek recall (0.725) ile pozitif sınıfı yakalamada başarılı ancak precision (0.233) düşük; model pozitif tahminlerde yanlış alarm veriyor. İş ihtiyacı doğrultusunda precision ağırlıklı iyileştirme veya threshold optimizasyonu önerilir.

## Submission (Kaggle tarzı çıktı)
Notebook sonundaki hücreler `data/cs-test.csv` üzerinde aynı ön işleme adımlarını uygulayıp `submission.csv` dosyası üretir. Üretim adımları:
1. `notebook/eda-model.ipynb` içindeki son hücreleri çalıştırın.
2. Oluşan `submission.csv` dosyasını kontrol edin — sütunlar `Id` ve `Probability` olmalıdır.

## Gereksinimler (örn. bir virtualenv içinde kurulacak paketler)
Önerilen minimum paketler:
```
pandas
numpy
matplotlib
seaborn
scikit-learn
xgboost
lightgbm

# opsiyonel (hız/çalışma):
joblib
```

Kurulum örneği:
```bash
python -m venv .venv
.venv\\Scripts\\activate    # Windows
pip install -r requirements.txt   # veya pip install pandas numpy scikit-learn xgboost lightgbm matplotlib seaborn
```

