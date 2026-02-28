# Bölüm 1: Bilgisayar Mimarisine Giriş

Bu bölümde yazılımın donanıma nasıl dönüştüğünü, bellek yapısını ve bir komutun işlemci tarafından nasıl yürütüldüğünü inceliyoruz.

## 1. Programlama Dilleri ve Derleme (Compilation) Süreci
Bilgisayarlar temelde sadece "açık/kapalı" (1/0) sinyallerini anlar. İnsan mantığını bu sinyallere dönüştürmek için katmanlı bir yapı kullanılır:

### Yüksek Seviyeli Diller (High-Level):

* Java, Python gibi dillerdir.
* **Kompleks işlemler**, insanın anlayabileceği kelimeler ve matematiksel ifadelerle yazılabilir.
* **Donanımdan bağımsızdır**.

### Alçak Seviyeli Diller (Low-Level):

* Assembly, temelde makine dilidir.
* **Yapılan işlemler çok basittir**. (Örn: bellekteki şu adresten sayıyı al, şuraya koy)
* Program yazımı çok zordur. **Donanıma çok yakındır**. Bilgisayarın işlemcisinin mimarisini bilmen gerekir.

**Not:** C dili bazen "Mid-level" (Orta seviye) olarak da anılır. Çünkü C, pointerlar aracılığıyla belleğe doğrudan erişebilir.

### Derleme (Compilation) Süreci
Yüksek seviyeli bir dilde yazılan kod sırasıyla şu aşamalardan geçerek donanıma ulaşır:
`High-level language -> Compiler -> Assembly language -> Assembler -> Machine language`

## 2. Bilgisayar Mimarisi ve ISA
Assembly dilinde programlama bilgisayar mimarisine bağlı olarak değişiklik gösterir (ISA).

**ISA (Instruction Set Architecture):** Bir işlemcinin hangi komutları, veri tiplerini anlayabileceğini belirleyen bir "sözleşme" gibidir.

### RISC ve CISC Yaklaşımları
İşlemciler felsefelerine göre ikiye ayrılırlar:

**RISC (Reduced Instruction Set Computer):** MIPS bu mimarinin en popüler örneğidir.

* Felsefesi: "Az sayıda, basit ve çok hızlı çalışan komutlar."
* Komut boyutu sabittir, bu da işlemcinin komutları daha hızlı çözmesini (decoding) sağlar.

**CISC (Complex Instruction Set Computer):** Intel (x86) ve AMD bu mimariyi kullanır.

* Felsefesi: "Karmaşık ve tek seferde çok iş yapan komutlar."

Her işlemci ailesi kendi "dilini" konuşur. Bir Intel işlemci için yazılmış makine kodu, bir MIPS işlemcide çalışmaz. **ISA markaya/aileye özeldir.** Farklı ISA'lar farklı sayıda register'a, farklı adresleme yöntemlerine ve farklı komut setlerine sahiptir.

### Neden MIPS?

* **Sabit Komut Boyutu:** Tüm komutlar 32-bittir. Bu sayede donanım, komutun nerede başlayıp bittiğini çözmek için fazladan efor sarf etmez; bu da tasarımı basit ve ucuz kılar.
* **Düzenli Yapı:** Toplama (add) veya çıkarma (sub) gibi işlemler her zaman 3 operanda (2 kaynak, 1 hedef) ihtiyaç duyar. Örn: `add a, b, c`
* **Küçük Olan Daha Hızlıdır:** Veriler doğrudan yavaş olan ana bellekte değil, işlemci içindeki sadece 32 adet olan çok hızlı Register (Yazmaç) birimlerinde işlenir. Register sayısının az tutulması, elektrik sinyallerinin katettiği mesafeyi azaltarak hızı artırır.
* **Sık Karşılaşılan Durumu Hızlandır:** Programlarda çok sık kullanılan küçük sabit sayılar doğrudan komutun içine gömülür. Böylece bu sayılar için tekrar tekrar belleğe gitmeye gerek kalmaz.

Şimdi bir örnek yapalım. Güncel skor tutarını hesaplayan basit bir C dili kodu yazalım:
`skor = taban + bonus - ceza;`

MIPS mimarisinde veriler doğrudan bellekte değil, işlemcinin içindeki çok hızlı yazmaçlarda işlenir. Bu yüzden derleyici, C kodundaki değişkenleri önce bu yazmaçlara atar. Genellikle kalıcı değişkenler için `$s` (saved) yazmaçları, ara işlemler için `$t` (temporary) yazmaçları kullanılır:

* `skor` değişkeni -> `$s0`
* `taban` değişkeni -> `$s1`
* `bonus` değişkeni -> `$s2`
* `ceza` değişkeni -> `$s3`

Şimdi işlemleri küçük parçalara bölelim:

```mips
add $t0, $s1, $s2   # $t0 = taban + bonus 
sub $s0, $t0, $s3   # $s0 = $t0 - ceza
