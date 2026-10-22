# Güç Devresi Tasarımı İçin Matematiksel Hesaplamalar (Design Calculations)

Bu dosya, (TPS54202) entegresine dayalı referans güç kartı (TIDA-00946) tasarımındaki elektronik bileşenlerin gerçek değerlerinin seçildiği matematiksel kanunları ve mühendislik denklemlerini belgelemektedir. Projemiz için temel veriler şunlardır:
-   **Nominal Giriş Gerilimi ($V_{IN}$):** $12V$ ($6.5V$ ile $20V$ aralığında çalışma yeteneğiyle)
-   **İstenen Çıkış Gerilimi ($V_{OUT}$):** $5V$
-   **Maksimum Çıkış Akımı ($I_{OUT}$):** $1A$
-   **Anahtarlama Frekansı ($f_{sw}$):** $500kHz$ (Çip içinde sabit değer)
-   **Entegrenin Dahili Referans Gerilimi ($V_{REF}$):** $0.596V$

---

## 1. Geri Besleme Gerilim Bölücüsünün Hesaplanması (Feedback Voltage Divider)

**Hedef:** Referans kart tasarımcılarının seçtiği değerlerle eşleşerek, çıkış geriliminin tam olarak $5V$'ta sabit kalmasını sağlamak için üst ($R_3$) ve alt ($R_6$) dirençlerin değerlerini belirlemek.

### Matematiksel Formül (The Formula)
Entegre, gerilim bölücüden gelen gerilimi dahili referans gerilimiyle karşılaştıran bir hata yükseltecine dayanır. Çıkış gerilimi denklemi şu şekilde verilir:

$$V_{OUT} = V_{REF} \times \left( 1 + \frac{R_3}{R_6} \right)$$

Üst direncin ($R_3$) değerini alt direnç biliniyorken bulmak için denklemi yeniden düzenliyoruz:

$$R_3 = R_6 \times \left( \frac{V_{OUT}}{V_{REF}} - 1 \right)$$

### Pratik Uygulama ve Tasarımcının Seçimleri (Application & Designer's Choice)
1. **Alt Direnç Değerinin Seçimi ($R_6$):** 
   Mühendislik açısından alt direnç için herhangi bir değer seçilebilir. Küçük değerler (örneğin $10k\Omega$) gürültüye karşı mükemmel bir kararlılık sağlar, ancak çıkıştan toprağa doğru sürekli bir akım çeker, bu da hafif yüklerde (Light Load Efficiency) verimliliği azaltır. 
   Çipin verimliliğini en üst düzeye çıkarmak ve güç tasarrufu modunu en iyi şekilde etkinleştirmek için tasarımcı, alt direnç için daha yüksek standart bir değer seçmiştir: **$R_6 = 26.7 k\Omega$**.

2. **Üst Direncin Hesaplanması ($R_3$):**
   Gerekli üst direnci hesaplamak için çıkarılan denklemde tasarımcının değerini yerine koyarsak:
   $$R_3 = 26,700 \times \left( \frac{5}{0.596} - 1 \right)$$
   $$R_3 = 26,700 \times (8.3892 - 1)$$
   $$R_3 = 26,700 \times 7.3892 \approx 197,291 \Omega \approx 197.3 k\Omega$$

3. **Standart Direnç Değerinin Seçimi (Standard 1% Resistor):**
   Piyasada $197.3k\Omega$ için mevcut en yakın standart değer (%1 toleranslı E96 serisi) **$200 k\Omega$**'dur.
   
   Bu dirençler ($R_3 = 200 k\Omega$ ve $R_6 = 26.7 k\Omega$) kullanıldığında gerçek çıkışın doğruluğunu kontrol etmek için:
   $$V_{OUT(Actual)} = 0.596 \times \left( 1 + \frac{200}{26.7} \right)$$
   $$V_{OUT(Actual)} = 0.596 \times (1 + 7.4906) = 0.596 \times 8.4906 = 5.06 V$$
   Gerçek gerilim ($5.06V$) son derece hassas kabul edilir ve hassas dijital yükleri beslemek için kabul edilebilir tolerans (Tolerance) sınırı içindedir.

**Bölüm 1 İçin Nihai Sonuç:**
-   **Üst Direnç $R_{FB\_Upper} (R_3) = 200 k\Omega$**
-   **Alt Direnç $R_{FB\_Lower} (R_6) = 26.7 k\Omega$**


## 2. Etkinleştirme Gerilim Bölücüsü ve Düşük Gerilim Korumasının Hesaplanması (Enable Divider & UVLO)

