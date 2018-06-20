#include <stdint.h>
#include "inc/tm4c123gh6pm.h"

#define RW 0x20 								// PA5
#define RS 0x40  								// PA6
#define E  0x80  								// PA7

volatile unsigned long delay;					// Port aktive ederken işimize yarayacak delay
int secim = 0;									// Kullanıcının hangi butona bastığını gösterecek seçim
int kacMS = 0;									// Kullanıcının bastığı butona kaç MS basılı tuttuğunu gösteren değişken
int needsReset = 0;								// Çözümle fonksiyonu çağırıldığında 1 olan değişken
int para[5] = {0,0,0,0,0};						// nokta, onlar, birler, ondabirler, yüzdebirler

int msArttir();									// Çağırdığı fonksiyon kendisini çağırdığı için protoipini yaptık

// LCD komutları:
// http://michaelhuang69.blogspot.com.tr/2014/05/tm4c123gxl-launchpad-lcd1602.html

void portlariAktiflestir(void){
	SYSCTL_RCGC2_R |= SYSCTL_RCGC2_GPIOA;		// A portunu aktive et
	delay = SYSCTL_RCGC2_R;						// A portunun aktive edilmesini 1 tick bekle
	GPIO_PORTA_AMSEL_R &= ~0b11100000;			// A portunun analog modunu devre dışı bırak
	GPIO_PORTA_PCTL_R &= ~0xFFF00000;			// A portundaki pinlerin voltajını düzenle (PCTL=Power Control)
	GPIO_PORTA_DIR_R |= 0b11100000;				// A portunun giriş çıkışlarını belirle
	GPIO_PORTA_AFSEL_R &= ~0b11100000;			// A portundaki alternatif fonksiyonları seç
	GPIO_PORTA_DEN_R |= 0b11100000;				// A portunun pinlerini aktifleştir
	GPIO_PORTA_DR8R_R |= 0b11100000;			// A portundaki pinlerin 8mA çıkışını aktive et

	SYSCTL_RCGC2_R |= SYSCTL_RCGC2_GPIOB;		// B portunu aktive et
	delay = SYSCTL_RCGC2_R;						// B portunun aktive edilmesini 1 tick bekle
	GPIO_PORTB_AMSEL_R &= ~0b11111111;			// B portunun analog modunu devre dışı bırak
	GPIO_PORTB_PCTL_R &= ~0xFFFFFFFF;			// B portundaki pinlerin voltajını düzenle (PCTL=Power Control)
	GPIO_PORTB_DIR_R |= 0b11111111;				// B portunun giriş çıkışlarını belirle
	GPIO_PORTB_AFSEL_R &= ~0b11111111;			// B portundaki alternatif fonksiyonları seç
	GPIO_PORTB_DEN_R |= 0b11111111;				// B portunun pinlerini aktifleştir
	GPIO_PORTB_DR8R_R |= 0b11111111;			// B portundaki pinlerin 8mA çıkışını aktive et

	SYSCTL_RCGC2_R |= SYSCTL_RCGC2_GPIOE;		// E portunu aktive et
	delay = SYSCTL_RCGC2_R;						// E portunun aktive edilmesini 1 tick bekle
	GPIO_PORTE_DIR_R |= 0x00;					// E portunun giriş çıkışlarını belirle
	GPIO_PORTE_DEN_R |= 0b00011111;				// E portunun pinlerini aktifleştir
}


void komutGonder(unsigned char LCD_Comment){
	GPIO_PORTA_DATA_R &= ~(RS+RW+E);			// Tüm pinleri sıfırla
	GPIO_PORTB_DATA_R = LCD_Comment;			// Komutu yazdır
	GPIO_PORTA_DATA_R |= E;						// E'yi aç
	GPIO_PORTA_DATA_R &= ~(RS+RW);				// RS ve RW kapat
	for (delay = 0 ; delay < 1; delay++);		// 1us bekle
	GPIO_PORTA_DATA_R &= ~(RS+RW+E);			// RS, RW ve E kapat
	for (delay = 0 ; delay < 1000; delay++);	// 1ms bekle
}

void veriGonder(unsigned char LCD_Data){
	GPIO_PORTB_DATA_R = LCD_Data;				// Write Data
	GPIO_PORTA_DATA_R |= RS+E;					// RS ve E aç
	GPIO_PORTA_DATA_R &= ~RW;					// RW kapat
	for (delay = 0 ; delay < 23 ; delay++);		// 230ns bekle
	GPIO_PORTA_DATA_R &= ~(RS+RW+E);			// RS, RW ve E kapat
	for (delay = 0 ; delay < 1000; delay++);	// 1ms bekle
}

