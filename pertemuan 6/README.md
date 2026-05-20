# Praktikum Sistem Mikrokontroler - Modul 6: Interrupt dan Timer

## Pertanyaan Praktikum Percobaan 1
1. Jelaskan proses bagaimana tombol dapat mengubah kondisi LED menggunakan interrupt!

    Jawaban: Pada program percobaan 1, push button dihubungkan ke pin 2 Arduino yang berfungsi sebagai external interrupt. Pin tersebut dikonfigurasi menggunakan INPUT_PULLUP, sehingga kondisi normal pin adalah HIGH. Ketika tombol ditekan, kondisi pin berubah menjadi LOW. Perubahan sinyal tersebut akan memicu interrupt dengan mode falling. saat interrupt terjadi, program utama dihentikan sementara, kemudian arduino menjalankan fungsi ISR bernama tombolInterrupt(). Di dalam fungsi ISR, nilai variabel ledState diubah menggunakan operator not(!). Setelah ISR selesai dijalankan, program utama kembali berjalan. Pada fungsi loop(), arduino menuliskan kondisi ledState ke LED menggunakan digitalWrite(13, ledState). Akibatnya, setelah tombol ditekan, kondisi LED akan berubah dari OFF menjadi ON atau dari ON menjadi OFF.

2. Apa fungsi attachInterrupt() pada program tersebut?

    Jawaban: Fungsi attachInterrupt() digunakan untuk menghubungkan pin interrupt dengan fungsi ISR yang akan dijalankan ketika kondisi interrupt tertentu terjadi.

3. Mengapa pada ISR tidak disarankan menggunakan delay() dan Serial.print()?

    Jawaban: ISR harus dijalankan secepat mungkin karena selama ISR berjalan, program utama tidak akan berjalan. Penggunaan delay() atau Serial.print() dalam ISR akan memperlama waktu eksekusi ISR, sehingga dapat menyebabkan hilangnya interrupt lain atau keterlambatan dalam merespon interrupt. Selain itu, delay() dan Serial.print() menggunakan sumber daya yang cukup besar, sehingga tidak disarankan untuk digunakan dalam ISR yang harus berjalan cepat.

4. Apa fungsi keyword volatile pada variabel ledState?

    Jawaban: keyword volatile digunakan untuk memberitahu conpiler bahwa nilai variabel dapat berubah sewaktu-waktu di luar alur program utama (misalnya dari ISR).

5. Pada percobaan digunakan mode interrupt FALLING. Modifikasikan program menggunakan mode interrupt lain (RISING, CHANGE, atau LOW) kemudian:
- Jelaskan perbedaan cara kerja masing-masing mode interrupt tersebut
- Analisis perubahan perilaku LED yang terjadi pada setiap mode
- Sertakan source code dan penjelasan program dalam bentuk README.md

    Jawaban:
    - RISING : Interrupt aktif ketika sinyal berubah dari LOW ke HIGH
    - FALLING : Interrupt aktif ketika sinyal berubah dari HIGH ke LOW
    - CHANGE : Interrupt aktif ketika sinyal berubah dari LOW ke HIGH atau dari HIGH ke LOW
    - LOW : Interrupt aktif ketika sinyal LOW

    Perilaku LED:
    - Mode RISING : LED berubah kondisi ketika tombol dilepas karena sinyal berubah dari LOW ke HIGH.
    - Mode FALLING : LED berubah kondisi ketika tombol ditekan karena sinyal berubah dari HIGH ke LOW.
    - Mode CHANGE : LED dapat berubah dua kali, yaitu saat tombol ditekan dan saat dilepas karena setiap perubahan sinyal memicu interrupt.
    - Mode LOW : Interrupt terus aktif selama tombol ditekan karena kondisi pin tetap LOW. Hal ini dapat menyebabkan LED berubah sangat cepat dan terlihat berkedip.

    source code mode RISING:
    ```cpp
    #include <Arduino.h>

    volatile bool ledState = false;

    void tombolInterrupt() {
        ledState = !ledState;
    }

    void setup() {
        pinMode(13, OUTPUT);
        pinMode(2, INPUT_PULLUP);
  
        attachInterrupt(digitalPinToInterrupt(2), 
        tombolInterrupt, 
        RISING // ubah mode interrupt pada bagian ini
        ); 
    }

    void loop() {
        digitalWrite(13, ledState);
    }
    ```

    **Penjelasan:**
    
    Kode ini mengontrol **LED** menggunakan **interrupt** dari tombol pada pin 2.

    **Bagian Utama**
    - **`volatile bool ledState`** — menyimpan status LED. `volatile` agar nilainya selalu dibaca ulang dari memori saat interrupt terjadi.
    - **`tombolInterrupt()`** — dipanggil otomatis saat interrupt; membalik nilai `ledState` (toggle).
    - **`setup()`** — mengatur pin 13 sebagai output LED, pin 2 sebagai input tombol, lalu mendaftarkan interrupt mode `RISING` (aktif saat sinyal naik LOW → HIGH).
    - **`loop()`** — terus menulis `ledState` ke pin 13 sehingga LED menyala/mati sesuai statusnya.

    **Alur Kerja**

    ```
    Tombol ditekan → sinyal RISING → interrupt terpicu → ledState toggle → LED berubah
    ```

