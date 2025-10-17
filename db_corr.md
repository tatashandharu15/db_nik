# Database Correlation Documentation (IPCDR)

## Overview
Dokumentasi ini menjelaskan korelasi dan hubungan antar data dalam berbagai file JSON yang terdapat dalam sistem IPCDR (IP Call Detail Record).

## File Structure Analysis

### 1. info_by_nik.json
**Deskripsi**: Data identitas lengkap berdasarkan NIK dengan foto dan informasi keluarga
**Total Records**: ~40,098 records
**Key Fields**:
- `nik` (Primary Key) - Nomor Induk Kependudukan
- `image` - Base64 encoded photo
- `nama_lengkap` - Nama lengkap
- `ttl` - Tempat tanggal lahir
- `nkk` - Nomor Kartu Keluarga
- `nama_lengkap_ayah`, `nama_lengkap_ibu` - Data orang tua
- `alamat`, `provinsi`, `kabupaten`, `kecamatan`, `kelurahan` - Data alamat
- `pekerjaan`, `pendidikan_terakhir` - Data sosial ekonomi

### 2. info_by_nik_clean.json
**Deskripsi**: Data identitas dalam format terstruktur dengan kode numerik
**Total Records**: ~25,002 records
**Key Fields**:
- `NIK` (Primary Key) - Nomor Induk Kependudukan
- `NAMA_LGKP` - Nama lengkap
- `JENIS_KLMIN` - Jenis kelamin (kode: 1=Laki-laki, 2=Perempuan)
- `TMPT_LHR`, `TGL_LHR` - Tempat dan tanggal lahir
- `GOL_DRH` - Golongan darah (kode numerik)
- `AGAMA` - Agama (kode numerik)
- `STAT_KWN` - Status perkawinan (kode numerik)
- `NO_KK` - Nomor Kartu Keluarga
- `NO_PROP`, `NO_KAB`, `NO_KEC`, `NO_KEL` - Kode wilayah administratif

### 3. info_by_nik_full.json
**Deskripsi**: Data identitas lengkap dengan informasi tambahan dokumen dan administrasi
**Total Records**: ~97,002 records
**Key Fields**:
- `NIK` (Primary Key) - Nomor Induk Kependudukan
- `NO_KTP` - Nomor KTP
- `NO_PASPOR` - Nomor paspor
- `AKTA_LHR`, `NO_AKTA_LHR` - Data akta kelahiran
- `AKTA_KWN`, `NO_AKTA_KWN`, `TGL_KWN` - Data akta perkawinan
- `AKTA_CRAI`, `NO_AKTA_CRAI`, `TGL_CRAI` - Data akta perceraian
- `NAMA_PET_REG`, `NIP_PET_REG` - Data petugas registrasi
- `TGL_ENTRI`, `TGL_UBAH` - Timestamp administrasi
- `TGL_CETAK_KTP`, `TGL_GANTI_KTP` - Data administrasi KTP

### 4. phone_number_nik.json
**Deskripsi**: Mapping antara nomor telepon dan NIK dengan data provider
**Total Records**: ~2,972 records
**Key Fields**:
- `msisdn` - Nomor telepon (format internasional)
- `nik` - NIK pemilik nomor telepon
- `provider` - Provider telekomunikasi (TELKOMSEL, INDOSAT, HUTCHISON_3, EXELCOMINDO)
- `reg_date` - Tanggal registrasi nomor

### 5. location_by_phone_number.json
**Deskripsi**: Data lokasi tracking berdasarkan nomor telepon
**Total Records**: ~1,164 records
**Key Fields**:
- `id` (Primary Key) - ID unik record
- `phone_number` - Nomor telepon
- `date` - Timestamp lokasi
- `cgi` - Cell Global Identity
- `coordinate` - Koordinat GPS (latitude,longitude)
- `imei` - International Mobile Equipment Identity
- `imsi` - International Mobile Subscriber Identity
- `address` - Alamat lengkap lokasi
- `maps` - Link Google Maps

### 6. ipcdr_account.json
**Deskripsi**: Data akun sistem IPCDR
**Total Records**: ~62 records
**Key Fields**:
- `id` (Primary Key) - ID unik akun
- `account_id` - ID akun
- `user_id` - ID pengguna
- `password` - Password terenkripsi
- `provider_id` - ID provider (credential)
- `created_at`, `updated_at` - Timestamp

