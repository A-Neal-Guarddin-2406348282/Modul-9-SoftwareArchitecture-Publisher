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