> Mode `RISING` bisa diganti `FALLING`, `CHANGE`, atau `LOW` sesuai kebutuhan.
    



## Pertenyaan Praktikum Percobaan 2
1. Jelaskan bagaimana fungsi millis() bekerja pada program tersebut!

    Jawaban: Fungsi millis() digunakan untuk membaca waktu sejak Arduino pertama kali menjala dalam satuan milidetik. Pada program: `unsigned long currentMillis = millis();` Nilai waktu saat ini dibandingkan dengan waktu sebelumnya: `if (currentMillis - previousMillis >= interval)` jika selisih waktu sudah mencapai interval tertentu maka nilai `previousMillis` diperbarui, status LSD dibalik, LED dinyalakan atau dimatikan. Dengan cara ini LED dapat berkedip tanpa menghentikan program utama.

2. Apa perbedaan utama antara delay() dan millis()?

    Jawaban: delay() akan menghentikan total seluruh proses program selama waktu tunggu (blocking), sedangkan millis() bekerja sebagai pengukur waktu(timer) di latar belakan tanpa mengganggu atau menghentika fungsi lain (non-blocking).

3. Mengapa metode millis() disebut non-blocking?

    Jawaban: metode millis() disebut non-blocking karena program tidak berhenti saat menunggu waktu tertentu. Arduino tetap menjalankan instruksi lain sambil terus mengecek waktu menggunakan millis()