**Hedef:** `EN` etkinleştirme pinine bağlı üst ($R_2 = 562 k\Omega$) ve alt ($R_5 = 102 k\Omega$) dirençleri için referans kartta tasarımcının seçtiği değerleri analiz etmek ve sistemin anahtarlamaya başlamasına izin vereceği "Güvenli Çalıştırma Eşiği"ni (Turn-on Voltage - $V_{START}$) çıkarmak.

### Matematiksel Formül (The Formula)
`EN` etkinleştirme pini, bu entegre ailesinde yaklaşık ($V_{EN\_Threshold} \approx 1.21V$) olan hassas bir referans çalıştırma eşiğine sahip dahili bir gerilim karşılaştırıcısına dayanır. 
Hesaplamayı basitleştirmek için (mikroamper düzeyindeki küçük dahili histerezis akımları ihmal edilerek), toplam sistem başlatma gerilimi ($V_{START}$) denklemi şu şekilde verilir:

$$V_{START} = V_{EN\_Threshold} \times \left( 1 + \frac{R_2}{R_5} \right)$$

### Pratik Uygulama (Application & Calculation)
Pratik şemada bulunan gerçek değerleri yerine koyarsak:
$$V_{START} = 1.21 \times \left( 1 + \frac{562}{102} \right)$$
$$V_{START} = 1.21 \times (1 + 5.5098)$$
$$V_{START} = 1.21 \times 6.5098 \approx 7.87 V$$

### Tasarımcının Seçimine Dair Mühendislik Notları (Engineering Notes on Designer's Choice)
Hesaplamalardan devrenin ancak gerilim yaklaşık ($7.87V$)'a ulaştığında başlayacağı görülmektedir; oysa (TPS54202) çipi teorik olarak $4.5V$ veya $6.5V$'tan başlayan bir gerilimle çalışabilme yeteneğine sahiptir. Bu kısıtlama kasıtlıdır ve iki mühendislik gerekçesine dayanmaktadır:

1. **Doğrusal Regülatör Simülasyonu (7805 Drop-in Replacement):** Baskılı devre kartı analizinde gösterildiği gibi, bu modül eski doğrusal regülatör (7805) için doğrudan bir alternatif olacak şekilde tasarlanmıştır. Fiziksel olarak, 7805 regülatörü düzgün çalışabilmesi için yaklaşık $2V$ ila $2.5V$'luk bir gerilim düşümüne (Dropout Voltage) ihtiyaç duyar (yani giriş $7.5V$ veya üzerinde olmadıkça kararlı bir $5V$ vermez). Modern anahtarlamalı kartın da tam olarak uyumlu bir alternatif gibi davranabilmesi için tasarımcı, gerilim $7.8V$'un altına düştüğünde kartı kasıtlı olarak kapatmıştır.
2. **12V Sistemlerin Koruması (12V Systems Protection):** Bu kartlar genellikle pillerle veya $12V$ besleme hatlarıyla kullanılır. Gerilimin $6.5V$ civarına düşmesi, çoğu sistemde pilin neredeyse boş (arızalı) olduğu veya bir yerlerde bu tehlikeli düşüşe neden olan bir kısa devre olduğu anlamına gelir. Bu nedenle, başlatma eşiğinin $7.87V$'a ayarlanması, sistemin kararsız güç koşullarında çalışmasını ve yüksek akım çekmesini engelleyen, böylece devrede dalgalanmayı (Chattering) önleyen mükemmel bir koruma önlemidir (UVLO).

**Bölüm 2 İçin Nihai Sonuç:**
-   **Güvenli Çalıştırma Eşiği ($V_{START}$): $\approx 7.87 V$**
-   **Bu eşik şu direnç ağı ile sağlanmıştır: $R_2 = 562 k\Omega$ ve $R_5 = 102 k\Omega$**


## 3. Çıkış ve Yumuşatma Bobini Değerinin Hesaplanması (Output Inductor - L1)

**Hedef:** Çıkış bobininin ($L_1$) matematiksel değerini çıkarmak ve tasarımcının pratik şemadaki standart ($22\mu H$) değerini seçmesine iten mühendislik nedenini kanıtlamak.

### Matematiksel Formül (The Formula)
Düşürücü dönüştürücülerde (Buck Converters) bobin değeri, "Dalgalanma Akımı"nın (Inductor Ripple Current - $\Delta I_L$) belirli sınırları aşmamasını sağlamak için hesaplanır. Bobin değerini hesaplamak için standart mühendislik denklemi şudur:

$$L = \frac{V_{OUT} \times (V_{IN} - V_{OUT})}{V_{IN} \times \Delta I_L \times f_{sw}}$$