## Data Correlations & Relationships

### Entity Relationship Diagram

```mermaid
graph TB
    subgraph "Identity Management"
        C[CITIZENS<br/>nik: PK<br/>nama_lengkap<br/>jenis_kelamin<br/>tempat_lahir<br/>tanggal_lahir<br/>no_kk: FK<br/>alamat<br/>provinsi<br/>kabupaten<br/>kecamatan<br/>kelurahan<br/>pekerjaan<br/>pendidikan_terakhir<br/>image_base64]
        
        F[FAMILY_CARDS<br/>no_kk: PK<br/>kepala_keluarga_nik: FK<br/>alamat_kk<br/>rt_rw]
    end
    
    subgraph "Communication Tracking"
        P[PHONE_REGISTRATIONS<br/>id: PK<br/>msisdn<br/>nik: FK<br/>provider<br/>reg_date]
        
        L[LOCATION_TRACKING<br/>id: PK<br/>phone_number<br/>timestamp<br/>latitude<br/>longitude<br/>cgi<br/>imei<br/>imsi<br/>address]
    end
    
    subgraph "System Management"
        S[SYSTEM_ACCOUNTS<br/>id: PK<br/>account_id<br/>user_id<br/>password_hash<br/>created_at<br/>updated_at]
    end
    
    C -->|"1:N memiliki"| P
    P -->|"1:N dilacak"| L
    F -->|"1:N anggota"| C
    
    style C fill:#e1f5fe
    style P fill:#f3e5f5
    style L fill:#ffebee
    style F fill:#e8f5e8
    style S fill:#fff3e0
```

### Data Flow Diagram

```mermaid
flowchart TD
    A[info_by_nik.json<br/>~40K records] --> D{NIK}
    B[info_by_nik_clean.json<br/>~25K records] --> D
    C[info_by_nik_full.json<br/>~97K records] --> D
    
    D --> E[phone_number_nik.json<br/>~3K records]
    E --> F{Phone Number}
    F --> G[location_by_phone_number.json<br/>~1K records]
    
    H[ipcdr_account.json<br/>~62 records] --> I[System Access Control]
    
    D --> J{Nomor KK}
    J --> K[Family Relationships]
    
    G --> L[Location Intelligence]
    K --> M[Demographic Analysis]
    L --> N[Surveillance & Tracking]
    M --> N
    
    style A fill:#e1f5fe
    style B fill:#e8f5e8
    style C fill:#fff3e0
    style E fill:#f3e5f5
    style G fill:#ffebee
    style H fill:#f1f8e9
```

### File Correlation Matrix

```mermaid
graph LR
    subgraph "Identity Data"
        A[info_by_nik.json]
        B[info_by_nik_clean.json]
        C[info_by_nik_full.json]
    end
    
    subgraph "Communication Data"
        D[phone_number_nik.json]
        E[location_by_phone_number.json]
    end
    
    subgraph "System Data"
        F[ipcdr_account.json]
    end
    
    A -.->|NIK| B
    A -.->|NIK| C
    A -.->|NIK| D
    B -.->|NIK| C
    B -.->|NIK| D
    C -.->|NIK| D
    D -.->|Phone Number| E
    
    A -.->|NO_KK| B
    B -.->|NO_KK| C
    
    style A fill:#ffcdd2
    style B fill:#c8e6c9
    style C fill:#ffe0b2
    style D fill:#e1bee7
    style E fill:#ffcdd2
    style F fill:#dcedc8
```

### Primary Correlations

#### 1. NIK sebagai Primary Key
```
info_by_nik.json (nik) ←→ info_by_nik_clean.json (NIK)
info_by_nik.json (nik) ←→ info_by_nik_full.json (NIK)
info_by_nik.json (nik) ←→ phone_number_nik.json (nik)
```

#### 2. Phone Number Correlation
```
phone_number_nik.json (msisdn) ←→ location_by_phone_number.json (phone_number)
```

#### 3. Family Relationship (Kartu Keluarga)
```
info_by_nik.json (nkk) ←→ info_by_nik_clean.json (NO_KK)
info_by_nik_clean.json (NO_KK) ←→ info_by_nik_full.json (NO_KK)
```

### Secondary Correlations

