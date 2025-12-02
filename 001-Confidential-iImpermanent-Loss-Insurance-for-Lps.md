# 001 - Confidential Impermanent Loss Insurance Hook

Project Link: [Confidential-iImpermanent-Loss-Insurance-for-Lps](
https://github.com/LayintonDev/Confidential-iImpermanent-Loss-Insurance-for-Lps)

Temel amaç LP sağlayıcının impermanent losstan korunmasını sağlayacak bir sigorta protokolü geliştirmek. 

Gizlilik esası ile çalışıyorlar. Çünkü, kim ne kadar token koymuş, hangi havuzda, ne zaman girmiş gibi bilgiler açığa çıksın istemiyorlar. Bu gizliliği sağlamak için FHE (Fully Homomorphic Encryption) kullanıyorlar. FHE, veriyi şifreliyken (kilitli bir kutunun içindeyken) işlemeye izin verir. Proje, Fhenix altyapısını kullanarak senin cüzdan bakiyeni ve pozisyonunu şifreliyor. Sigorta protokolü, senin ne kadar paran olduğunu görmeden, riskini hesaplayabiliyor ve primini belirleyebiliyor. Yani matematiksel işlem yapılıyor ama veri gizli kalıyor.

EigenLayer üzerinde çalışan özel doğrulayıcı servisleri (AVS — Actively Validated Service) bu sigorta sisteminin işlemlerini doğruluyor. AVS, sigorta şartlarının oluşup oluşmadığını kontrol etmek için kendi zayıf ağını kurmak yerine, Ethereum'un güvenliğini arkasına almış EigenLayer AVS operatörlerini kullanıyor. Bu operatörler, "Evet, bu hesap matematiksel olarak zarar etti, ödeme yapılmalı" diye onay veriyor (attestation).

"Decentralized M-of-N operator verification" : Bu, “bir grubun içinden en az M kişi onay verirse işlem geçerli sayılır” anlamına gelen bir çoklu-imza / çoklu-doğrulama yapısıdır.

## 1. Entegrasyon

- Bir sigorta sisteminde hasar tespiti yapan hakemler (validator) vardır. Hakem rüşvet alıp "Hasar var, ödeme yapın" diye yalan söylerse ne olur? Bu proje, EigenLayer'ı bir "teminat" sistemi olarak kullanıyor. Doğrulayıcılar (Validator), sisteme girmek için ortaya para (stake) koymak zorundalar.

> **Mekanizma:**
>
> Doğrulayıcı, bir sigorta ödemesini onaylar (Attestation/İmza atar).
>
> Eğer bu onayın sahte veya hatalı olduğu anlaşılırsa, akıllı kontrat otomatik olarak devreye girer.
>
> Slashing: Doğrulayıcının ortaya koyduğu paranın bir kısmı veya tamamı yakılır (elinden alınır).

- Geçici Kayıp (IL) hesaplamak için kullanıcının cüzdanına ne zaman girdiği, ne zaman çıktığı ve havuzdaki payı bilinmelidir. Bu veriyi verirsen, stratejin ifşa olur. Proje, Fhenix'in hazır yapı taşlarını (toplama, çıkarma, mantıksal kıyaslama gibi şifreli fonksiyonlar) kullanıyor.

> **Mekanizma:**
>
> Girdi: Kullanıcı verisini şifreleyip yollar. Kimse içeriği görmez.
>
> İşlem: Sistem, Fhenix üzerindeki şifreli veriyi alır, "Giriş Fiyatı" ile "Çıkış Fiyatı" arasındaki farkı şifreli olarak hesaplar.
>
>Sonuç: Sonuç "Ödeme Yapılmalı" veya "Yapılmamalı" olarak çıkar, ancak aradaki sayılar (bakiye, strateji) asla açığa çıkmaz.


## 2. Fikrin Arkasındaki Motivasyon

DeFi'da insanlar paralarını havuzlara koyup (LP olup) işlem ücretlerinden (fee rewards) para kazanmak isterler. Kripto paraların fiyatları çok oynak olduğu için, bazen havuzdaki varlıkların değeri öyle bir değişir ki, zararınız kazandığınız faizden daha büyük olur. Bu "gizli risk" yüzünden insanlar korkar ve DeFi sistemlerine likidite sağlamaktan vazgeçerler. Sistem tıkanır. 

Bu nedenle, kayba karşı bir sigorta öneriliyor. Mevcut sigortalardan farklı olarak;
1. Decentralized (Merkeziyetsiz): Başında bir patron yok, kodlar ve topluluk (EigenLayer operatörleri) yönetiyor.

2. Verifiable (Doğrulanabilir): "Zarar ettim" dediğinde, sistem bunu matematiksel olarak kanıtlayabiliyor (Yalan yok).

3. Confidential (Gizli): Tüm bu işlemler yapılırken, senin finansal verilerin (Fhenix sayesinde) şifreli kalıyor. Kimse cüzdanının içini görmüyor.



| Katkı | Etki | Açıklama |
|------|-------------|----------|
| Güven Artışı | IL riskini azaltma | LP'ler "Param güvende" diyerek sisteme daha çok inanır.|
| Derin Likidite | Deeper Liquidity | Daha fazla LP sisteme para koyar. Havuzlar büyür, paranın hacmi artar. |
| Sıkışık Fiyat Farkları | Tighter Spreads | Havuzda para arttıkça, büyük işlemler yapsanız bile fiyat kayması (slippage) azalır. Alım satım fiyatları arasındaki fark (spread) daralır. Bu, tüccarlar için daha iyi fiyat demektir. |
| Sağlıklı Piyasalar | Healthier DeFi Markets |Genel olarak fiyatlar daha istikrarlı hale gelir, sistem daha güvenilir olur. |


## 3. Challenges

**Problem:** IL hesaplarken sadece fiyat değişimine bakamazsınız.

1. Dinamik LP Komisyonları (Dynamic LP Fees): Likidite sağlayanlar sabit bir faiz değil, işlem hacmine göre değişen (dinamik) komisyonlar kazanırlar. Bu kazancın, oluşan zararı ne kadar dengelediğini doğru hesaplamak gerekir.

2. Fiyat Oynaklığı (Price Volatility): Fiyatlar saniyeler içinde değişebilir. Bu anlık oynamaların hasar üzerindeki etkisini doğru şekilde kaydetmek ve modele dahil etmek büyük bir matematiksel zorluktur.