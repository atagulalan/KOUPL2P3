Para Bozma Makinesi

Ata Gülalan		-	160202034
Oğuzhan Türker	-	160202015

Bu readme.txt dosyası, Para Bozma Makinesi projesine aittir.
Bu paket, kaynak kodu ile aynı dizin içerisinde bulunacaktır.


1-PAKETİN İÇERİĞİ:
----------
160202034-160202015.txt - Projenin tek dosyaya indirgenmiş salt kaynak kodu.
readme.txt - Bu dosya.
kaynak.zip - Projenin kaynak kodunun ve yardımcı dosyaların ziplenmiş hali.
rapor.pdf - Proje raporu.
----------


2-SİSTEM GEREKSİNİMLERİ:
-------------------
Oracle VM virtualbox - https://www.virtualbox.org/
Sanal Makine İmajı   - https://github.com/KOU-Embedded-System-Lab/os-base-image/releases
-------------------


3-PROJEYİ ÇALIŞTIRMAK:
-------------------
Paket içeriğini yukarıda görebilirsiniz.

Bu kod, 2 adet önceden tanımlanmış sanal makinelerde çalıştırıldı.

Sanal makineyi indirmek için;
https://github.com/KOU-Embedded-System-Lab/os-base-image/releases
adresini ziyaret edebilirsiniz.

Bu iki durumda da, kod, herhangi bir hata vermeksizin, daha önceden
belirlenen kriterlere uygun çalıştı.


4-KODU DERLEMEK:
------------------
Artık bilgisayarımızda kurulu olan sanal makine ile kodu kolayca derleyebiliriz.

Projeyi geliştirim kartında çalıştırmak için kodu daha önceden oluşturulmuş Tiva C
projesinde yer alan "main.c"ye yapıştırıp build butonuna tıkladıktan sonra debug 
butonuna tıklamanız yeterli.

Eğer kart, sanal makine tarafından bulunamadıysa (OpenOCD hatası) sanal makineye
bağlı olan bir kart olmayabilir. Sağ alt kısımdan usb ikonuna sağ tıklayarak 
Texas Instruments'ı seçip sanal makineye bağlamanız gerekmektedir.

Ana Makinesi GNU/Linux olan makinelerde VirtualBox, kullanıcı izinlerine sahip
olmayabiliyor. Eğer takılı olan USB aygıtları tanımıyorsa, 
'sudo usermod -G vboxusers -a $USER'
kodunu çalıştırmanız ve sisteminizi yeniden başlatmanız gerekmektedir. 
------------------


5- PARAMETRELER
---------------------------
Kodun çalışması için başlangıçta herhangi bir parametre gerekmiyor.
------------------


6- PROGRAMIN KULLANIMI
-----------------------------
Para Bozma Makinesi, kullanıcıdan, butonlar yardımı ile girdi alarak
girilen miktarın yirmilik, onluk, beşlik, birlik, yarımlık, çeyreklik,
metelik, delik ve kuruşluk cinsine bölünmesini sağlar.

Programa maksimum 99.99 miktarı girilebilir.

Birinci, ikinci, dördüncü ve beşinci butonlara, istenen basamak maksimum
eşik değerine (9) ulaştıktan sonra basılması bu basamağın sıfırlanmasını
sağlar.

İlk buton, onlar basamağını girmek için kullanılır. Eğer halihazırda
ikinci butona basılarak birler basamağı girilmemiş ise, birler basamağına
sıfır atayarak 10'un katları halinde gitmesini sağlar.

İkinci buton, birler basamağını girmek için kullanılır.

Üçüncü buton, girilen sayının ondalıklı olup olmayacağının girilmesine
yarar. Aynı zamanda bu butona 5 saniyeden uzun basmak, sistemin yeniden
başlamasını (reset) sağlar.

Dördüncü buton, eğer kullanıcı 3. butona bir kere basarak nokta koyup
sayıyı ondalıklı hale getirmiş ise, onda birler basamağını girmek için
kullanılır.

Beşinci buton da dördüncü butonda olduğu gibi, eğer kullanıcı 3. butona
bir kere basarak nokta koyup sayıyı ondalıklı hale getirmiş ise, yüzde 
birler basamağını girmek için kullanılır.

Eğer girilen değer sıfırdan farklı ise, daha önceden bir çözümleme
yapılmamış (reset beklenmiyor) ve herhangi bir butona 5 saniye tıklanmamış
ise çözümleme yapılmaya başlanır. Çözümleme, en yüksek bölünebilir sayıya
ve bu sayının, verilen miktarın içerisinde kaç adet bulunduğuna göre
değişiklik gösterir. Örneğin girilen değer 39.94 ise program size bir adet
yirmilik, bir adet onluk, bir adet beşlik, dört adet birlik, bir adet
yarımlık, bir adet çeyreklik, bir adet metelik, bir adet delik, dört
adet kuruşluk olarak böler.

Üçüncü butona herhangi bir anda (veri girişi, çözümleme veya bitiş ekranı)
5 saniyeden daha fazla basılı tutmak programı resetler.

Veri girişi ekranındaki 5 saniye butona basılmama beklemesi, reset tuşuna
basılı iken çalışmaz.

Program resetlendikten sonra üçüncü butonun bırakılması, "Reset Başarılı"
yazısının ekrandan silinmesini sağlar.