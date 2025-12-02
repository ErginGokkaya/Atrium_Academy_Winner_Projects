# 002 - Alpha Engine Hook

Project Link: [AlphaEngineHook](https://github.com/Nilay27/AlphaEngineHook)

Temel amaç mahremiyet korunaklı swap ve otomatik strateji çalıştırma.

| Kavram | Açıklama |
|---------|--------|
| Privacy-Preserving Swaps (Gizliliği Koruyan Takaslar) | Bu, bir kullanıcının yaptığı takas işleminin ayrıntılarının (örneğin, ne kadar ve hangi fiyattan alıp sattığı) blok zincirinde herkes tarafından görülemiyor olması demektir. Bu, trader'lar için önemli bir avantajdır, çünkü büyük işlemlerinin diğer trader'lar tarafından hemen fark edilip, fiyatları manipüle etme amaçlı *front-running* girişimlerini engeller. |
| Automated Strategy Execution (Otomatik Strateji Yürütme)  | Bu, trader'ların belirlediği alım-satım stratejilerinin (örneğin, belirli bir fiyat aralığında otomatik olarak al veya sat gibi) merkezi bir sunucuya ihtiyaç duymadan, otomatik olarak ve güvenilir bir şekilde çalıştırılmasıdır. |
| FHE (Fully Homomorphic Encryption) | Bu, verilerin şifresi çözülmeden hesaplama yapılmasını sağlayan çığır açıcı bir şifreleme teknolojisidir. AlphaEngine, bu teknolojiyi kullanarak, trader stratejilerinin veya hesaplamalarının şifreli kalmasını sağlarken, yine de blok zincirinde çalışmasına olanak tanır. Gizliliği koruyan takasları mümkün kılan anahtar budur. |
| EigenLayer AVS (Actively Validated Services) | EigenLayer, Ethereum'daki mevcut staking güvenliğini diğer merkeziyetsiz protokollere (AVS'ler) kiralamayı sağlayan bir protokoldür. AlphaEngine, EigenLayer'ın bu **aktif olarak doğrulanmış hizmetler** yapısını kullanır. Bu, otomatik strateji yürütme işlevinin, Ethereum'un büyük güvenlik gücüyle desteklenen, güvenilir ve merkeziyetsiz bir şekilde çalışmasını sağlar. |

# 1. Entegrasyon

**Fhenix CoFHE:** Kullanıcı bir takas emri (swap intent) verdiğinde, bu emir başlangıçta şifrelenmiştir. İşlemin gerçekleşeceği ana gelindiğinde, Fhenix sistemi, emrin içeriğinin (hangi fiyattan, ne kadar alınacağı) gizliliği tehlikeye atmadan, sadece gerekli hesaplamalar için güvenli bir şekilde deşifre edilmesini (açılmasını) sağlar.

**EigenLayer AVS (Actively Validated Service):** "Slashable Correctness" (Cezalandırılabilir Doğruluk), AVS'lerin kritik bir özelliğidir. Eğer komitedeki bir operatör hile yapmaya kalkışırsa veya hatalı/kötü niyetli bir işlem doğrularsa, EigenLayer mekanizması devreye girer ve o operatörün stake ettiği ETH'yi keser (slash). Bu, komitenin her zaman doğru ve dürüst çalışmasını garantileyen güçlü bir güvenlik garantisidir.

# 2. Problem ve Motivasyon

- DeFi'deki mevcut getiriler (yields), yalnızca birkaç fon yöneticisinin bilgisiyle (knowledge of a few fund managers) sınırlı kalıyor. Geleneksel finansta olduğu gibi, DeFi'de de yüksek getiri elde edebilen stratejilere sadece az sayıda, genellikle kapalı fon veya uzman erişebiliyor. Bu, sermayenin geniş bir havuzdaki en iyi ve çeşitli stratejilere erişimini kısıtlar. Başarılı stratejiler dar bir çevrede kaldığı için, genel DeFi kullanıcıları potansiyel maksimum getiriyi alamıyor.
- On-chain traders alım-satım stratejilerini açığa çıkarıyorlar (expose strategies) ve MEV (Maximal Extractable Value) kayıplarına uğruyorlar. MEV, blok üreticileri (validator'lar) veya özel botların, bir işlemi blok içine dahil etme sırasını değiştirerek veya işlemden önce/sonra kendi işlemlerini ekleyerek (front-running, back-running) kullanıcı işleminden kar maksimize etme yeteneğidir.

AlphaEngine'i benzersiz kılan şey, daha önce ayrı ayrı kullanılan üç güçlü teknolojik bileşeni ilk kez, tek bir uyumlu varlık yönetimi paradigması altında birleştirmesidir:

1. FHE Gizliliği (FHE Privacy): Ticari sırları ve stratejileri açığa çıkmaktan korur.

2. Uniswap v4 Hook'ları (Uniswap v4 Hooks): Platforma yerel, zincir üzerinde otomasyon ve özelleştirme sağlar.

3. EigenLayer AVS Güvenliği (EigenLayer AVS Security): Bu otomasyonun ve gizliliğin, Ethereum'un en yüksek düzeyde güvenlik garantisiyle (cezalandırılabilir doğruluk - slashable correctness) merkeziyetsiz bir şekilde çalışmasını sağlar. AVS;
    - Simülasyon ve Doğrulama: AVS operatörleri, trader'ların gizli FHE stratejilerini çalıştırmadan önce, stratejiyi simüle eder ve doğrulamasını yapar.

    - Kârlılık Kısıtlaması (Enforcing Profitability Constraints): Operatörler, sadece kârlı olduğu ve yüksek alfa (high-alpha, yani piyasadan daha iyi getiri) üretme potansiyeli olduğu kanıtlanan işlemlerin sisteme girmesine izin verir.

    - Etkisi: Bu, havuzlara giren işlemlerin kalitesini en üst düzeye çıkarır. Fonlar, rastgele veya zayıf stratejiler yerine, sadece doğrulanmış ve kârlı işlemler için kullanılır.

# 3. Challenges
Bir işlemin kârlılık eşiğini geçip geçmediğini zincir dışında güvenli bir şekilde kontrol ettikten sonra, bu bilginin gizliliği koruyarak zincire geri iletilmesi, teknik olarak çok zorlu bir süreçtir.

- Gizlilik: İşlemlerin gizli (FHE) kalması gerekiyordu.

- Verimlilik: İşlemlerin hızlı ve zamanında yapılması gerekiyordu.

- Doğrulama Maliyeti (Validation Costs): AVS operatörlerinin simülasyon ve doğrulama yapmasının maliyetini (gas ücretleri ve zaman) makul tutmak gerekiyordu.