# 004 - Symbiote

Project Link: [Symbiote Hook](https://github.com/0xCamax/SymbioteHook)

Proje verimli likidite sağlamak için AAVE entegrasyonu yapmayı öne süren bir V4 Hook projesi.

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Uniswap V4    │    │  SymbioteHook   │    │   Aave Pool     │
│   Pool Manager  │◄──►│   (JIT Logic)   │◄──►│   (Lending)     │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```
## 1. Problem

Diyelim ki elinde 10.000 doların var. Şu anki sistemde iki seçeneğin var:

- **Seçenek A (Uniswap):** Paranı bir döviz bürosuna (Uniswap) koyarsın. İnsanlar senin paranla dolar/euro alıp satar, sen de işlem ücreti kazanırsın. Buna Likidite Sağlamak (LP) denir. Ama paran sadece orada durur, başka hiçbir şey yapamaz.

- **Seçenek B (Aave):** Paranı bir bankaya (Aave) faize yatırırsın. Faiz kazanırsın ama bu sefer de işlem ücreti kazanamazsın.

Mevcut yöntemlerde lending ve likiditeyi birbirine bağlamak çoklu adımdan (Multi Step) oluşan kompleks bir işlem. Bu sebeple kullanıcılar için cazip değiller.

Symbiote, Uniswap'in içine Hook tanımlıyor, sen paranı Uniswap'e koyduğunda;

1. Sistem arka planda bu parayı Aave ile konuşturuyor.

2. Paran Aave'de "varlık" olarak görünüp faiz getirisi sağlayabiliyor veya Aave'den borçlanma (leverage) için teminat olarak kullanılabiliyor.

3. Aynı anda Uniswap üzerinde alım-satım işlemlerine de likidite sağlıyor. Çifte Kazanç (Dual Yield)

Bu proje sayesinde kullanıcı tek bir işlem yapıyor (Single Step). Ama arka planda akıllı kontratlar hem bankacılık (Aave) hem de borsa (Uniswap) işlemlerini birleştiriyor.

Bunun kullanıcıya faydası ise raporda "Dual Yield" olarak geçen Çifte Getiri:

- **Aave Lending Interest:** Paranız içeride durduğu için faiz kazanırsınız.

- **Uniswap LP Fees:** Paranızla al-sat yapıldığı için komisyon kazanırsınız.

## 2. Etki

**Basitleştirilmiş Kullanıcı Deneyimi (Simplifies UX):**

- Tek bir işlem yaparsın (One Action). Arka planda Hook sistemi borçlanmayı ve likidite eklemeyi senin yerine halleder.

**Artırılmış Getiri (Boosts Yield):**

- Kullanıcılar "Çifte Getiri" (Double Dip) yapar. Hem Aave'ye para koymuş gibi faiz alırlar, hem de Uniswap'te işlem yapılıyor gibi komisyon alırlar.

**Erişimi Genişletmek (Expands Access):**

- Genelde kaldıraçlı işlem yapmak karmaşık matematik ve sürekli takip gerektirdiği için "Balina" dediğimiz büyük/uzman oyuncuların işidir. Symbiote, bu karmaşık stratejiyi basitleştirerek küçük bütçeli kullanıcıların (smaller users) da paralarını kaldıraçla büyütüp likidite sağlamasına olanak tanır.

## 3. Challenge

Uniswap ve Aave arasında likidite, teminat ve borçlanmayı, istismarlara maruz kalmadan senkronize etmek büyük bir zorluktu. Başlangıçta eksiksiz bir protokol oluşturmak hedeflenmiş, ancak daha sonra kendi kendini yöneten bir hook yaklaşımına geçilmiş. Karmaşıklık, her kullanıcının getirisini, borcunu ve likiditesini zaman içinde doğru bir şekilde izlemekten kaynaklanıyormuş.

## 4. JIT Operation Flow
**Before Swap (beforeSwap):**

- Hook detects incoming swap
- Calculates required liquidity windows
- Temporarily adds JIT liquidity to the pool
- Stores active liquidity reference in transient storage

**After Swap (afterSwap):**

- Verifies slippage protection conditions
- Removes JIT liquidity from the pool
- Settles any token imbalances through Aave
- Returns excess tokens to Aave for yield