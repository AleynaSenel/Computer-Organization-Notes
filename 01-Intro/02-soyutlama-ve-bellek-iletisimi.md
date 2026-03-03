# Bölüm 2: Soyutlama Katmanları ve Bellek İletişimi

Bu bölümde karmaşık donanım yapılarının nasıl soyutlandığını ve işlemcinin bellek ile nasıl haberleştiğini inceliyoruz.

## 1. MIPS RISC Mimarisinde Soyutlama (Abstraction) Katmanları

MIPS'in kendi içinde bile "anlaşılabilirlik" için katmanlara ayrıldığını görüyoruz.

Dilleri en "ilkel" olandan en "gelişmiş" olana doğru sıralayalım:

*  **MIPS RISC Makine Dili:** En içteki halka. Sadece 0 ve 1'ler.
   
*  **TAL (True Assembly Language):** ISA'daki gerçek komutlardan oluşur. TAL'dan daha aşağı seviyede (0-1'ler hariç) programlama yapılamaz. İşlemcinin tam olarak ne yapacağını birebir söyler.
  
*  **MAL (More Abstract Language): MIPS Assembly dili** olarak da bilinir. Programcıya bazı kolaylıklar sağlar. Assembler, MAL programını önce TAL'a, sonra makine koduna çevirir. MAL programlar da TAL kadar etkindir.
  
*  **SAL (Simple Abstract Language):** En dıştaki halka. MAL'dan daha yüksek seviyelidir. Yüksek seviyeli dillerle (C, Java gibi) Assembly dili arasındaki boşluğu doldurmak için kullanılır.
  
 

  <img width="523" height="534" alt="image" src="https://github.com/user-attachments/assets/69f8117d-c486-4393-895b-1acd30960b3f" />

  
  

### TAL (True Assembly Language)
**Varoluş amacı: Karmaşıklığı gizlemek değil, makinenin anladığı 0 ve 1'leri (Makine Dili) insanların okuyabileceği kelimelere dönüştürmektir.**

**Nasıl yapar?** TAL aslında donanımın ta kendisidir, çıplak gerçekliktir. Senden hiçbir donanım kısıtlamasını gizlemez. Ancak, 00000010000100000100000000100000 gibi korkunç bir 0-1 dizilimi görmek yerine, en azından `add $t0, $s0, $s1` gibi harflerden oluşan (mnemonics) komutlar görmeni sağlar.



### MAL (More Abstract Language - MIPS Assembly)
**Varoluş amacı: TAL'ın (donanımın) katı ve acımasız kurallarını esneterek, Assembly seviyesinde programlama yapmayı insanlar için katlanılabilir, hızlı ve daha az hataya açık hale getirmektir.**

**Nasıl yapar?** Donanım kısıtlamaları yüzünden TAL'da 3-4 satırda yazman gereken sıkıcı işlemleri, **sözde komutlar (pseudoinstructions)** sayesinde tek satırda halletmeni sağlar.
Örneğin, bir veriyi başka bir yere kopyalamak için TAL seviyesinde doğrudan bir move komutu yoktur (bunun yerine matematiksel `add` komutunu sıfırla toplayarak hileli bir şekilde kullanır). Ama MAL sana `move` komutunu sunar. Sen sadece move yazarsın, MAL onu TAL'ın anlayacağı o karmaşık şekle sessizce çevirir. Yani, seninle donanım arasında mükemmel bir tampon bölge oluşturur.



### SAL (Simple Abstract Language)
**Varoluş amacı: Donanımı senden tamamen gizlemek ve sadece çözdüğün probleme odaklanmanı sağlamaktır.**

**Nasıl yapar?** SAL katmanındayken işlemcinin içinde yazmaç (register) mı var, bellek adresi neresi, veriler nasıl taşınıyor hiç umursamazsın. Sen sadece insan mantığına en yakın şekilde "Eğer kullanıcı şifreyi doğru girerse giriş yap" `(if password == true)` dersin.


  Bu üçlüyü yan yana koyduğumuzda ortaya şöyle bir tablo çıkıyor:
  
  **SAL:** İnsan mantığına en yakın olan (Problemi çözer).
  
  **MAL:** İnsan ile donanım arasında köprü kuran (İşleri kolaylaştırır).
  
  **TAL:** Makineye en yakın olan (Donanımın gerçek kapasitesini kullanır).


### Neden Bu Kadar Çok Katman Var?
Mühendislikte bir iş ne kadar zorsa, onu o kadar çok **soyutlama (abstraction)** katmanına böleriz. Makine dilinde kod yazmak imkansız gibidir, bu yüzden TAL'ı; TAL çok kısıtlı olduğu için MAL'ı; MAL ile uğraşırken de işleri daha da basitleştirmek için SAL'ı kullanırız.


## 2.İşlemci ve Bellek İlişkisi
Programın çalışması sırasında Merkezi İşlem Birimi (CPU) komutları yürütür. CPU, bellek (Memory) ile sürekli iletişim halindedir:

### Bellek (Memory/RAM) Mantığı

**Sabit ve Geniş:** Bellek boyutu sabittir ama oldukça geniştir.
**Değişkenler ve Adresler:** Programda tanımladığın her bir değişkene (variable) karşılık bir **bellek gözü (memory location)** tahsis edilir. 
**Binding:** Bir bellek gözünün bir değişkene ayrılması işlemine biz **"binding"** diyoruz. 


### CPU ve Bellek Arasındaki İletişim
İşlemci (CPU) ve Bellek (Memory) sürekli bir alışveriş içindedir: 
**Read (Okuma/Load):** Bir değişkenin değerini öğrenmek, o bellek gözünü okumak demektir. 
**Write (Yazma/Store):** Bir değişkene yeni değer atamak, o bellek gözüne yazmak demektir.

### MIPS Neden "Load/Store Mimarisi" Olarak Anılır?
Yazdığın o **Load (Okuma)** ve **Store (Yazma)** kısmı MIPS için hayati önem taşır. Neden mi? Çünkü CPU (İşlemci) biraz "tembel" ama inanılmaz hızlı bir patrondur. CPU, gidip o devasa ve yavaş Bellek (RAM) üzerinde doğrudan toplama, çıkarma veya çarpma işlemi yapmayı reddeder.

Eğer bir işlem yapılacaksa, MIPS işlemcisinin katı bir kuralı vardır:

1. Önce veriyi bellekten CPU'nun içine getir **(Load)**.

2. İşlemi CPU'nun kendi içindeki çok hızlı küçük odacıklarda (Yazmaçlar) yap.

3. Çıkan yeni sonucu tekrar belleğe geri götür ve yaz **(Store)**

**NOT:** Bilgisayar ne kadr hızlı olursa olsun eğer CPU ile bellek arasındaki bu trafik kapalıysa bilgisayar yavaşlar. Buna **Von Neumann Darboğazı(Von Neumann Bottleneck)** denir. Yani veriyi belleğe Load/Store yapmak işlemcinin içindekilerden daha maliyetli bir iştir.


## 3. Bir Komutun Çalışma Döngüsü
Şimdi `add $s1, $s2, $s3` komutunu örnek alarak işlemcinin arka planda sırasıyla attığı bu adımlara yakından bakalım:

**1. Instruction Fetch (Komut Getirimi):** Komut bellekten (RAM) CPU'ya getirilir.

**2. Program Counter (PC) Update:** İşlemci, bir sonraki komutun nerede olduğunu unutmamak için adres sayacını bir sonraki komuta günceller.

**3. Instruction Decode (Komut Çözümü):** CPU, "Bu komut `add` mi, `sub` mu, yoksa `beq` mi?" diye anlamaya çalışır.

**4. Operand Load (Operand Yükleme):** İşlem yapılacak sayılar (mesela x2 ve x3) ilgili yerlerden getirilir.

**5. Operation Execution (İşlem Yürütümü):** Asıl işlem (toplama, çıkarma vb.) yapılır.

**6. Storage of Results (Sonuçların Saklanması):** Çıkan sonuç hedef değişkene veya belleğe kaydedilir.


<img width="624" height="277" alt="image" src="https://github.com/user-attachments/assets/05be9052-84fe-444d-9c12-01a5cc90b495" />