void ekraniAktiflestir(){
	portlariAktiflestir();						// Portları aktifleştir
	for (delay = 0 ; delay < 15000; delay++);	// 15ms bekle
	komutGonder(0x38);							// 0b00111000 -> PortB
	for (delay = 0 ; delay < 5000; delay++);	// 5ms bekle
	komutGonder(0x38);							// 0b00111000 -> PortB
	for (delay = 0 ; delay < 150; delay++);		// 150us bekle
	komutGonder(0x0C);							// 0b00001010 -> PortB
	komutGonder(0x01);							// Ekranı Temizle
	komutGonder(0x06);							// 0b00000110 -> PortB
	for (delay = 0 ; delay < 50000; delay++);	// 50ms bekle
}

void ekranaYazdir(unsigned int line,unsigned int digit, unsigned char *str){
	unsigned int lineCode = line==1 ?0x80:0xC0;	// Line 1 ise 0x80, 2 ise 0xC0 komutu kullanılması gerekiyor
	komutGonder(lineCode + digit);				// Yazının nereden başlayacağını LCD'ye bildir
	while(*str != 0){ veriGonder(*str++); }		// Ve yazdır
}

void paraYazdir(int *scopePara){

    // Bu alandaki integer'ları 48 ile toplayarak Char'a dönüştürdük.
    // Bknz: 0 ASCII 48, 1 ASCII 49 => int+48 == Sayının Char'ı

    char fullStr[6] = {'-','-','-','-','-', '\0'};
    if(scopePara[0]){        					// Nokta var ise
        if(scopePara[1]){    					// Binler basamağı var ise
            fullStr[0] = scopePara[1]+48;    	//
        }
        if(scopePara[1] || scopePara[2]){		// Birler basamağı veya onlar basamağı var ise
            fullStr[1] = scopePara[2]+48;		// Birler basamağı gösterilmeli
        }
        fullStr[2] = '.';						// Noktayı koy
        fullStr[3] = scopePara[3]+48;			// Noktadan sonraki kısım ne olursa olsun
        fullStr[4] = scopePara[4]+48;			// gösterilecek. (00 dahi olsa)
    }else{
        if(scopePara[1]){    					// Onlar basamağı var ise
            fullStr[3] = scopePara[1]+48;		// Onlar basamagi gösterilmeli
        }
        if(scopePara[1] || scopePara[2]){		// Birler basamağı veya onlar basamağı var ise
            fullStr[4] = scopePara[2]+48;		// Birler basamağı gösterilmeli
        }
    }
    ekranaYazdir(1,0,"-----------");			// Sayıdan önceki tüm karakterler tire olacak
    ekranaYazdir(1,11,fullStr);					// ve sayı, en sağa yaslı şekilde gösterilecek

}

void cozumle(){
	int cozumleme[9] = {2000,1000,500,100,50,25,10,5,1};
	char* cozumlemeStr[9] = {"yirmilik", "onluk", "beslik", "birlik", "yarimlik", "ceyreklik", "metelik", "delik", "kurusluk"};
	int i;
	int kaynak = para[1]*1000+para[2]*100+para[3]*10+para[4];

	// Önemli not: Kaynak maksimum yüzdebirler basamağına gittiği için buradaki işlemler
	// floatig point kullanılmadan, girilen sayının 100 katı alınarak hesaplanır.
	// Örn: 12.34 = 1234

	needsReset=1;								// Çözümleme başladıktan sonra yeni veri girişi kabul edilmez.

    for(i=0;i<9;i++){							// Tüm çözümlemeleri dolaş
    	int comeout = 0;						// Çözümleme, reset edilmek için bölünürse; bu değişkene basılan buton atanır.
    	if(kaynak/cozumleme[i]){				// Kaynak bölünebiliyorsa
    		int kaynakCycle = 0;				// Kaynak gösterilirken 3 saniye bekleyecek
            int yeniKaynak = kaynak - (kaynak/cozumleme[i])*cozumleme[i];
            int yeniKaynakArr[5] = {
            		1,							// Çözümlerken nokta koy
					((yeniKaynak/1000)),		// Onlar basamağı
					((yeniKaynak/100)%10),		// Birler basamağı
					((yeniKaynak/10)%10),		// Ondabirler basamağı
					((yeniKaynak)%10)			// Yüzdebirler basamağı
            };
            char fullStr[2] = {(kaynak/cozumleme[i])+48, '\0'};
            paraYazdir(yeniKaynakArr);			// Kaynaktan düşürülmüş veriyi yazdır
            ekranaYazdir(2,0,fullStr);			// Kaç adet para verilecek ise yazdır
            ekranaYazdir(2,1," - ");			// Daha sonra tire çek
            ekranaYazdir(2,4,"------------");	// Eski kalan verinin üzerine tire çek
            ekranaYazdir(2,4,cozumlemeStr[i]);	// Yeni veriyi yazdır

            //3sn kadar bekle, bu sırada da reset butonuna basılıyor mu diye kontrol et
            for (kaynakCycle = 0 ; kaynakCycle < 3000; kaynakCycle++){
            	comeout = msArttir();			// 3. butona 5 saniyeden fazla basıldı ise 3 döndürür
            	if(comeout==3){ break; }		// Eğer 3. butona en az 5sn basıldı ise 3 saniye beklemeyi bırak
            	for (delay = 0 ; delay < 1000 ; delay++);
            }

            if(comeout==3){ break; }			// Eğer 3. butona en az 5sn basıldı ise çözümlemeye yapmayı bırak

            kaynak = yeniKaynak;				// Kaynağı güncelleyip çözümlemeye devam et
        }
    }
	if(kaynak==0){								// Kaynak tamamen çözümlenmişse işlem başarıyla tamamlanmıştır
		ekranaYazdir(1,0," Islem Basarili ");
		ekranaYazdir(2,0,"Reset Bekleniyor");
	}
}

