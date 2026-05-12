# Tutorial 9 - Publisher

***Oleh Neal Guarddin - 2406348282***

### a. Berapa banyak data yang dikirimkan oleh program *publisher* ke *message broker* dalam satu kali jalan (*run*)?

Dalam satu kali eksekusi (satu kali *run*), program *publisher* saya akan menembakkan tepat **5 buah data (pesan)** ke *message broker*.

Jika melihat pada file `src/main.rs`, terdapat pemanggilan *method* `publish_event` yang dilakukan sebanyak lima kali secara berturut-turut. Setiap barisnya mengirimkan *event* `"user_created"` dengan muatan objek `UserCreatedEventMessage` yang isinya berbeda-beda (dengan parameter `user_id` 1 sampai 5 dan `user_name` unik seperti Amir, Neal, Rina, Tina, dan Siti).

Berikut adalah potongan kode yang mengeksekusi pengiriman lima pesan tersebut:

```rust
fn main() {
    let mut p = CrosstownBus::new_queue_publisher("amqp://guest:guest@localhost:5672".to_owned()).unwrap();
    
    _ = p.publish_event("user_created".to_owned(), UserCreatedEventMessage {
        user_id: "1".to_owned(), user_name: "2406348281-Amir".to_owned() });
    _ = p.publish_event("user_created".to_owned(), UserCreatedEventMessage {
        user_id: "2".to_owned(), user_name: "2406348282-Neal".to_owned() });
    _ = p.publish_event("user_created".to_owned(), UserCreatedEventMessage {
        user_id: "3".to_owned(), user_name: "2406348283-Rina".to_owned() });
    _ = p.publish_event("user_created".to_owned(), UserCreatedEventMessage {
        user_id: "4".to_owned(), user_name: "2406348284-Tina".to_owned() });
    _ = p.publish_event("user_created".to_owned(), UserCreatedEventMessage {
        user_id: "5".to_owned(), user_name: "2406348285-Siti".to_owned() });
}
```

### b. Apa makna dari URL `"amqp://guest:guest@localhost:5672"` yang digunakan secara identik pada program subscriber?

Penggunaan alamat yang seragam ini menandakan bahwa kedua program tersebut saling berkomunikasi melalui **satu *instance message broker* (RabbitMQ) yang identik**.

Dalam sistem *message-oriented middleware*, URL ini berfungsi sebagai titik temu utama. Agar proses pertukaran data bisa 
terjadi, *publisher* harus mengirimkan pesan ke lokasi yang sama dengan tempat *subscriber* menunggu. Jika terdapat perbedaan 
pada alamat atau port, pesan yang diterbitkan oleh *publisher* tidak akan pernah sampai karena *subscriber* mendengarkan pada 
jalur komunikasi yang berbeda.

### RabbitMQ Interface
![RabbitMQ Interface](assets/images/RabbitMQ-Interface.png)

### RabbitMQ Connection
![RabbitMQ Connection](assets/images/RabbitMQ-Connection.png)
![5 Message Events](assets/images/5MessageEvent.png)

Setelah publisher dan subscriber jalan, kita bisa melihat koneksinya aktif di dashboard RabbitMQ. Waktu kita jalankan 
`cargo run` di publisher, kelima pesan yang dikirim bakal langsung ditangkap dan diproses oleh subscriber yang lagi 
standby di antrean yang sama.

### RabbitMQ Monitoring
![RabbitMQ Monitoring](assets/images/RabbitMQ-SubscriberConnection.png)
![5 Message Events Spike](assets/images/5MessageEventSpike.png)

Begitu publisher kita jalankan, kelihatan ada lonjakan di grafik RabbitMQ karena ngirim 5 pesan sekaligus secara cepat. Saya eksekusi `cargo run` sebanyak 2 kali.
Setelah pesan masuk antrean, grafiknya bakal nunjukin kalau pesan tersebut langsung diproses sama subscriber. Ini jadi 
bukti kalau kedua layanan sudah berhasil saling komunikasi.

### Simulation Slow Subscriber
![RabbitMQ Connection Sleep](assets/images/RabbitMQ-ConnectionSleep.png)
![Console Connection Sleep](assets/images/ConsoleConnectionSleep.png)

Di sini kita bisa lihat kalau producer (pengirim) bisa terus-terusan mengirim data, dan semua data itu bakal numpuk dulu 
di antrean (message queue). Nantinya, consumer (penerima) akan memproses antrean pesan tersebut pelan-pelan, satu per satu.

### Many requests by typing `cargo run`
![RabbitMQ Many Requests](assets/images/RabbitMQ-ManyRequests.png)
![Console Many Requests](assets/images/ConsoleManyRequests.png)

**Mengapa lonjakan (spike) antrean pesan turun lebih cepat dari sebelumnya?**
Lonjakan antrean pesan pada grafik menurun jauh lebih cepat karena saat ini kita menjalankan **lebih dari satu program subscriber (minimal tiga)** secara bersamaan.

RabbitMQ secara otomatis mendistribusikan beban kerja tersebut ke semua *subscriber* yang sedang aktif secara bergantian 
z(*Round-Robin dispatching*). Ibaratnya, jika sebelumnya hanya ada 1 kasir yang melayani 5 pelanggan, sekarang ada 3 kasir yang bekerja bersamaan. Hal ini membuat proses pengolahan pesan di dalam antrean menjadi jauh lebih cepat. Konsep ini dikenal dengan sebutan **Competing Consumers Pattern**, di mana beberapa *consumer* (subscriber) bekerja sama untuk mengosongkan satu antrean yang sama.

**Apa yang bisa diperbaiki (improvement) dari kode publisher dan subscriber?**
Jika kita perhatikan kodenya, proses penanganan pesan di *subscriber* saat ini bersifat *synchronous* dan memblokir (blocking) eksekusi program, terutama jika kita menggunakan `thread::sleep` untuk mensimulasikan proses yang lambat.