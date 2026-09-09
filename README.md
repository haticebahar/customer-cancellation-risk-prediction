# Müşteri İptal Riski Tahmini ve Önceliklendirme Modeli
End-to-end ML + LLM project: predicts customer cancellation risk from e-commerce order behavior and generates natural-language risk summaries with actionable recommendations via Claude.

Müşteri İptal Riski Tahmini ve Önceliklendirme Modeli

E-ticaret sipariş verilerinden müşteri iptal/iade riskini tahmin eden ve bu riski operasyon ekipleri için anlaşılır, aksiyona dönüştürülebilir bir formata çeviren uçtan uca bir ML + LLM projesi.

Problem

Bir sipariş verildiğinde, bu müşterinin ileride siparişi iptal etme riski ne kadar yüksek? Bunu önceden tahmin edebilmek, operasyon ve kampanya ekiplerinin reaktif değil proaktif davranmasını sağlar.

Veri Seti

UCI Online Retail Dataset — 2010-2011 dönemine ait, İngiltere merkezli bir online perakendecinin gerçek sipariş verileri (541.910 satır, 4.372 müşteri).

Yaklaşım
Zaman ayrımlı feature/label mühendisliği: Veri, geçmiş dönem (özellik çıkarma) ve gelecek dönem (etiket oluşturma) olarak ikiye bölündü. Müşteri bazlı davranışsal özellikler türetildi: sipariş sıklığı, harcama, ürün çeşitliliği, geçmiş iptal oranı, recency/tenure.

Kritik bir metodolojik düzeltme: 
İlk etiket tanımı, "iptal edecek mi" sorusunu değil, dolaylı olarak "gelecekte tekrar aktif olacak mı" sorusunu da ölçüyordu — çünkü bir müşterinin iptal davranışının gözlemlenebilmesi için önce gelecekte tekrar alışveriş yapması gerekiyordu. Bu, popülasyon "gelecekte zaten aktif olan müşteriler" ile sınırlandırılarak düzeltildi.

Model geliştirme: 
Baseline olarak Logistic Regression kuruldu, bir multicollinearity sorunu (total_orders ve total_line_items arasında 0.79 korelasyon) tespit edilip düzeltildi. Random Forest ile karşılaştırıldı, daha dengeli performansı nedeniyle final model olarak seçildi.

LLM katmanı: 
Modelin ürettiği risk skorları tek başına iş birimi için yeterli değil — "bu müşterinin iptal riski %86" çıktısı "neden, ne yapmalıyım" sorularını cevapsız bırakıyor. Bu yüzden risk skorları ve davranışsal veriler, rol + görev + biçim + kısıt çerçevesiyle tasarlanmış bir prompt aracılığıyla Claude'a aktarılarak, operasyon ekibinin anlayacağı dilde risk nedeni ve somut aksiyon önerisi üretildi.

PO/ürün katmanı: 
Proje, mini SWOT analizi, RICE önceliklendirmesi ve bir epic'in user story/task seviyesine kırılmasıyla desteklendi — teknik çözümün iş kararlarına nasıl bağlandığını göstermek için.

Sonuçlar
Model	Precision	Recall	Accuracy
Logistic Regression	0.40	0.54	0.63
Random Forest (final)	0.49	0.52	0.71

En belirleyici özellikler: total_spend, past_cancel_rate, avg_order_value, recency_days.

LLM Katmanı Hakkında Bir Not

Prompt'lar kod ile otomatik olarak müşteri verisiyle dolduruldu, ancak LLM'e gönderme adımı bilinçli bir kaynak kullanım kararıyla manuel yapıldı (Claude.ai arayüzü üzerinden) — prototip aşamasında gereksiz API maliyeti oluşturmadan prompt tasarımını doğrulamak amacıyla. Üretim ortamında bu adım birkaç satır kodla (anthropic Python SDK ile) otomatikleştirilebilir; asıl tasarım emeği zaten prompt şablonunun kurgusunda.

Kullanılan Teknolojiler

Veri işleme: pandas, numpy

Modelleme: scikit-learn (Logistic Regression, Random Forest)

LLM: Anthropic Claude (prompt engineering ile)

Ortam: Jupyter Notebook

Proje Yapısı
├── musteri_iptal_riski_tahmini.ipynb   # Ana notebook (uctan uca analiz)
├── llm_risk_ozetleri.csv                # LLM tarafindan uretilen risk ozetleri
└── README.md

Sıradaki Adımlar
Karar eşiğini (threshold) iş ihtiyacına göre kalibre ederek recall oranını artırmak
LLM katmanına, çıktıların kullanıcı geri bildirimiyle geliştirilebildiği bir onay/geri bildirim döngüsü eklemek
Kampanya ve stok verisiyle modeli zenginleştirmek
