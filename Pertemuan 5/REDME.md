## Modul 5: (RTOS) Real-Time Operating System
## Pertanyaan Percobaan 1
1. Apakah ketiga task berjalan secara bersamaan atau bergantian? Jelaskan mekanismenya!

    Jawab: Ketiga task tidak berjalan secara bersamaan melainkan berjalan secara bergantian. Mekanisme ini diatur oleh sceduler dari FreeRTOS menggunakan metode preemptive scheduling. Setiap task mendapatkan jatah waktu eksekusi dari CPU secara cepat sehingga terlihat seperti berjalan secara bersamaan. Ketika sebuah task menjalankan fungsi vTaskDelay(), task tersebut masuk ke kondisi blocked sehingga akan memberikan CPU kepada task lain yang siap  dijalankan.

2. Bagaimana cara menambahkan task keempat? Jelaskan langkahnya!

    Jawab:
    1. Deklarasikan kerangka fungsi di bagian paling atas program
    ```cpp
    void TaskKeempat(void *pvParameters);
    ```
    2. Tambahkan perintah untuk membuat task di dalam fungsi setup() menggunakan perintah xTaskCreate() dan ketik sebelum vTaskStartScheduler()
    ```cpp
    xTaskCreate(TaskKeempat, "task4", 128, NULL, 1, NULL);
    ```
    3. Tulis isi program di bagian paling bawah
    ```cpp
    void TaskKeempat(void *pvParameters) {
    while(1) {
        vTaskDelay(1000 / portTICK_PERIOD_MS);
        }
    }
    ``` 

3. Modifikasilah program dengan menambah sensor (misalnya potensiometer), lalu gunakan nilainya untuk mengontrol kecepatan LED! Bagaimana hasilnya? Jelaskan program pada file README.md.

    Jawab:
    ```cpp
    #include <Arduino_FreeRTOS.h>

    // Deklarasi pin sensor potensiometer
    const int potPin = A0;

    // Deklarasi fungsi task
    void TaskBlink1(void *pvParameters);
    void TaskBlink2(void *pvParameters);
    void Taskprint(void *pvParameters);

    void setup() {
        // Inisialisasi komunikasi serial pada 9600 bps:
        Serial.begin(9600);

        // Membuat task FreeRTOS
        xTaskCreate(TaskBlink1, "task1", 128, NULL, 1, NULL);
        xTaskCreate(TaskBlink2, "task2", 128, NULL, 1, NULL);
        xTaskCreate(Taskprint, "task3", 128, NULL, 1, NULL);

        // Memulai scheduler
        vTaskStartScheduler();
    }

    void loop() {
    }

    void TaskBlink1(void *pvParameters) {
        pinMode(8, OUTPUT);

        while (1) {
            // Membaca nilai dari potensiometer (0 - 1023)
            int potValue = analogRead(potPin);
    
            // Mapping nilai potensiometer menjadi delay 50ms - 1000ms
            int delayTime = map(potValue, 0, 1023, 50, 1000);

            Serial.print("Task1 nyala, Delay: ");
            Serial.println(delayTime);

            digitalWrite(8, HIGH);
            vTaskDelay(delayTime / portTICK_PERIOD_MS);
            digitalWrite(8, LOW);
            vTaskDelay(delayTime / portTICK_PERIOD_MS);
        }
    }

    void TaskBlink2(void *pvParameters) {
        pinMode(7, OUTPUT);

        while (1) {
            // Membaca nilai potensiometer yang sama
            int potValue = analogRead(potPin);
    
            // Mapping dengan rentang sedikit berbeda agar ritmenya unik (100ms - 1500ms)
            int delayTime = map(potValue, 0, 1023, 100, 1500);

            digitalWrite(7, HIGH);
            vTaskDelay(delayTime / portTICK_PERIOD_MS);
            digitalWrite(7, LOW);
            vTaskDelay(delayTime / portTICK_PERIOD_MS);
        }
    }

    void Taskprint(void *pvParameters) {
        // Modifikasi Taskprint untuk menampilkan nilai mentah sensor ke Serial Monitor
        while (1) {
            int potValue = analogRead(potPin);
    
            Serial.print("Nilai Mentah Potensiometer (Task3): ");
            Serial.println(potValue);
    
            // Delay 500ms agar Serial Monitor tidak terlalu banjir data (flooding)
            vTaskDelay(500 / portTICK_PERIOD_MS);
        }
    }
    ```
    Penjelasan:
    Pada program tersebut, ditambahkan sebuah potensiometer yang terhubung ke pin analog A0. Nilai dari potensiometer tersebut digunakan untuk mengontrol kecepatan kedipan LED pada task1 dan task2. Semakin kecil nilai potensiometer, semakin cepat kedipan LED. Hasilnya LED berkedip sesuai dengan nilai pembacaan dari potensiometer.