void resetle(){
	para[0]=1;									// 3. buton bırakıldığında nokta koymaması için toggle edilebilen bir nokta koy
	para[1]=0;									// Onlar basamağını sıfırla
	para[2]=0;									// Birler basamağını sıfırla
	para[3]=0;									// Ondabirler basamağını sıfırla
	para[4]=0;									// Yüzdebirler basamağını sıfırla
	kacMS = 0;									// Butona basılı tutma zamanını sıfırla
	needsReset = 0;								// Çözümleme yapılabilir hale getir
	secim=0;									// Basılan butonu sıfırla
	ekranaYazdir(1,0,"----------------");		// En baştaki halini aldır ve
	ekranaYazdir(2,0," Reset Basarili ");		// Ekrana resetin başarılı olduğunu aktar
}

int basiliButon(){								// Şu anda basılı olan butonu döndüren fonksiyon
	return    (GPIO_PORTE_DATA_R & 0b00000001) == 0 ? 1
			: (GPIO_PORTE_DATA_R & 0b00000010) == 0 ? 2
			: (GPIO_PORTE_DATA_R & 0b00000100) == 0 ? 3
			: (GPIO_PORTE_DATA_R & 0b00001000) == 0 ? 4
			: (GPIO_PORTE_DATA_R & 0b00010000) == 0 ? 5
			: 0 ;
}

int msArttir(){
	if(basiliButon()==3){						// 3. butona basılıyorsa
		kacMS++;								// Butona basma zamanını arttır
		if(kacMS>5000){							// Eğer 5 saniyeden uzun basıyorsa
			resetle();							// Resetle
			return 3;							// ve Resetlendiğini bildir
		}
	}else if(basiliButon()==0 && !needsReset &&	// Eğer hiçbir butona basılmıyorsa ve çözümleme yapılmamışsa ve
		!(para[1]==0 && para[2]==0 &&			// herhangi veri girişi yapılmışsa
				para[3]==0 && para[4]==0)){
		kacMS++;								// Butona basma zamanını arttır
		if(kacMS>5000){							// Eğer 5 saniyeden uzun basıyorsa
			kacMS=0;							// Sayacı sıfırla ve
			cozumle();							// Çözümlemeye başla
		}
	}
	return 0;									// İstenen butona basılmıyorsa 0 döndür
}

int main(){
	ekraniAktiflestir();										// Ekranı aktifleştir
	paraYazdir(para);											// Girişmemiş parayı (-----) yazdır
	while(1){
		if(basiliButon()){										// Eğer herhangi bir butona basılmışsa
			secim = basiliButon();								// Hangi butona basıldığını bir değişkene ata
		}else{													// Buton bırakıldığında (veya hiç basılmadığında)
			if(secim!=0){										// Herhangi bir buton bırakıldıysa
					 if(secim==1){para[1] = (para[1]+1)%10;}	// 1. Butona tıklanmış ise onlar basamağını bir arttır
				else if(secim==2){para[2] = (para[2]+1)%10;}	// 2. Butona tıklanmış ise birler basamağını bir arttır
				else if(secim==3){								// 3. Butona tıklanmış ise
					para[3] = 0; para[4] = 0;					// Noktadan sonraki basamakları sıfırla
					para[0] = (para[0]+1)%2;					// Nokta koy / kaldır
					if(!needsReset){
						ekranaYazdir(2,0,"                ");	// Reset Basarili yazısı var ise kaldır
					}
				}
				else if(secim==4 && para[0]==1){
					para[3] = (para[3]+1)%10;					// 4. Butona tıklanmış ise ondabirler basamağını bir arttır
				}
				else if(secim==5 && para[0]==1){
					para[4] = (para[4]+1)%10;					// 5. Butona tıklanmış ise yüzdebirler basamağını bir arttır
				}
				if(!needsReset){ paraYazdir(para); }			// Eğer çözümleme yapılmadıysa parayı yazdır
				kacMS=0;										// Butona basılı tutma zamanını sıfırla
			}
			secim=0;											// Seçimi sıfırla
		}
		msArttir();												// Buton bırakma sayacını bir arttır ve butonları kontrol et
	    for (delay = 0 ; delay < 1000 ; delay++);
	}
}
