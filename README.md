# 🚀 STM32F407 Gömülü Sistemler Mühendisliği Günlüğü

Bu depo; **STM32F407VG Discovery** kartı üzerinde **STM32CubeIDE** geliştirme ortamı ve **STM32Cube HAL** kütüphanesini kullanarak geliştirdiğim gömülü sistem projelerini belgeleyen kişisel mühendislik günlüğüdür.

Projeler modüler olarak geliştirilmekte ve tamamlandıkça depoya eklenmektedir.

---

## 📺 Donanım Uygulamaları & Video Kütüphanesi

Tüm projelerin fiziksel donanım üzerindeki çalışma videoları, osiloskop/ölçüm sonuçları ve devre görselleri Google Drive klasöründe saklanmaktadır:

🔗 **[STM32 Proje Videoları Klasörüne Erişmek İçin Buraya Tıklayın](https://drive.google.com/drive/folders/1QEF27VpM2UhSrB0GhmENcJqSesYxxM8_?usp=sharing)**

---

## 🛠️ Tamamlanan ve Geliştirilen Uygulamalar

### 🔹 GPIO ve Temel Çevre Birimleri
* **GPIO İşlemleri:** Digital Output (LED, Buzzer) ve Digital Input (Button) uygulamaları
* **Button State Control & Counter:** Aktif-Düşük (Active-Low) ve Aktif-Yüksek buton okuma, buton ile sayıcı (Counter) mantığı
* **Hata Ayıklama (Debug):** STM32CubeIDE üzerinden **SWV / ITM Live Data Trace** ile canlı değişken ve durum takibi

### 🔹 Analog Arayüzler (ADC / DAC)
* **ADC Uygulamaları:** Potansiyometre ve NTC Termistör ile analog veri okuma, sıcaklık/gerilim hesaplamaları

### 🔹 Zamanlayıcılar (Timers) & Kesmeler (Interrupts)
* **External Interrupt (EXTI):** Harici kesme mekanizmaları ile buton kontrolü
* **Hardware Timers:** Timer kesmeleri (TIM Interrupts) ile zamanlama kontrolü

### 🔹 Motor Sürücüler ve Aktüatörler
* Transistör ile DC Motor kontrolü
* **L293D Entegresi** ile DC Motor yön ve hız kontrolü
* **ULN2003AN Entegresi** ile Stepper (Adım) Motor sürülmesi
* **Seven Segment Display:** Çoklamalı (Multiplexed) ekran sürücü uygulamaları

### 🔹 Seri İletişim Protokolleri *(Aktif Çalışma)*
* **UART / USART:** Asenkron seri haberleşme ve veri iletimi

---

👩‍💻 **Geliştirici:** Feyza Yağmur Arat  
🎓 **Bölüm:** Mersin Üniversitesi - Elektrik-Elektronik Mühendisliği  
🛠️ **Geliştirme Ortamı:** STM32CubeIDE | STM32Cube HAL  
🎯 **Donanım:** STM32F407VG Discovery Board