4. Modifikasi program agar:
- LED pertama berkedip setiap 1 detik
- LED kedua berkedip setiap 500 ms
- Tanpa menggunakan delay()
Berikan penjelasan setiap baris program dalam bentuk README.md

    Jawaban:
    ```cpp

    #include <Arduino.h>

    const int led1 = 13;
    const int led2 = 12;

    unsigned long previousMillis1 = 0;
    unsigned long previousMillis2 = 0;

    const long interval1 = 1000;
    const long interval2 = 500;

    bool ledState1 = false;
    bool ledState2 = false;

    void setup() {
        pinMode(led1, OUTPUT);
        pinMode(led2, OUTPUT);
    }

    void loop() {
        unsigned long currentMillis = millis();

        if(currentMillis - previousMillis1 >= interval1) {
            previousMillis1 = currentMillis;
            ledState1 = !ledState1;
            digitalWrite(led1, ledState1);
        }

        if(currentMillis - previousMillis2 >= interval2) {
            previousMillis2 = currentMillis;
            ledState2 = !ledState2;
            digitalWrite(led2, ledState2);
        }
    }

    ```

    ## Penjelasan:

    ### Bagian 1 — Library & Deklarasi Pin

    ```cpp
    #include <Arduino.h>
    ```
    > Mengimpor library inti Arduino. Wajib disertakan saat menggunakan framework Arduino di luar Arduino IDE (misalnya PlatformIO), agar fungsi seperti `pinMode()`, `digitalWrite()`, dan `millis()` dapat digunakan.
 
    ---
 
    ```cpp
    const int led1 = 13;
    ```
    > Mendeklarasikan konstanta integer bernama `led1` dengan nilai `13`. Ini berarti LED pertama terhubung ke **pin digital 13** pada board Arduino. Kata kunci `const` mencegah nilai ini diubah di bagian lain program.
 
    ---
 
    ```cpp
    const int led2 = 12;
    ```
    > Mendeklarasikan konstanta `led2` dengan nilai `12`, yaitu **pin digital 12** untuk LED kedua.
 
    ---

    ### Bagian 2 — Variabel Waktu Sebelumnya
 
    ```cpp
    unsigned long previousMillis1 = 0;
    ```
    > Mendeklarasikan variabel `previousMillis1` bertipe `unsigned long` (bilangan bulat positif besar, hingga ~4,2 miliar). Variabel ini menyimpan **waktu terakhir LED 1 berubah status**. Diinisialisasi dengan nilai `0` saat program pertama berjalan.
 
    ---
 
    ```cpp
    unsigned long previousMillis2 = 0;
    ```
    > Sama seperti di atas, tetapi untuk **LED 2**. Menyimpan waktu terakhir LED 2 berubah status.
 
    ---
 
    ### Bagian 3 — Interval Kedip
 
    ```cpp
    const long interval1 = 1000;
    ```
    > Mendeklarasikan konstanta `interval1` bernilai `1000` (dalam milidetik). Artinya LED 1 akan **berkedip setiap 1 detik** (1000 ms = 1 detik).
 
    ---
 
    ```cpp
    const long interval2 = 500;
    ```
    > Mendeklarasikan konstanta `interval2` bernilai `500` milidetik. Artinya LED 2 akan **berkedip setiap 0,5 detik** (2 kali lebih cepat dari LED 1).
 
    ---
 
    ### Bagian 4 — Status LED
 
    ```cpp
    bool ledState1 = false;
    ```
    > Mendeklarasikan variabel boolean `ledState1` dengan nilai awal `false` (mati/LOW). Variabel ini melacak **kondisi saat ini dari LED 1** — apakah sedang menyala atau mati.
 
    ---
 
    ```cpp
    bool ledState2 = false;
    ```
    > Sama seperti di atas untuk **LED 2**. Nilai awal `false` berarti LED 2 juga mulai dalam kondisi mati.
 
    ---
 
    ### Bagian 5 — Fungsi `setup()`
 
    ```cpp
    void setup() {
    ```
    > Mendefinisikan fungsi `setup()`. Fungsi ini **hanya dijalankan sekali** saat Arduino dinyalakan atau di-reset. Digunakan untuk konfigurasi awal perangkat keras.
 
    ---
 
    ```cpp
    pinMode(led1, OUTPUT);
    ```
    > Mengatur **pin 13** (led1) sebagai **OUTPUT**, sehingga pin tersebut dapat mengeluarkan tegangan untuk menyalakan LED.
 
    ---
 
    ```cpp
    pinMode(led2, OUTPUT);
    ```
    > Mengatur **pin 12** (led2) sebagai **OUTPUT** dengan fungsi yang sama untuk LED 2.
 
    ---
 
    ```cpp
    }
    ```
    > Menutup blok fungsi `setup()`. 
 
    ---
 
    ### Bagian 6 — Fungsi `loop()`
 
    ```cpp
    void loop() {
    ```
    > Mendefinisikan fungsi `loop()`. Fungsi ini **berjalan terus-menerus** (berulang tanpa henti) selama Arduino menyala. Semua logika utama program diletakkan di sini.
 
    ---
 
    ```cpp
    unsigned long currentMillis = millis();
    ```
    > Membaca nilai dari fungsi `millis()`, yaitu **jumlah milidetik sejak Arduino dinyalakan**, lalu menyimpannya ke variabel `currentMillis`. Ini adalah "jam" program yang digunakan sebagai acuan waktu.
 
    ---
 
    ### Sub-bagian: Kontrol LED 1
 
    ```cpp
    if(currentMillis - previousMillis1 >= interval1) {
    ```
    > Memeriksa apakah **selisih waktu** antara waktu sekarang (`currentMillis`) dan waktu terakhir LED 1 berubah (`previousMillis1`) sudah mencapai atau melebihi `interval1` (1000 ms). Jika ya, sudah waktunya LED 1 berubah status. 
 
    ---
 
    ```cpp
    previousMillis1 = currentMillis;
    ```
    > **Memperbarui** `previousMillis1` dengan waktu sekarang sebagai titik referensi baru, agar penghitungan interval berikutnya dimulai dari sini.
 
    ---
 
    ```cpp
    ledState1 = !ledState1;
    ```
    > **Membalik** nilai `ledState1`. Jika sebelumnya `true` (menyala) menjadi `false` (mati), dan sebaliknya. Tanda `!` adalah operator NOT (negasi logika). 
 
    ---
 
    ```cpp
    digitalWrite(led1, ledState1);
    ```
    > Menulis nilai `ledState1` ke pin LED 1. Jika `ledState1` bernilai `true`, pin akan bernilai HIGH (menyalakan LED). Jika `false`, pin bernilai LOW (mematikan LED). 
 
    ---
 
    ```cpp
    }
    ```
    > Menutup blok `if` untuk LED 1. 
 
    ---
 
    ### Sub-bagian: Kontrol LED 2
 
    ```cpp
    if(currentMillis - previousMillis2 >= interval2) {
    ```
    > Memeriksa apakah sudah waktunya **LED 2 berubah status**, dengan cara yang sama seperti LED 1 tetapi menggunakan `previousMillis2` dan `interval2` (500 ms). 
 
    ---
 
    ```cpp
    previousMillis2 = currentMillis;
    ```
    > Memperbarui `previousMillis2` dengan waktu sekarang sebagai referensi baru untuk LED 2.
 
    ---
 
    ```cpp
    ledState2 = !ledState2;
    ```
    > Membalik nilai `ledState2` untuk mengubah status LED 2. 
 
    ---
 
    ```cpp
    digitalWrite(led2, ledState2);
    ```
    > Menulis nilai `ledState2` ke pin LED 2, menyalakan atau mematikannya sesuai kondisi terkini. 
 
    ---
 
    ```cpp
    }
    }
    ```
    > Menutup blok `if` untuk LED 2, lalu menutup fungsi `loop()`.
 
    ---
 
    ## Ringkasan Cara Kerja
 
    | Komponen | Pin | Interval Kedip |
    |---|---|---|
    | LED 1 | Pin 13 | Setiap **1000 ms** (1 detik) |
    | LED 2 | Pin 12 | Setiap **500 ms** (0,5 detik) |
    