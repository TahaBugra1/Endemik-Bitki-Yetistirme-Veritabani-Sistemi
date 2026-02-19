# 🌿 Endemik Bitki Yetiştirme Bölge Yönetim Sistemi

> Fırat Üniversitesi – BMÜ329 Veri Tabanı Sistemleri Dönem Projesi  
> SQL Server tabanlı, 3NF normalizasyon uygulanmış ilişkisel veri tabanı sistemi

---

## 📌 Proje Amacı

Bu sistem, Türkiye’deki **endemik bitki türlerinin** farklı coğrafi bölgelerdeki yetiştirilme potansiyelini analiz edebilmek amacıyla geliştirilmiştir.

Sistem;

- Bitki türlerini
- Coğrafi bölgeleri
- Habitat/iklim koşullarını
- Deneme süreçlerini
- Ölçüm verilerini

merkezi bir ilişkisel veri tabanı yapısında yönetir.

Amaç: Veri bütünlüğü korunmuş, normalizasyon kurallarına uygun ve transaction kontrollü bir akademik veri tabanı sistemi geliştirmektir.

---

## 🧱 Kullanılan Teknolojiler

- Microsoft SQL Server
- T-SQL (DDL & DML)
- Stored Procedure
- Trigger
- Transaction Yönetimi
- Index Yapıları
- 3NF Normalizasyon

---

## 🗄️ Veri Tabanı Mimarisi

### 📊 Ana Tablolar

| Tablo | Açıklama |
|-------|----------|
| `BITKI` | Endemik bitki bilgileri |
| `BOLGE` | Coğrafi bölge bilgileri |
| `HABITAT` | İklim ve toprak özellikleri |
| `KULLANICI` | Rol bazlı sistem kullanıcıları |
| `DENEME` | Yetiştirme denemeleri |
| `OLCUM` | Deneme sürecindeki ölçüm verileri |

---

### 🔗 İlişki Yapısı

- BITKI → DENEME (1:N)
- BOLGE → HABITAT (1:N)
- HABITAT → DENEME (1:N)
- KULLANICI → DENEME (1:N)
- DENEME → OLCUM (1:N)

---

## 🔐 Veri Bütünlüğü ve İş Kuralları

Sistem aşağıdaki kısıtları içerir:

- `FOREIGN KEY` ilişkileri
- `CHECK` constraint kontrolleri
- `UNIQUE` constraint
- `NOT NULL` zorunlulukları
- Başarı oranı: **0–100 arası**
- Başlangıç tarihi ≤ Bitiş tarihi
- Aynı bitki + habitat + başlangıç tarihi için mükerrer kayıt engelleme
- Ölçüm kaydı olmayan deneme `BASARILI/BASARISIZ` olamaz

---

## ⚙️ Stored Procedure

### `sp_DenemeVeIlkOlcumEkle`

Bu prosedür:

- Yeni bir deneme kaydı oluşturur
- Aynı transaction içinde ilk ölçüm kaydını ekler
- Hatalı durumda tüm işlemi geri alır (ROLLBACK)

### Transaction Yapısı

```sql
BEGIN TRAN

INSERT INTO Deneme (...)
VALUES (...);

DECLARE @deneme_id INT = SCOPE_IDENTITY();

INSERT INTO Olcum (...)
VALUES (...);

COMMIT
```

Hata durumunda:

```sql
ROLLBACK
```

Bu yapı sayesinde veri tutarlılığı garanti edilir.

---

## 🔁 Trigger

### `trg_Deneme_Durum_Kontrol`

Deneme durumu:

- `BASARILI`
- `BASARISIZ`

olarak güncellendiğinde:

✔ En az bir ölçüm kaydı olup olmadığı kontrol edilir  
❌ Ölçüm yoksa işlem iptal edilir (ROLLBACK)

Bu sayede mantıksal veri bütünlüğü korunur.

---

## 📈 Performans Optimizasyonu

Tanımlanan indexler:

- `IX_Deneme_Bitki`
- `IX_Deneme_Habitat`
- `IX_Deneme_Durum`
- `IX_Olcum_DenemeTarih`

Amaç:

- Sık kullanılan sorguların hızlandırılması
- Ortalama sorgu süresinin < 3 saniye olması

---

## 🧪 Örnek SQL Sorguları

### 🔎 Belirli bir bitkiye ait denemeler

```sql
SELECT d.deneme_id, b.bilimsel_ad, d.baslangic_tarihi, d.durum
FROM Deneme d
JOIN Bitki b ON d.bitki_id = b.bitki_id
WHERE b.bilimsel_ad = 'Astragalus gummifer';
```

---

### 🔎 Devam eden denemeler

```sql
SELECT d.deneme_id, k.ad_soyad, d.durum
FROM Deneme d
JOIN Kullanici k ON d.arastirmaci_id = k.kullanici_id
WHERE d.durum = 'DEVAM';
```

---

### 🔎 Belirli bir tarihten sonraki ölçümler

```sql
SELECT olcum_id, deneme_id, tarih
FROM Olcum
WHERE tarih >= '2025-01-01';
```

---

## 🧠 Normalizasyon

Veri modeli:

- 1NF → Atomik alanlar
- 2NF → Kısmi bağımlılık yok
- 3NF → Geçişli bağımlılık yok

Tüm tablolar **en az 3NF** seviyesindedir.

---

## 🚀 Kurulum

1. SQL Server açılır
2. Veri tabanı oluşturulur:

```sql
CREATE DATABASE EndemikBitkiDB;
USE EndemikBitkiDB;
```

3. Tablo oluşturma scriptleri çalıştırılır
4. Örnek veri scriptleri eklenir
5. Stored procedure ve trigger scriptleri yüklenir

---

## 👥 Takım

| İsim | Rol |
|------|------|
| Yusuf ÇINAR | Proje Lideri / Veri Tabanı Tasarım |
| Taha Buğra AK | SQL Geliştirici / Kalite Kontrol |
| Mustafa ÇEKCEOĞLU | Dokümantasyon / Test |

---

## 🎓 Akademik Kazanımlar

Bu proje kapsamında:

- E-R modelleme
- İlişkisel şemaya dönüşüm
- 3NF normalizasyon
- Constraint yönetimi
- Transaction kontrolü
- Stored Procedure geliştirme
- Trigger mantığı
- Hata senaryosu testleri

uygulanmıştır.

---

## 📌 Sonuç

Bu sistem:

- Veri bütünlüğünü önceliklendiren
- Kurallarla güvence altına alınmış
- Transaction kontrollü
- Akademik olarak doğru modellenmiş

bir SQL Server veri tabanı projesidir.
