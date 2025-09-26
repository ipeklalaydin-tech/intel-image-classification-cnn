
# Intel Image Classification (CNN)

Bu proje, [Intel Image Classification](https://www.kaggle.com/datasets/puneet6060/intel-image-classification) veri seti üzerinde  
**CNN kullanarak Buildings, Forest, Glacier, Mountain, Sea ve Street görüntülerini sınıflandırmayı** hedefler.  

---

## Proje Özeti
Bu çalışmaya başlarken birkaç farklı deneme yaptım ve süreç boyunca hem model mimarimi hem de eğitim stratejimi geliştirdim:  

- **İlk Deneme:** Validation split kullanmadığım için model aşırı öğrenme (overfitting) yaptı. Accuracy yaklaşık **%63** civarındaydı ve gerçek performansı yansıtmıyordu.  
- **İkinci Deneme:** Validation split ekledim ve mimariyi biraz derinleştirdim. Accuracy **%86**’ya kadar çıktı ama 50 epoch’luk eğitimin sadece 15 epoch’u bile **1.5 saat** sürdü — yani çok yavaştı.  
- **Son Model (Bu Repo):** Daha optimize bir yaklaşım izledim. Mixed precision ve `tf.data` optimizasyonları ekledim. EarlyStopping sayesinde eğitim 15. epoch civarında durdu ve:  
  - **Eğitim süresi:** ~30 dakika  
  - **Validation accuracy:** ~%80  
  - **Test accuracy:** ~%76  

Bu haliyle model hem daha hızlı hem de daha genellenebilir oldu.

---

## Model & Eğitim Ayarları
- **Model Mimarisinde:**  
  5 adet Conv2D + MaxPooling2D bloğu → Flatten → Dense (512 → 128) → Dropout (0.2)  
  Her konvolüsyon katmanında L2 regularizasyonu eklendi.  
- **Optimizer:** Adamax (lr=5e-4)  
- **Loss:** Sparse Categorical Crossentropy  
- **Callbacks:**  
  - `EarlyStopping` (val_accuracy, patience=4)  
  - `ReduceLROnPlateau` (val_loss, factor=0.3)  
- **Validation Split:** %15 - Overfit kontrolü sağladı  
- **Batch Size:** 64 - GPU belleği ve hız dengesi  
- **Ekstra:** Mixed Precision ile eğitim süresi önemli ölçüde hızlandı
- 
## Hiperparametre Denemeleri

Aşağıdaki tablo, farklı hiperparametre kombinasyonlarıyla yapılan denemeleri ve sonuçlarını özetler:

| Deneme | Learning Rate | Batch Size | Epoch | Dropout | Optimizer | Val Accuracy | Notlar |
|-------|---------------|-----------|-------|---------|-----------|-------------|--------|
| 1 | 0.001 | 32 | 20 | 0.3 | Adam | 0.78 | Overfit gözlendi, val. loss arttı |
| 2 | 0.0005 | 64 | 25 | 0.4 | Adam | 0.81 | Daha dengeli sonuçlar, loss daha stabil |
| 3 | 0.0001 | 64 | 30 | 0.4 | Adam | 0.80 | Eğitim yavaşladı, iyileşme sınırlı |
| 4 | ... | ... | ... | ... | ... | ... | ... |

---

## Sonuçlar
- **Validation Accuracy:** ~80%  
- **Test Accuracy:** ~76%  
- **Toplam Eğitim Süresi:** ~30 dakika  

---

## Süreçte Öğrendiklerim
- Validation split kullanmamak - overfit ve düşük genelleme  
- L2 regularizasyon + Dropout eklemek -overfit azaldı
- Mixed precision + prefetching - eğitim süresi %70 kısaldı
- IMG_SIZE’ı 192x192’a düşürmek - GPU bellek taşmalarını önledi 

---

## Gelecek Planlar
- Farklı optimizer’lar (SGD + momentum, AdamW) denemek  
- Transfer learning (EfficientNetB0, ResNet50 gibi) ile doğruluğu artırmak  
- Veri artırma (augmentation) stratejilerini daha da geliştirmek  

## Kaggledan görmek için:
https://www.kaggle.com/code/lalayd/son-proje