#### 1. Geographic Data
- Wilayah administratif dapat dikorelasikan antara:
  - `info_by_nik.json` (provinsi, kabupaten, kecamatan, kelurahan)
  - `info_by_nik_clean.json` (NO_PROP, NO_KAB, NO_KEC, NO_KEL)
  - `location_by_phone_number.json` (address)

#### 2. Device Tracking
- IMEI dan IMSI dalam `location_by_phone_number.json` dapat digunakan untuk tracking perangkat
- CGI (Cell Global Identity) untuk identifikasi cell tower

## Database Schema Recommendations

### Tabel Utama yang Disarankan:

#### 1. citizens (dari info_by_nik_*)
```sql
CREATE TABLE citizens (
    nik VARCHAR(16) PRIMARY KEY,
    nama_lengkap VARCHAR(255),
    jenis_kelamin ENUM('LAKI-LAKI', 'PEREMPUAN'),
    tempat_lahir VARCHAR(255),
    tanggal_lahir DATE,
    no_kk VARCHAR(16),
    alamat TEXT,
    provinsi VARCHAR(100),
    kabupaten VARCHAR(100),
    kecamatan VARCHAR(100),
    kelurahan VARCHAR(100),
    -- fields lainnya
    INDEX idx_no_kk (no_kk),
    INDEX idx_nama (nama_lengkap)
);
```

#### 2. phone_registrations (dari phone_number_nik.json)
```sql
CREATE TABLE phone_registrations (
    id INT AUTO_INCREMENT PRIMARY KEY,
    msisdn VARCHAR(15),
    nik VARCHAR(16),
    provider ENUM('TELKOMSEL', 'INDOSAT', 'HUTCHISON_3', 'EXELCOMINDO'),
    reg_date DATE,
    FOREIGN KEY (nik) REFERENCES citizens(nik),
    INDEX idx_msisdn (msisdn),
    INDEX idx_nik (nik)
);
```

#### 3. location_tracking (dari location_by_phone_number.json)
```sql
CREATE TABLE location_tracking (
    id INT AUTO_INCREMENT PRIMARY KEY,
    phone_number VARCHAR(15),
    timestamp DATETIME,
    latitude DECIMAL(10, 8),
    longitude DECIMAL(11, 8),
    cgi VARCHAR(20),
    imei VARCHAR(15),
    imsi VARCHAR(15),
    address TEXT,
    INDEX idx_phone (phone_number),
    INDEX idx_timestamp (timestamp),
    INDEX idx_imei (imei)
);
```

#### 4. system_accounts (dari ipcdr_account.json)
```sql
CREATE TABLE system_accounts (
    id VARCHAR(26) PRIMARY KEY,
    account_id VARCHAR(26),
    user_id VARCHAR(26),
    password_hash TEXT,
    created_at TIMESTAMP,
    updated_at TIMESTAMP
);
```

## Data Quality Issues

### 1. Inkonsistensi Format
- NIK format berbeda antara file (dengan/tanpa quotes)
- Nomor telepon format berbeda (dengan/tanpa prefix 62)
- Timestamp format bervariasi

### 2. Data Missing/NULL
- Banyak field NULL dalam info_by_nik_full.json
- Beberapa NIK tidak memiliki korelasi phone number

### 3. Duplikasi Data
- Kemungkinan duplikasi NIK antar file
- Satu NIK bisa memiliki multiple phone numbers

## Rekomendasi Implementasi

### 1. Data Normalization
- Standardisasi format NIK (16 digit)
- Standardisasi format nomor telepon (62xxx)
- Unifikasi format timestamp

### 2. Data Validation
- Validasi NIK format dan checksum
- Validasi nomor telepon Indonesia
- Validasi koordinat GPS

### 3. Indexing Strategy
- Primary index pada NIK
- Secondary index pada nomor telepon
- Spatial index untuk koordinat lokasi
- Composite index untuk query kompleks

### 4. Privacy & Security
- Enkripsi data sensitif (NIK, nomor telepon)
- Audit trail untuk akses data
- Role-based access control

## Kesimpulan

Data IPCDR memiliki struktur yang kompleks dengan multiple correlation points. NIK berfungsi sebagai primary key utama yang menghubungkan data identitas, sementara nomor telepon menjadi bridge ke data lokasi tracking. Implementasi database yang tepat memerlukan normalisasi data, validasi yang ketat, dan strategi indexing yang optimal untuk mendukung query performance yang baik.