## Pertanyaan Percobaan 2
1. Apakah kedua task berjalan secara bersamaan atau bergantian? Jelaskan mekanismenya!

    Jawab: Kedua task pada program tersebut berjalan secara bergantian (konkuren), bukan bersamaan secara harfiah. Hal ini dikarenakan mikrokontroler standar Arduino umumnya memiliki prosesor inti tunggal yang hanya mampu mengeksekusi satu instruksi pada satu waktu. FreeRTOS menciptakan ilusi berjalan bersamaan melalui teknik *multitasking*, di mana sistem secara cerdas memindahkan kendali prosesor antar task dengan sangat cepat. 

    Mekanisme pergantian ini dikendalikan dengan sangat rapi melalui penggunaan antrean (*Queue*) dan penundaan waktu (*Delay*). Pada awalnya, task `read_data` akan memproduksi dan mengirimkan data suhu serta kelembapan ke dalam antrean, lalu sengaja masuk ke mode tidur sementara waktu. Kekosongan kendali ini dimanfaatkan oleh sistem untuk menjalankan task `display`. Task `display` kemudian mengambil data dari antrean tersebut, menampilkannya ke Serial Monitor, dan otomatis ikut masuk ke mode tidur karena antrean sudah kosong dan ia harus menunggu data baru. Ketika waktu penundaan task `read_data` sudah habis, task tersebut akan terbangun dan kembali mengirimkan data baru, yang secara otomatis ikut membangunkan task `display`. Siklus pergantian operan yang saling merespons inilah yang membuat program berjalan secara harmonis tanpa saling bertabrakan. 

2. Apakah program ini berpotensi mengalami race condition? Jelaskan!

    Jawab: Program tersebut tidak berpotensi mengalami *race condition* karena pertukaran datanya dirancang dengan sangat aman. Alih-alih membiarkan kedua task berebut mengakses variabel global, program ini mengandalkan fitur antrean (*Queue*) bawaan FreeRTOS yang sudah terjamin keamanannya (*thread-safe*). Selain itu, variabel penyimpan data dideklarasikan secara lokal di masing-masing task, dan FreeRTOS memindahkan data tersebut dengan cara menyalin nilainya (*copy by value*) ke dalam antrean. Dengan mekanisme ini, sistem secara otomatis mengunci dan melindungi data saat proses kirim-terima berlangsung, sehingga tidak ada risiko kedua task memanipulasi memori yang sama pada waktu yang bersamaan.

3. Modifikasilah program dengan menggunakan sensor DHT sesungguhnya sehingga informasi yang ditampilkan dinamis. Bagaimana hasilnya? Jelaskan program pada file README.md.

    Jawab:
    ```cpp
    #include <Arduino_FreeRTOS.h>
    #include <queue.h>
    #include <DHT.h>

    #define DHTPIN 2       
    #define DHTTYPE DHT22  
    DHT dht(DHTPIN, DHTTYPE);

    struct readings {
        float temp;
        float h;
    };

    QueueHandle_t my_queue;

    void read_data(void *pvParameters);
    void display(void *pvParameters);

    void setup() {
        Serial.begin(9600);
        Serial.println("Memulai Sistem..."); 

        dht.begin();

        my_queue = xQueueCreate(1, sizeof(struct readings));

        if (my_queue != NULL) {
            xTaskCreate(read_data, "read sensors", 128, NULL, 1, NULL);
            xTaskCreate(display, "display", 128, NULL, 1, NULL);

            Serial.println("Memulai Task Scheduler...");
            vTaskStartScheduler();
    
            Serial.println("ERROR: Memori tidak cukup untuk menjalankan FreeRTOS!");
        } else {
            Serial.println("ERROR: Gagal membuat Queue!");
        }
    }

    void loop() {
    }

    void read_data(void *pvParameters) {
        struct readings x;

        for (;;) {
            float humidity = dht.readHumidity();
            float temperature = dht.readTemperature();

            if (isnan(humidity) || isnan(temperature)) {
                Serial.println("Gagal membaca data dari sensor DHT!");
            } else {
                x.temp = temperature;
                x.h = humidity;
                xQueueSend(my_queue, &x, portMAX_DELAY);
            }

            vTaskDelay(2000 / portTICK_PERIOD_MS);
        }
    }

    void display(void *pvParameters) {
        struct readings x;

        for (;;) {
            if (xQueueReceive(my_queue, &x, portMAX_DELAY) == pdPASS) {
                Serial.print("Temperature = ");
                Serial.print(x.temp);
                Serial.println(" *C");

                Serial.print("Humidity    = ");
                Serial.print(x.h);
                Serial.println(" %");
                Serial.println("----------------------");
            }
        }
    }
    ```
    Penjelasan:
    Pada program tersebut, digunakan sensor DHT22 sesungguhnya untuk membaca nilai suhu dan kelembapan secara real time. Task read_data membaca nilai sensor dan mengirimkannya ke task display melalui queue. Task display menerima nilai sensor dari queue dan menampilkannya ke Serial Monitor. Hasilnya nilai suhu dan kelembapan berubah secara dinamis, data berhasil dikirim antar task menggunakan queue, tidak terjadi race condition.
