# Membangun Chatbot Aplikasi yang Praktis

Panduan ramah pemula untuk membuat chatbot yang dapat menjawab pertanyaan dari **database aplikasi**, **dokumentasi internal**, dan secara opsional menggunakan **LLM**.

Panduan ini berfokus pada pola yang praktis:

> **Gunakan data aplikasi sebagai sumber kebenaran utama.**  
> **Gunakan NLP/routing untuk memahami apa yang diminta user.**  
> **Gunakan LLM untuk menjelaskan atau merangkum — bukan untuk mengarang data bisnis.**

Hasil akhirnya adalah chatbot yang lebih mirip **asisten aplikasi** daripada sekadar kotak chat AI umum.

---

## Daftar Isi

1. [Apa yang Akan Kita Bangun?](#1-apa-yang-akan-kita-bangun)
2. [Mental Model Utama](#2-mental-model-utama)
3. [Sebelum Menulis Code](#3-sebelum-menulis-code)
4. [Urutan Development yang Disarankan](#4-urutan-development-yang-disarankan)
5. [Langkah 1 — Buat Endpoint Chat](#5-langkah-1--buat-endpoint-chat)
6. [Langkah 2 — Buat UI Chat](#6-langkah-2--buat-ui-chat)
7. [Langkah 3 — Tambahkan Intent Detection](#7-langkah-3--tambahkan-intent-detection)
8. [Langkah 4 — Normalisasi Pertanyaan User](#8-langkah-4--normalisasi-pertanyaan-user)
9. [Langkah 5 — Ambil Data](#9-langkah-5--ambil-data)
10. [Langkah 6 — Format Response yang Stabil](#10-langkah-6--format-response-yang-stabil)
11. [Langkah 7 — Tambahkan Knowledge Retrieval / RAG](#11-langkah-7--tambahkan-knowledge-retrieval--rag)
12. [Langkah 8 — Tambahkan LLM](#12-langkah-8--tambahkan-llm)
13. [Langkah 9 — Tambahkan Authentication dan Permission](#13-langkah-9--tambahkan-authentication-dan-permission)
14. [Langkah 10 — Tambahkan Logging dan Observability](#14-langkah-10--tambahkan-logging-dan-observability)
15. [Bagaimana Alur Request Secara Penuh](#15-bagaimana-alur-request-secara-penuh)
16. [Struktur Backend Generik](#16-struktur-backend-generik)
17. [Cara Mendesain Intent](#17-cara-mendesain-intent)
18. [Database vs Knowledge vs LLM](#18-database-vs-knowledge-vs-llm)
19. [Strategi Testing](#19-strategi-testing)
20. [Kesalahan yang Sering Terjadi](#20-kesalahan-yang-sering-terjadi)
21. [Mengembangkan Chatbot Lebih Lanjut](#21-mengembangkan-chatbot-lebih-lanjut)
22. [Opsional: Page-Aware Chat](#22-opsional-page-aware-chat)
23. [Struktur Project yang Disarankan](#23-struktur-project-yang-disarankan)
24. [Checklist untuk Menggunakan Pola Ini di Aplikasi Lain](#24-checklist-untuk-menggunakan-pola-ini-di-aplikasi-lain)
25. [Glosarium](#25-glosarium)
26. [Latihan Belajar](#26-latihan-belajar)

---

# 1. Apa yang Akan Kita Bangun?

Bayangkan kamu punya sebuah aplikasi internal.

Aplikasi tersebut sudah memiliki:

- user,
- customer,
- order,
- device,
- invoice,
- ticket,
- status,
- report,
- dokumentasi.

Lalu kamu ingin user bisa bertanya seperti:

```text
berapa jumlah device aktif?
tampilkan order dari Jakarta
service ID site ABC apa?
cari serial number SN1234
tampilkan detail customer Acme
device mana yang sedang down?
bagaimana cara approve order?
status HOLD itu artinya apa?
```

Chatbot yang praktis tidak seharusnya menjawab semua pertanyaan itu dengan cara yang sama.

Ada pertanyaan yang harus mengambil data dari database.

Ada yang sebaiknya mencari dokumentasi.

Ada juga yang cocok dibantu LLM.

Perbedaan inilah yang menjadi fondasi arsitektur chatbot ini.

---

# 2. Mental Model Utama

Anggap chatbot sebagai beberapa sistem kecil yang bekerja bersama.

```text
User
  |
  v
Chat UI
  |
  v
Chat API / Controller
  |
  +--> Authentication
  |
  +--> Intent Detection
  |
  +--> Query Normalization
  |
  +--> Data Retrieval
  |      |
  |      +--> Database
  |      +--> API
  |      +--> Knowledge Base
  |
  +--> Response Formatter
  |
  +--> LLM (opsional)
  |
  +--> Logging
  |
  v
JSON Response
  |
  v
Chat UI
```

Chatbot **bukan hanya LLM**.

Dalam banyak chatbot aplikasi yang berguna, LLM hanya satu bagian kecil dari keseluruhan sistem.

---

## Aturan paling sederhana

Gunakan aturan berikut untuk menentukan sumber jawaban:

```text
FAKTA TENTANG KONDISI APLIKASI SAAT INI
→ Database / API

CARA KERJA / SOP / POLICY / DOKUMENTASI
→ Knowledge Base

PENJELASAN / RINGKASAN / BAHASA NATURAL
→ LLM
```

Contoh:

```text
"Berapa jumlah device aktif?"
→ database

"Bagaimana cara aktivasi device?"
→ dokumentasi

"Ringkas aktivitas operasional hari ini"
→ ambil data dari database dulu, lalu LLM merangkum
```

---

# 3. Sebelum Menulis Code

Ini adalah fase paling penting.

Jangan mulai dari:

```text
Model AI apa yang harus saya install?
```

Mulailah dari:

```text
User sebenarnya akan bertanya apa?
```

---

## 3.1 Kumpulkan pertanyaan user yang nyata

Tulis 10–30 pertanyaan yang realistis.

Untuk aplikasi inventory:

```text
cari device ABC
berapa device yang aktif?
tampilkan device di Bandung
serial number router A berapa?
tampilkan customer Acme
device mana yang sedang down?
bagaimana cara register device baru?
```

Untuk aplikasi finance:

```text
berapa invoice yang belum dibayar?
tampilkan invoice customer A
berapa total invoice bulan ini?
bagaimana cara membuat credit note?
```

Untuk aplikasi helpdesk:

```text
berapa ticket yang masih open?
tampilkan ticket yang assigned ke saya
cari ticket HD-2026-001
bagaimana cara escalate ticket?
```

Pertanyaan-pertanyaan tersebut akan menjadi requirement development chatbot.

---

## 3.2 Kelompokkan pertanyaan menjadi intent

**Intent** berarti:

> Apa yang sebenarnya ingin dilakukan user?

Contoh:

| Intent | Pertanyaan user |
|---|---|
| `device_search` | `cari device ABC` |
| `device_count` | `berapa device yang aktif?` |
| `device_serial` | `serial number router A berapa?` |
| `customer_search` | `tampilkan customer Acme` |
| `ticket_search` | `cari ticket HD-001` |
| `knowledge` | `bagaimana cara escalate ticket?` |

Jangan membuat satu intent untuk setiap kalimat.

Gabungkan pertanyaan yang melakukan operasi yang sama.

---

## 3.3 Cari source of truth

Untuk setiap intent, tentukan dengan pasti jawaban berasal dari mana.

Contoh:

| Intent | Sumber |
|---|---|
| `device_search` | tabel `devices` |
| `device_count` | `devices.status` |
| `customer_search` | tabel `customers` |
| `ticket_search` | tabel `tickets` |
| `knowledge` | dokumentasi Markdown |

Sebelum menulis code chatbot, tes datanya secara manual.

Contoh:

```sql
SELECT id, name, status
FROM devices
WHERE city = 'Bandung';
```

Kalau kamu belum bisa mengambil jawabannya secara konsisten secara manual, chatbot juga belum bisa menjawabnya dengan konsisten.

---

## 3.4 Pahami arti data

Jangan langsung berasumsi bahwa nama field memiliki arti tertentu.

Contoh:

```text
service_id
```

bisa saja isinya:

```text
M2M
VSAT
INTERNET
```

dan bukan identifier unik.

Selalu lihat data nyata terlebih dahulu.

Ini mencegah salah satu bug chatbot yang paling umum:

> Query-nya benar, tapi interpretasi field-nya salah.

---

## 3.5 Tentukan aturan akses

Tanyakan:

```text
Siapa yang boleh melihat data ini?
```

Contoh:

```text
sales user
→ data customer

operations user
→ status device

finance user
→ invoice

administrator
→ semuanya
```

Chatbot harus mengikuti permission yang sama dengan aplikasi.

Chatbot tidak boleh menjadi jalan pintas untuk melewati authorization.

---

## 3.6 Siapkan knowledge base

Gunakan dokumentasi untuk pertanyaan seperti:

```text
bagaimana cara submit order?
apa arti status ACTIVE?
bagaimana alur approval?
bagaimana proses billing?
```

Folder sederhana sudah cukup untuk awal:

```text
knowledge/
├── user_manual.md
├── operations.md
├── billing.md
└── troubleshooting.md
```

Kamu belum membutuhkan vector database di hari pertama.

---

# 4. Urutan Development yang Disarankan

Urutan development yang bagus:

```text
1. Endpoint dasar
2. UI chat dasar
3. Satu fitur pencarian database
4. Intent detection
5. Query normalization
6. Tambahkan capability database lainnya
7. Response formatter
8. Permission
9. Knowledge retrieval
10. LLM
11. Logging
12. Testing
13. Fitur lanjutan
```

Kenapa urutannya seperti ini?

Karena kamu perlu membuktikan dulu bahwa chatbot bisa menjawab pertanyaan nyata dari aplikasi **sebelum** menambah kompleksitas AI.

---

# 5. Langkah 1 — Buat Endpoint Chat

Buat satu endpoint backend.

Contoh:

```text
GET /chat?q=device+bandung
```

atau:

```text
POST /chat
{
  "message": "device bandung"
}
```

Contoh sederhana gaya Yii2:

```php
public function actionChat(string $q = ''): array
{
    Yii::$app->response->format = Response::FORMAT_JSON;

    $query = trim($q);

    if ($query === '') {
        return [
            'type' => 'message',
            'message' => 'Silakan masukkan pertanyaan.',
        ];
    }

    return [
        'type' => 'message',
        'message' => 'Pesan diterima: ' . $query,
    ];
}
```

Pada tahap ini belum ada AI.

Itu tidak masalah.

Pastikan dulu alur:

```text
browser
→ endpoint
→ JSON response
```

berjalan dengan benar.

---

# 6. Langkah 2 — Buat UI Chat

Frontend minimum membutuhkan:

```text
chat window
input
send button
message list
loading indicator
fetch()
response renderer
```

Alurnya:

```text
User mengetik
   ↓
JavaScript fetch()
   ↓
Backend endpoint
   ↓
JSON
   ↓
Render message
```

Usahakan business logic tidak ditaruh di JavaScript.

Frontend sebaiknya fokus pada:

```text
kirim pertanyaan
terima JSON
render response
```

---

## Contoh kontrak response

Text biasa:

```json
{
  "type": "message",
  "message": "Saat ini ada 128 device aktif."
}
```

List:

```json
{
  "type": "device-list",
  "message": "Saya menemukan 3 device.",
  "items": [
    {
      "id": "D001",
      "name": "Router Bandung",
      "status": "ACTIVE"
    }
  ]
}
```

Detail:

```json
{
  "type": "device-detail",
  "device": {
    "id": "D001",
    "name": "Router Bandung",
    "status": "ACTIVE"
  }
}
```

Menggunakan response type yang jelas membuat frontend lebih mudah diprediksi.

---

# 7. Langkah 3 — Tambahkan Intent Detection

Sekarang backend harus menentukan:

```text
User sebenarnya ingin apa?
```

Ini adalah layer NLP pertama.

---

## Mulai dari rule-based NLP

Untuk versi pertama, kamu tidak harus menggunakan model AI.

Contoh:

```php
private function looksLikeDeviceCountQuery(string $query): bool
{
    $hasCount = preg_match('/\b(how many|count|total|jumlah|berapa)\b/i', $query) === 1;
    $hasDevice = preg_match('/\b(device|router|equipment)\b/i', $query) === 1;

    return $hasCount && $hasDevice;
}
```

Contoh lain:

```php
private function looksLikeCustomerQuery(string $query): bool
{
    return preg_match('/\b(customer|company|client)\b/i', $query) === 1;
}
```

---

## Kenapa rule-based NLP berguna?

Karena:

- mudah di-debug,
- hasilnya predictable,
- murah,
- cepat,
- mudah diaudit,
- cocok untuk aplikasi domain-specific.

Kalau:

```text
"berapa device aktif?"
```

masuk ke intent yang salah, kamu bisa langsung memeriksa regex-nya.

Itu jauh lebih mudah daripada debugging model black-box.

---

# 8. Langkah 4 — Normalisasi Pertanyaan User

Intent detection memberi tahu kita **operasi apa yang diminta user**.

Normalization mengambil **keyword apa yang sebenarnya perlu dicari**.

Contoh:

```text
"tolong tampilkan semua device aktif yang ada di Bandung dong"
```

Keyword yang berguna mungkin hanya:

```text
Bandung
```

---

## Contoh

```php
private function stripDeviceQueryWords(string $query): string
{
    $clean = preg_replace(
        '/\b(tolong|tampilkan|cari|device|devices|aktif|di|kota|semua|dong)\b/i',
        ' ',
        $query
    ) ?? $query;

    $clean = trim(preg_replace('/\s+/', ' ', $clean) ?? $clean);

    return $clean;
}
```

Hasil:

```text
tolong tampilkan device aktif di kota Bandung dong
                        ↓
                     Bandung
```

---

## NLP punya dua pekerjaan berbeda

```text
Intent Detection
→ Operasinya apa?

Query Normalization
→ Keyword/entity-nya apa?
```

Pisahkan keduanya.

Dengan begitu code lebih mudah dibaca dan dikembangkan.

---

# 9. Langkah 5 — Ambil Data

Setelah intent diketahui, panggil function retrieval yang khusus.

Jangan letakkan semua SQL langsung di endpoint utama chatbot.

Kurang bagus:

```php
if (...) {
    // 60 baris SQL
}

if (...) {
    // 80 baris SQL lagi
}
```

Lebih bagus:

```php
if ($this->looksLikeDeviceCountQuery($query)) {
    $count = $this->countDevices(...);
}
```

Lalu:

```php
private function countDevices(?string $status = null): int
{
    // query database
}
```

---

## Contoh function pencarian

```php
private function searchDevices(string $keyword): array
{
    $like = '%' . $keyword . '%';

    return Yii::$app->db->createCommand(
        <<<'SQL'
SELECT id, name, serial_number, city, status
FROM devices
WHERE id LIKE :q
   OR name LIKE :q
   OR serial_number LIKE :q
   OR city LIKE :q
ORDER BY name
LIMIT 10
SQL,
        [':q' => $like],
    )->queryAll();
}
```

---

## Contoh function count

```php
private function countDevices(string $status): int
{
    return (int) Yii::$app->db->createCommand(
        'SELECT COUNT(*) FROM devices WHERE status = :status',
        [':status' => $status],
    )->queryScalar();
}
```

---

## Aturan penting

Untuk data aplikasi yang faktual:

```text
Pertanyaan user
      ↓
Intent
      ↓
Query database
      ↓
Jawaban
```

Bukan:

```text
Pertanyaan user
      ↓
LLM menebak jawabannya
```

---

# 10. Langkah 6 — Format Response yang Stabil

Row database dan response frontend jangan terlalu terikat langsung.

Misalnya database menghasilkan:

```php
[
    'customer_id' => 'C001',
    'customer_name' => 'Acme Jakarta',
    'status_aktif' => 'ACTIVE',
]
```

Buat formatter:

```php
private function formatCustomerItem(array $row): array
{
    return [
        'id' => (string) ($row['customer_id'] ?? ''),
        'name' => (string) ($row['customer_name'] ?? '-'),
        'status' => (string) ($row['status_aktif'] ?? '-'),
    ];
}
```

Sekarang kontrak API kamu menjadi:

```json
{
  "id": "C001",
  "name": "Acme Jakarta",
  "status": "ACTIVE"
}
```

---

## Kenapa formatter penting?

Tanpa formatter:

```text
Database schema
      ↓
Frontend
```

Dengan formatter:

```text
Database schema
      ↓
Formatter
      ↓
Stable API
      ↓
Frontend
```

Kalau database berubah, kamu hanya perlu menyesuaikan retrieval layer atau formatter.

---

# 11. Langkah 7 — Tambahkan Knowledge Retrieval / RAG

Tidak semua pertanyaan adalah query database.

Contoh:

```text
bagaimana alur approval order?
apa arti status HOLD?
bagaimana cara membuat customer?
```

Pertanyaan seperti itu lebih cocok dijawab dari knowledge base.

---

## Apa itu RAG?

RAG adalah singkatan dari:

```text
Retrieval-Augmented Generation
```

Konsep sederhananya:

```text
Pertanyaan user
      ↓
Cari dokumentasi yang relevan
      ↓
Berikan konteks tersebut ke language model
      ↓
Generate jawaban
```

---

## Bisa mulai tanpa embedding

Untuk dokumentasi yang masih kecil:

```text
tokenize query
      ↓
cari kata di Markdown
      ↓
beri score pada section yang cocok
      ↓
ambil hasil terbaik
```

Contoh:

```php
private function tokenizeKnowledgeQuery(string $query): array
{
    $stopWords = [
        'apa',
        'bagaimana',
        'yang',
        'itu',
        'tolong',
    ];

    $tokens = preg_split('/[^a-z0-9_.-]+/i', mb_strtolower($query)) ?: [];

    return array_values(array_filter(
        $tokens,
        fn ($token) => mb_strlen($token) >= 3
            && !in_array($token, $stopWords, true)
    ));
}
```

---

## Format Markdown yang disarankan

Markdown dengan heading yang jelas membantu retrieval.

```md
# Billing

## Invoice Status

ACTIVE berarti ...

## HOLD

HOLD berarti ...

## Generate Invoice

1. Buka Finance.
2. Pilih periode.
3. Klik Generate.
```

Heading tersebut nantinya bisa menjadi section retrieval.

---

# 12. Langkah 8 — Tambahkan LLM

Tambahkan LLM setelah chatbot sudah bisa mengambil informasi yang benar.

LLM sebaiknya digunakan untuk:

- summarization,
- explanation,
- rewriting,
- membuat jawaban lebih natural,
- menggabungkan knowledge yang sudah di-retrieve,
- mengubah statistik menjadi narasi.

---

## Penggunaan yang baik

Backend mengambil:

```json
{
  "active_devices": 128,
  "open_tickets": 12,
  "new_orders": 6
}
```

Lalu LLM menerima fakta tersebut dan menulis:

```text
Saat ini terdapat 128 device aktif. Tim support memiliki 12 ticket yang masih terbuka, sementara 6 order baru tercatat hari ini.
```

Faktanya tetap berasal dari aplikasi.

---

## Penggunaan yang buruk

Prompt:

```text
Berapa device aktif di sistem kami?
```

lalu berharap LLM mengetahuinya.

LLM tidak bisa tahu kondisi live database kecuali kamu mengambil dan memberikan datanya terlebih dahulu.

---

## Anggap LLM sebagai narator

Mental model yang berguna:

```text
Database = akuntan
Knowledge Base = perpustakaan
NLP router = resepsionis
LLM = penulis
```

Penulis tidak boleh mengarang angka milik akuntan.

---

## Selalu siapkan fallback jika memungkinkan

Contoh:

```php
$narrative = $llm->generateText($prompt);

if ($narrative === null) {
    $narrative = $this->buildFallbackSummary($stats);
}
```

Dengan begitu fitur penting tetap bisa dipakai walaupun model sedang tidak tersedia.

---

# 13. Langkah 9 — Tambahkan Authentication dan Permission

Endpoint chatbot tetaplah endpoint aplikasi.

Cek authentication terlebih dahulu:

```php
if (Yii::$app->user->isGuest) {
    return [
        'type' => 'message',
        'message' => 'Silakan login ulang.',
    ];
}
```

Kemudian cek permission.

Contoh:

```php
if (!Permission::can('device', 'view')) {
    return [
        'type' => 'message',
        'message' => 'Anda tidak memiliki akses melihat informasi device.',
    ];
}
```

---

## Lebih baik permission per capability

Daripada satu permission besar:

```text
chatbot.view
```

lebih baik pertimbangkan:

```text
device.view
customer.view
invoice.view
ticket.view
```

Dengan begitu chatbot mengikuti hak akses aplikasi yang sudah ada.

---

# 14. Langkah 10 — Tambahkan Logging dan Observability

Log hal-hal seperti:

```text
user
question
detected intent
response type
result count
duration
error
timestamp
```

Kenapa penting?

Karena pengembangan chatbot sangat bergantung pada cara user benar-benar menggunakannya.

Log membantu menjawab:

```text
User paling sering bertanya apa?
Query mana yang gagal?
Intent mana yang sering salah?
Pertanyaan apa yang menghasilkan 0 data?
Query mana yang lambat?
```

---

## Contoh

```php
ChatLog::record(
    userId: $userId,
    query: $query,
    intent: $intent,
    resultCount: $resultCount,
    durationMs: $duration
);
```

Log kemudian bisa menjadi dataset untuk meningkatkan chatbot.

---

# 15. Bagaimana Alur Request Secara Penuh

Misalnya user bertanya:

```text
berapa device aktif di Bandung?
```

Alurnya:

```text
1. Terima query
2. Cek login
3. Deteksi intent
      device_count
4. Deteksi status
      ACTIVE
5. Normalisasi query
      Bandung
6. Query database
      COUNT device
      WHERE status = ACTIVE
      AND city LIKE '%Bandung%'
7. Build response
8. Simpan log
9. Return JSON
```

LLM tidak diperlukan.

---

Pertanyaan lain:

```text
bagaimana cara aktivasi device?
```

Alurnya:

```text
1. Terima query
2. Tidak ada operational intent yang cocok
3. Cari knowledge files
4. Temukan section "Device Activation"
5. Opsional: berikan section ke LLM
6. LLM menjelaskan secara natural
7. Return response
```

---

Pertanyaan lain:

```text
ringkas operasional hari ini
```

Alurnya:

```text
1. Deteksi summary intent
2. Query statistik dari database
3. Bentuk structured JSON
4. Kirim JSON ke LLM
5. Validasi output
6. Gunakan fallback jika perlu
7. Return narasi
```

---

# 16. Struktur Backend Generik

Satu controller masih cukup untuk chatbot kecil.

Gunakan section yang jelas.

```php
class ChatController extends Controller
{
    // =========================================================
    // [1] ENTRY POINT / ROUTER
    // =========================================================

    public function actionChat(string $q = ''): array
    {
        // authentication
        // intent routing
        // fallback
    }


    // =========================================================
    // [2] LOGGING / RESPONSE
    // =========================================================

    private function logAndReturn(...): array
    {
    }


    // =========================================================
    // [3] DATA RETRIEVAL
    // =========================================================

    private function searchDevices(...): array
    {
    }

    private function countDevices(...): int
    {
    }


    // =========================================================
    // [4] FORMATTERS
    // =========================================================

    private function formatDeviceItem(...): array
    {
    }


    // =========================================================
    // [5] NLP - INTENT DETECTION
    // =========================================================

    private function looksLikeDeviceQuery(...): bool
    {
    }

    private function looksLikeDeviceCountQuery(...): bool
    {
    }


    // =========================================================
    // [6] NLP - QUERY NORMALIZATION
    // =========================================================

    private function stripDeviceQueryWords(...): string
    {
    }


    // =========================================================
    // [7] KNOWLEDGE / RAG
    // =========================================================

    private function searchKnowledge(...): array
    {
    }


    // =========================================================
    // [8] HELPERS
    // =========================================================

    private function formatDate(...): string
    {
    }
}
```

Struktur seperti ini membuat controller besar tetap mudah dipelajari.

---

## Kapan controller perlu dipecah?

Kalau sudah terlalu besar, kamu bisa pecah menjadi service:

```text
ChatController
      |
      +--> IntentRouter
      +--> DeviceChatService
      +--> CustomerChatService
      +--> KnowledgeService
      +--> LlmService
      +--> ChatLogger
```

Tapi jangan buru-buru refactor.

Controller 500 baris yang terstruktur dengan jelas kadang lebih mudah dipahami daripada 10 class kecil ketika project masih sering berubah.

---

# 17. Cara Mendesain Intent

Intent yang baik harus menunjuk ke capability yang konkret.

Bagus:

```text
device_search
device_count
ticket_search
invoice_total
customer_detail
knowledge
```

Terlalu umum:

```text
ask_question
general_info
data
```

Terlalu spesifik:

```text
device_count_active_bandung
device_count_active_jakarta
device_count_active_surabaya
```

Lebih baik:

```text
intent = device_count
status = ACTIVE
city = Bandung
```

Intent menjelaskan tindakan.

Entity menjelaskan parameternya.

---

## Intent + Entity

Pertanyaan:

```text
berapa device aktif di Bandung?
```

bisa direpresentasikan menjadi:

```json
{
  "intent": "device_count",
  "entities": {
    "status": "ACTIVE",
    "city": "Bandung"
  }
}
```

Kamu tidak perlu entity parser yang rumit di awal.

Helper function sederhana sering kali sudah cukup.

---

# 18. Database vs Knowledge vs LLM

Gunakan tabel keputusan berikut.

| Pertanyaan | Sumber terbaik |
|---|---|
| Berapa jumlah record aktif? | Database |
| ID customer A berapa? | Database |
| Serial number device A berapa? | Database |
| Total invoice berapa? | Database |
| Bagaimana cara membuat order? | Knowledge |
| Apa arti status HOLD? | Knowledge |
| Jelaskan workflow approval | Knowledge + LLM opsional |
| Ringkas operasional hari ini | Database + LLM |
| Tolong rapikan penjelasan ini | LLM |

Chatbot akan jauh lebih reliable kalau batas ini jelas.

---

# 19. Strategi Testing

Jangan hanya mengetes:

```text
Apakah chatbot membalas?
```

Tes keseluruhan routing behavior.

---

## 19.1 Buat test matrix

Contoh:

| Input | Expected Intent | Expected Keyword | Expected Source |
|---|---|---|---|
| `device Bandung` | device_search | Bandung | DB |
| `berapa device aktif?` | device_count | kosong | DB |
| `device aktif Bandung` | device_search/count | Bandung | DB |
| `serial number router ABC` | device_serial | ABC | DB |
| `bagaimana cara aktivasi device?` | knowledge | activate device | docs |

---

## 19.2 Tes variasi bahasa

User bisa mengetik:

```text
device bandung
devices di bandung
tampilkan device bandung
cari device bandung
device kota bandung dong
```

Idealnya semua diarahkan ke capability yang sama.

---

## 19.3 Tes kata ambigu

Beberapa kata bisa memiliki banyak arti.

Contoh:

```text
status
id
service
order
active
```

Gunakan data aplikasi nyata dan bahasa user nyata untuk memperbaiki rule.

---

## 19.4 Tes kondisi zero result

Bagus:

```text
Saya tidak menemukan device yang cocok dengan "XYZ".
```

Buruk:

```text
SQL error...
```

atau malah mengarang hasil.

---

## 19.5 Tes permission

Tes sebagai:

```text
admin
normal user
restricted user
guest
```

Chatbot tidak boleh membuka data lebih banyak daripada halaman aplikasi normal.

---

# 20. Kesalahan yang Sering Terjadi

## Kesalahan 1 — Mulai dari LLM

Urutan yang kurang tepat:

```text
install model
buat prompt
berharap model mengerti aplikasi
```

Lebih baik:

```text
tentukan capability
petakan data
buat retrieval
baru tambahkan LLM
```

---

## Kesalahan 2 — Membiarkan LLM menjawab data bisnis live

Jangan minta model mengarang:

```text
status saat ini
jumlah saat ini
invoice saat ini
device saat ini
data customer
serial number
```

Ambil faktanya terlebih dahulu.

---

## Kesalahan 3 — Menebak arti field database

Field bernama:

```text
service_id
```

belum tentu berarti seperti yang kamu kira.

Periksa nilai sebenarnya.

---

## Kesalahan 4 — Satu generic SQL search untuk semuanya

Generic search bagus sebagai fallback.

Tapi pertanyaan khusus harus punya query khusus.

Contoh:

```text
berapa device aktif?
```

seharusnya menggunakan:

```sql
COUNT(*)
```

bukan mengambil 10 row lalu menghitung hasil tersebut.

---

## Kesalahan 5 — Tidak punya response schema

Mengembalikan struktur JSON yang berubah-ubah membuat frontend susah dikembangkan.

Gunakan response type yang konsisten.

---

## Kesalahan 6 — Mengabaikan authorization

Chatbot bisa tanpa sengaja membuka:

```text
data finance
customer milik user lain
catatan internal
informasi pribadi
```

Gunakan kembali permission aplikasi.

---

## Kesalahan 7 — NLP terlalu rumit terlalu cepat

Untuk versi awal, kemungkinan kamu belum butuh:

```text
fine-tuned classifier
vector database besar
multi-agent architecture
```

Mulailah dari rule dan log penggunaan nyata.

---

## Kesalahan 8 — Tidak ada log

Tanpa log, kamu akan terus menebak user sebenarnya membutuhkan apa.

Dengan log, improvement bisa berdasarkan bukti.

---

# 21. Mengembangkan Chatbot Lebih Lanjut

Arsitektur yang bagus memungkinkan satu layer berkembang tanpa harus menulis ulang semuanya.

---

## Tahap 1 — Rule-Based Router

```text
regex
keyword
simple entity extraction
```

Cocok untuk:

```text
aplikasi internal kecil
vocabulary yang cukup dikenal
operasi bisnis yang jelas
```

---

## Tahap 2 — Entity Extraction yang Lebih Baik

Tambahkan:

```text
date parsing
city parsing
status parsing
ID recognition
number extraction
```

Contoh:

```text
tampilkan ticket dari 1 September sampai 7 September
```

---

## Tahap 3 — Semantic Knowledge Search

Ganti keyword retrieval menjadi:

```text
embeddings
vector search
semantic similarity
```

Bagian chatbot lainnya bisa tetap sama.

---

## Tahap 4 — LLM Intent Classification

Untuk bahasa user yang lebih fleksibel, model bisa menghasilkan:

```json
{
  "intent": "device_count",
  "status": "ACTIVE",
  "city": "Bandung"
}
```

Tapi semua output tetap harus divalidasi sebelum digunakan untuk query.

Jangan mengeksekusi SQL yang dibuat langsung oleh model tanpa safety layer yang ketat.

---

## Tahap 5 — Tool Calling

Chatbot bisa memiliki tool yang sudah disetujui:

```text
search_device
count_device
search_customer
get_invoice
search_docs
```

Model memilih tool.

Backend tetap mengontrol apa yang boleh dilakukan setiap tool.

---

## Tahap 6 — Actions

Nantinya chatbot bisa melakukan:

```text
create ticket
approve request
update status
send email
```

Ini jauh lebih berisiko daripada chatbot read-only.

Sebelum menambah action, siapkan:

- permission check,
- confirmation,
- audit log,
- validation,
- idempotency,
- rollback jika diperlukan.

---

# 22. Opsional: Page-Aware Chat

Chatbot juga bisa mengetahui halaman yang sedang dibuka user.

Contoh:

```text
User sedang membuka:
Device Detail / D001

User bertanya:
"kenapa device ini down?"
```

Frontend mengirim:

```json
{
  "message": "kenapa device ini down?",
  "context": {
    "page": "device-detail",
    "entity_type": "device",
    "entity_id": "D001"
  }
}
```

Backend menggunakan:

```text
entity_id = D001
```

untuk mengambil data terbaru.

Jangan mengirim seluruh HTML halaman kalau ID saja sudah cukup.

---

## Aturan precedence yang penting

Input eksplisit user harus menang.

Contoh:

```text
Current page = Device A

User bertanya:
"tampilkan Device B"
```

Gunakan:

```text
Device B
```

bukan context halaman saat ini.

Page context adalah bantuan tambahan, bukan pengganti pertanyaan user.

---

# 23. Struktur Project yang Disarankan

Versi sederhana:

```text
controllers/
└── ChatController.php

components/
└── LlmService.php

models/
└── ChatLog.php

knowledge/
├── user_manual.md
├── workflow.md
└── troubleshooting.md

views/
└── layouts/
    └── _chat.php
```

Kalau chatbot semakin besar:

```text
controllers/
└── ChatController.php

services/
├── ChatRouter.php
├── DeviceChatService.php
├── CustomerChatService.php
├── KnowledgeService.php
└── LlmService.php

repositories/
├── DeviceRepository.php
├── CustomerRepository.php
└── TicketRepository.php

models/
└── ChatLog.php

knowledge/
└── *.md
```

Gunakan struktur sederhana sampai kompleksitas benar-benar membutuhkan pemisahan lebih lanjut.

---

# 24. Checklist untuk Menggunakan Pola Ini di Aplikasi Lain

Saat menambahkan chatbot ke aplikasi baru, jawab dulu pertanyaan-pertanyaan berikut.

## Business

- [ ] Siapa yang akan menggunakan chatbot?
- [ ] Apa 10 pertanyaan paling penting?
- [ ] Jawaban mana yang harus real-time?
- [ ] Jawaban mana yang berasal dari dokumentasi?
- [ ] Apa yang tidak boleh diakses chatbot?

## Database

- [ ] Tabel mana yang menyimpan jawaban?
- [ ] View mana yang aman digunakan untuk read-only?
- [ ] Apa arti sebenarnya dari setiap field?
- [ ] Apakah sudah ada permission filter?
- [ ] Apakah query SQL sudah dites manual?

## NLP

- [ ] Apa saja intent yang dibutuhkan?
- [ ] Entity apa yang perlu diambil?
- [ ] Filler word apa yang harus dibuang?
- [ ] Intent mana yang harus diprioritaskan dibanding generic search?
- [ ] Apa yang dilakukan kalau query ambigu?

## Backend

- [ ] Apakah ada satu endpoint chat yang jelas?
- [ ] Apakah retrieval function sudah dipisahkan?
- [ ] Apakah response formatter sudah dipisahkan?
- [ ] Apakah permission dicek?
- [ ] Apakah error ditangani dengan aman?

## Knowledge

- [ ] Apakah dokumentasi akurat?
- [ ] Apakah heading-nya cukup jelas?
- [ ] Apakah keyword search cukup untuk awal?
- [ ] Apakah benar-benar perlu vector database sekarang?

## LLM

- [ ] Tugas spesifik apa yang dilakukan LLM?
- [ ] Apakah LLM menerima konteks yang terpercaya?
- [ ] Apakah ada fallback?
- [ ] Apakah output bisa divalidasi?
- [ ] Apakah LLM dicegah menjadi sumber fakta bisnis live?

## Frontend

- [ ] Apakah mendukung response `message`?
- [ ] Apakah mendukung list?
- [ ] Apakah mendukung detail card?
- [ ] Apakah ada loading state?
- [ ] Apakah nilai HTML di-escape?

## Testing

- [ ] Happy path
- [ ] Typo / slang
- [ ] Multiple matches
- [ ] Zero matches
- [ ] Unauthorized user
- [ ] LLM unavailable
- [ ] Database error
- [ ] Slow query

---

# 25. Glosarium

## Chat Endpoint

URL backend yang menerima pesan dari user.

---

## Intent

Operasi yang ingin dilakukan user.

Contoh:

```text
device_count
customer_search
invoice_total
```

---

## Entity

Parameter di dalam pertanyaan user.

Contoh:

```text
city = Bandung
status = ACTIVE
customer = Acme
```

---

## Intent Detection

Proses menentukan user ingin melakukan apa.

---

## Query Normalization

Proses membersihkan kata-kata natural yang tidak diperlukan dan mengambil keyword pencarian yang berguna.

---

## Retrieval

Proses mengambil informasi dari:

```text
database
API
dokumen
```

---

## RAG

Retrieval-Augmented Generation.

Ambil informasi yang relevan terlebih dahulu, lalu biarkan language model membuat jawaban berdasarkan informasi tersebut.

---

## LLM

Large Language Model.

Berguna untuk tugas bahasa, penjelasan, dan summarization.

LLM tidak seharusnya otomatis dianggap sebagai sumber data aplikasi live.

---

## Hallucination

Kondisi ketika model menghasilkan informasi yang terdengar masuk akal tetapi tidak memiliki dukungan atau salah.

---

## Response Formatter

Function yang mengubah raw row database menjadi struktur API yang stabil.

---

## Fallback

Alternatif aman saat metode utama gagal.

Contoh:

```text
LLM unavailable
→ deterministic summary
```

---

## Page-Aware Chat

Chatbot yang menerima metadata aman mengenai halaman yang sedang dilihat user.

---

# 26. Latihan Belajar

Kalau sedang belajar, coba bangun fitur berikut secara berurutan.

---

## Latihan 1 — Search

Buat:

```text
cari device Bandung
```

Skill yang dipelajari:

```text
endpoint
SQL search
JSON
frontend rendering
```

---

## Latihan 2 — Count

Buat:

```text
berapa device aktif?
```

Skill yang dipelajari:

```text
intent detection
status parsing
aggregate SQL
```

---

## Latihan 3 — Natural Language Cleanup

Support:

```text
tolong tampilkan device di Bandung
device kota Bandung
tampilkan semua device Bandung dong
```

Semua harus mencari:

```text
Bandung
```

Skill yang dipelajari:

```text
query normalization
```

---

## Latihan 4 — Knowledge

Support:

```text
bagaimana cara aktivasi device?
```

dari:

```text
knowledge/user_manual.md
```

Skill yang dipelajari:

```text
document retrieval
dasar RAG
```

---

## Latihan 5 — LLM Summary

Ambil:

```text
active devices
open tickets
new orders
```

Lalu minta model merangkumnya.

Skill yang dipelajari:

```text
structured context
prompting
fallback
```

---

## Latihan 6 — Permission

Pastikan user terbatas tidak bisa bertanya data finance lewat chatbot.

Skill yang dipelajari:

```text
authorization
security boundary
```

---

## Latihan 7 — Page Awareness

Di halaman detail:

```text
Device D001
```

support:

```text
"serial number-nya berapa?"
```

Skill yang dipelajari:

```text
frontend context
entity references
context precedence
```

---

# Arsitektur Akhir

Chatbot aplikasi yang reliable biasanya kurang lebih seperti ini:

```text
                        ┌─────────────────┐
                        │      USER       │
                        └────────┬────────┘
                                 │
                                 ▼
                        ┌─────────────────┐
                        │    CHAT UI      │
                        └────────┬────────┘
                                 │
                                 ▼
                        ┌─────────────────┐
                        │ AUTH / PERMISSION│
                        └────────┬────────┘
                                 │
                                 ▼
                        ┌─────────────────┐
                        │  INTENT ROUTER  │
                        └────────┬────────┘
                                 │
                    ┌────────────┼────────────┐
                    │            │            │
                    ▼            ▼            ▼
              ┌──────────┐ ┌──────────┐ ┌──────────┐
              │ DATABASE │ │   APIs   │ │KNOWLEDGE │
              └────┬─────┘ └────┬─────┘ └────┬─────┘
                   │            │            │
                   └────────────┼────────────┘
                                │
                                ▼
                       ┌─────────────────┐
                       │ RESPONSE FORMAT │
                       └────────┬────────┘
                                │
                            opsional
                                │
                                ▼
                       ┌─────────────────┐
                       │       LLM       │
                       └────────┬────────┘
                                │
                                ▼
                       ┌─────────────────┐
                       │   VALIDATION    │
                       └────────┬────────┘
                                │
                                ▼
                       ┌─────────────────┐
                       │    RESPONSE     │
                       └─────────────────┘
```

Ide paling pentingnya sederhana:

> **Bangun chatbot berdasarkan capability aplikasi dan data yang terpercaya, lalu tambahkan AI hanya di bagian yang memang terbantu dengan AI.**

Pendekatan ini lebih mudah di-debug, lebih aman digunakan, dan jauh lebih mudah dipakai ulang ketika kamu ingin menambahkan chatbot ke aplikasi lain.