Burada:
-   $V_{IN}$: Giriş gerilimi (Nominal gerilim olan $12V$'u kullanacağız).
-   $V_{OUT}$: İstenen çıkış gerilimi ($5V$).
-   $f_{sw}$: Entegrenin anahtarlama frekansı ($500kHz$ veya $500,000 Hz$).
-   $\Delta I_L$: Bobin içindeki izin verilen dalgalanma akımı.

**$\Delta I_L$ nasıl belirlenir?**
Mühendislik açısından, güç kaynakları tasarımındaki altın kural, maksimum çıkış akımının ($I_{OUT}$) %20'si ile %40'ı arasında değişen bir dalgalanma akımı seçmektir. 
Devremizin maksimum çıkış akımı $1A$ olduğundan, ideal ve dengeli bir oran olan %30'u (yani katsayı $K_{IND} = 0.3$) seçeceğiz.
Böylece:
$$\Delta I_L = 1A \times 0.3 = 0.3 A$$

### Pratik Uygulama (Application & Calculation)
Tüm değerleri temel denklemde yerine koyarsak:

$$L = \frac{5 \times (12 - 5)}{12 \times 0.3 \times 500,000}$$

$$L = \frac{5 \times 7}{12 \times 150,000}$$

$$L = \frac{35}{1,800,000}$$

$$L \approx 0.0000194 H \approx 19.4 \mu H$$

### Tasarımcının Seçimine Dair Mühendislik Notları (Engineering Notes on Designer's Choice)
Kesin hesaplama sonucu, $19.4 \mu H$ değerinde bir bobine ihtiyacımız olduğunu göstermiştir. Ancak elektronik dünyasında bileşenler sabit standart değerlerde (Standard Values) üretilir. 

1. **En Yakın Üst Standart Değere Yuvarlama:** Piyasada ticari olarak bulunan en yakın standart değerler $15\mu H$ ve $22\mu H$'dir. Tasarımcı en yüksek değeri, yani **$22\mu H$**'yi seçmiştir.
2. **Mühendislik Gerekçesi:** Daha yüksek değere (19.4'ten 22'ye) yuvarlama çok zekice bir mühendislik kararıdır; çünkü bobinin değerini artırmak fiziksel olarak "dalgalanma akımını" ($\Delta I_L$), varsaydığımız orandan (%30) daha fazla azaltır. Dalgalanmaların azaltılması, çıkışta daha kararlı ve saf bir doğru akım (DC) elde etmek anlamına gelir ki bu da gömülü sistemlerdeki hassas yükleri beslemek için son derece önemlidir.

**Bölüm 3 İçin Nihai Sonuç:**
-   **Teorik Olarak Hesaplanan Değer:** $19.4 \mu H$
-   **Tasarım İçin Seçilen Gerçek Değer:** $L_1 = 22 \mu H$


## 4. Çıkış Kondansatörleri Kapasitesinin Hesaplanması (Output Capacitors - C4 & C5)

**Hedef:** Gerilim kararlılığını sağlamak için gereken minimum çıkış kondansatörü ($C_{OUT}$) kapasitesini çıkarmak ve tasarımcının her biri ($22\mu F$) değerinde paralel iki kondansatör (toplamda $44\mu F$) kullanmayı tercih etmesinin mühendislik gerekçesini açıklamak.

### Matematiksel Formül (The Formula)
Çıkış kondansatörünün kapasitesi, "Çıkış Geriliminde İzin Verilen Dalgalanma"ya (Output Voltage Ripple - $\Delta V_{OUT}$) göre hesaplanır. Seramik kondansatörlerin (ESR direncinin neredeyse sıfır olduğu) kullanıldığı durumda, gereken minimum kapasite denklemi şu şekilde verilir:

$$C_{OUT(min)} = \frac{\Delta I_L}{8 \times f_{sw} \times \Delta V_{OUT}}$$

Burada:
-   $\Delta I_L$: Daha önce hesapladığımız bobin dalgalanma akımı ($0.3 A$).
-   $f_{sw}$: Anahtarlama frekansı ($500,000 Hz$).
-   $\Delta V_{OUT}$: İzin verilen maksimum gerilim dalgalanması. Mühendislik açısından, hassas yükler için dalgalanmanın çıkış geriliminin ($5V$) %0.5 ile %1'ini geçmemesi tercih edilir. Biz çok sıkı bir dalgalanma olan $25mV$ (yani $0.025V$) varsayacağız.

### Pratik Uygulama (Application & Calculation)
Değerleri denklemde yerine koyarsak:

$$C_{OUT(min)} = \frac{0.3}{8 \times 500,000 \times 0.025}$$

$$C_{OUT(min)} = \frac{0.3}{100,000} = 0.000003 F = 3 \mu F$$

### Tasarımcının Seçimine Dair Mühendislik Notları (Engineering Notes on Designer's Choice)
Salt teorik hesaplamalar, dalgalanmaları yumuşatmak için sadece $3 \mu F$ kapasiteli bir kondansatöre ihtiyacımız olduğunu göstermiştir! Ancak pratik şemaya baktığımızda, tasarımcının her biri $22\mu F$ olan (toplam $44\mu F$) iki kondansatör ($C_4$ ve $C_5$) kullandığını görüyoruz. 
Teorik hesaplama ($3\mu F$) ile pratik uygulama ($44\mu F$) arasındaki bu büyük farkın üç kritik mühendislik nedeni vardır:

1. **Sürekli Ön Gerilim Düşüşü Etkisi (DC Bias Derating):** İç yapı analizinde bahsettiğimiz gibi, seramik kondansatörler (MLCC) üzerlerine doğru akım (DC) uygulandığında gerçek kapasitelerinin büyük bir kısmını kaybederler. Nominal değeri $22\mu F$ olan bir kondansatörün gerçek kapasitesi, üzerine sürekli olarak $5V$ uygulandığında $10\mu F$ veya daha altına düşebilir. Bu nedenle tasarımcı, bu kaçınılmaz kaybı telafi etmek için kapasiteyi iki katına çıkarmıştır.
2. **Geçici Yüke Yanıt (Transient Response):** Önceki denklem "Kararlı Durum" (Steady-State) koşulunu hesaplar. Ancak sistem aniden yüksek akım çekerse ne olur? Gerilimde ani bir düşüş (Voltage Dip) meydana gelecektir. Büyük kapasite ($44\mu F$), dahili hata yükselteci yanıt vermeden önce bu eksikliği telafi etmek için mikrosaniyeler içinde müdahale eden büyük bir enerji deposu olarak görev yapar ve böylece sistemin kararlılığını korur.
3. **Toplam ESR Direncinin Azaltılması:** Paralel iki kondansatör yerleştirmek, iç direnci (ESR) yarıya indirir, bu da harcanan ısıyı azaltır ve yüksek frekans filtreleme işlemini tek bir büyük kondansatör kullanmaktan çok daha verimli hale getirir.

**Bölüm 4 İçin Nihai Sonuç:**
-   **Teorik Olarak Hesaplanan Minimum Kapasite:** $3 \mu F$
-   **Tasarım İçin Seçilen Gerçek Kapasite:** $C_4 + C_5 = 44 \mu F$ (DC Bias etkisinin üstesinden gelmek ve geçici yanıtı iyileştirmek için).


## 5. Sonuç ve Mühendislik Kararı (Engineering Verdict)

Bu dosyada belgelenen hesaplamalar, güç kartı için değerlerin ve elektronik bileşenlerin seçiminin rastgele olmadığını; saf teorik performans ile gerçek dünya çalışma ortamı gereksinimlerini dengeleyen hassas mühendislik standartlarına dayandığını kanıtlamaktadır. 

Bu hesaplama dizisi aracılığıyla, tasarımın güvenliği ve güvenilirliği çeşitli temel yönlerden doğrulanmıştır:
- **Mutlak Regülasyon Hassasiyeti:** Geri besleme ağının hassas ayarı sayesinde, dijital yükleri beslemek için çıkış geriliminin tam olarak $5V$'ta sabit kalmasını sağlamak.
- **Güvenilirlik ve Proaktif Koruma:** Devre dalgalanmasını önlemek ve düşük güç koşullarında güç kaynağını aşırı akım çekiminden korumak için sistemin başlatma eşiğini (UVLO) önceden $7.87V$'a ayarlamak.
- **Güç Kalitesi ve Safiyeti (Power Quality):** Dalgalanma akımını en aza indirmek için $22\mu H$ değerinde bir yumuşatma bobini seçmek ve akım çekimindeki ani ve şiddetli değişiklikleri (Transient Response) emmek ve kapasite düşüşü (DC Bias Derating) olgusunun tamamen üstesinden gelmek için önleyici bir mühendislik önlemi olarak çıkış kondansatörlerinin kapasitesini $44\mu F$'a çıkarmak.

Bu titiz matematiksel metodoloji, düşürücü dönüştürücü ünitenin (TPS54202) sürekli, saf ve yüksek verimli güç sağlama yeteneğini garanti ederek, performans dalgalanması veya termal çökme endişesi olmadan bu sistemdeki karmaşık sensörleri çalıştırmak için sağlam bir mühendislik temeli oluşturur.