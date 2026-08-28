# ThesisBreaker AI — Sectors Configuration

Dokumen ini mendefinisikan konfigurasi dan strategi penggunaan integrasi Sectors untuk **ThesisBreaker AI**.

> **Project:** ThesisBreaker AI  
> **Track:** Track 1 — AI Agents & Assistants  
> **Core purpose:** Membantu pengguna menguji, menantang, dan menemukan kontradiksi dalam thesis atau asumsi investasi menggunakan data Sectors dan analisis AI.
>
> **Penting:** Akses endpoint dan konsumsi credit harus selalu disesuaikan dengan entitlement akun Sectors dan API credits hackathon yang tersedia. Jangan mengasumsikan seluruh fitur premium dapat diakses tanpa verifikasi.

---

## 1. Feature Integrations

ThesisBreaker AI menggunakan empat kapabilitas utama:

1. **Sectors Financial API** — sumber data finansial terstruktur utama.
2. **Financial Search Engine** — pencarian perusahaan dan data finansial berbasis query.
3. **Financial Analytics** — analisis, perbandingan, dan pengolahan insight dari data.
4. **IDX Companies** — konteks perusahaan Indonesia, sektor, performa, dan klasifikasi.

Prinsip utama:

```text
User Thesis
    ↓
AI Claim Extraction
    ↓
Sectors Data Retrieval
    ↓
Evidence Analysis
    ↓
Bull vs Bear Comparison
    ↓
Contradiction Detection
    ↓
ThesisBreaker Report
```

---

# 2. Sectors Financial API

## 2.1 Role

**Sectors Financial API adalah core data source ThesisBreaker AI.**

Tanpa data dari Sectors Financial API, fitur utama analisis thesis tidak dapat berjalan secara lengkap.

API digunakan untuk mengambil data faktual yang dibutuhkan AI sebelum memberikan hasil analisis.

## 2.2 Primary Use Cases

Gunakan API untuk:

- Mengambil data perusahaan IDX.
- Mengambil laporan atau data detail perusahaan.
- Melakukan company screening.
- Membandingkan performa perusahaan.
- Mengambil ranking yang relevan.
- Mengambil data IPO dan performance bila tersedia.
- Mengambil news atau filings secara selektif sebagai evidence.
- Memvalidasi klaim pengguna menggunakan data aktual.

## 2.3 API Configuration

### Environment Variables

```env
SECTORS_API_BASE_URL=https://api.sectors.app
SECTORS_API_KEY=
SECTORS_API_TIMEOUT=30
SECTORS_API_CACHE_TTL=3600
SECTORS_API_RETRY=2
```

> API key tidak boleh di-commit ke repository publik.

Gunakan `.env` untuk menyimpan API key:

```env
SECTORS_API_KEY=your_api_key_here
```

Tambahkan ke `.gitignore`:

```gitignore
.env
.env.*
!.env.example
```

Buat `.env.example`:

```env
SECTORS_API_BASE_URL=https://api.sectors.app
SECTORS_API_KEY=
SECTORS_API_TIMEOUT=30
SECTORS_API_CACHE_TTL=3600
SECTORS_API_RETRY=2
```

## 2.4 Laravel Configuration

Buat konfigurasi:

`config/services.php`

```php
'sectors' => [
    'base_url' => env('SECTORS_API_BASE_URL', 'https://api.sectors.app'),
    'api_key' => env('SECTORS_API_KEY'),
    'timeout' => env('SECTORS_API_TIMEOUT', 30),
],
```

Gunakan HTTP Client Laravel:

```php
$response = Http::withHeaders([
    'Authorization' => config('services.sectors.api_key'),
])->timeout(config('services.sectors.timeout'))
  ->get($url);
```

> Endpoint final harus mengikuti dokumentasi Sectors dan entitlement akun yang aktif.

## 2.5 API Request Rules

Setiap request harus:

1. Dijalankan dari backend Laravel.
2. Tidak pernah mengekspos API key ke React/browser.
3. Memiliki timeout.
4. Memiliki error handling.
5. Memiliki logging tanpa mencatat API key.
6. Menggunakan cache jika data tidak perlu real-time.
7. Meminimalkan request duplikat.
8. Menyimpan metadata penggunaan credit jika informasi tersebut tersedia.

## 2.6 Recommended Endpoint Priority

### Priority 1 — Core

Gunakan terlebih dahulu:

- Company detail/report.
- Company screener.
- Company performance.
- Rankings.

### Priority 2 — Evidence

Gunakan ketika relevan dengan thesis:

- News.
- Filings.
- Corporate information.

### Priority 3 — Optional

Gunakan hanya jika tersedia dan benar-benar diperlukan:

- Transaction data.
- Broker data.
- Advanced datasets.
- Regional market data.

## 2.7 Credit Optimization

Karena API credits terbatas, jangan melakukan request API untuk setiap aksi UI.

Gunakan strategi:

```text
User types thesis
        ↓
AI extracts company/symbol
        ↓
Check cache
   ↙             ↘
Hit              Miss
↓                 ↓
Use cache      Call Sectors API
                   ↓
                Save cache
                   ↓
              Analyze evidence
```

### Cache Strategy

Rekomendasi awal:

- Company profile: 24 jam.
- Sector/classification: 24 jam.
- Rankings: 6–24 jam.
- Historical performance: 6–24 jam.
- News/filings: 30–60 menit atau sesuai kebutuhan.
- Identical thesis analysis: cache hasil sementara berdasarkan hash input.

---

# 3. Financial Search Engine

## 3.1 Role

Financial Search Engine digunakan sebagai lapisan **financial discovery dan retrieval**.

Tujuannya membantu ThesisBreaker AI menemukan perusahaan, data, atau konteks finansial yang relevan sebelum proses analisis lebih dalam.

## 3.2 Primary Use Cases

Contoh query:

```text
"Bank Indonesia dengan pertumbuhan laba tertinggi"
```

```text
"Perusahaan consumer dengan performa lebih baik dari sektornya"
```

```text
"Bandingkan perusahaan A dengan perusahaan B"
```

```text
"Perusahaan dengan ownership structure tertentu"
```

## 3.3 ThesisBreaker AI Integration

Workflow:

```text
User Question / Thesis
        ↓
Search Intent Detection
        ↓
Financial Search Engine
        ↓
Candidate Companies / Financial Context
        ↓
Select Relevant Evidence
        ↓
Sectors Financial API
        ↓
Deep Analysis
```

## 3.4 Search Rules

Financial Search Engine **bukan sumber dekoratif**.

Hasil pencarian harus dipakai dalam workflow nyata, misalnya:

- Menemukan kandidat perusahaan.
- Mengidentifikasi perusahaan dari bahasa natural.
- Menemukan data pembanding.
- Memperluas konteks sebuah claim.
- Menghasilkan evidence yang kemudian diverifikasi.

## 3.5 Request Optimization

Jangan melakukan pencarian setiap kali user mengetik.

Gunakan:

- Debounce 500–800 ms untuk search UI.
- Minimum query length 3 karakter.
- Server-side rate limiting.
- Cache untuk query populer.
- Maximum result limit yang wajar.
- Search hanya saat benar-benar dibutuhkan oleh agent.

## 3.6 Configuration

Tambahkan jika Search Engine memiliki endpoint/API terpisah:

```env
SECTORS_SEARCH_ENABLED=true
SECTORS_SEARCH_TIMEOUT=20
SECTORS_SEARCH_CACHE_TTL=1800
```

Jika Financial Search Engine menggunakan entitlement atau mekanisme akses berbeda, konfigurasi final harus mengikuti dokumentasi resmi yang tersedia pada akun Sectors.

---

# 4. Financial Analytics

## 4.1 Role

Financial Analytics digunakan untuk mengubah data mentah menjadi **contextual evidence**.

Feature ini tidak menjadi pengganti AI. Financial Analytics menyediakan data, metrik, perbandingan, atau konteks; sedangkan AI melakukan reasoning dan menjelaskan hubungan dengan thesis pengguna.

## 4.2 Primary Use Cases

Gunakan untuk:

- Perbandingan performa.
- Analisis perubahan periode.
- Perbandingan dengan sektor.
- Identifikasi outlier.
- Analisis tren.
- Membandingkan perusahaan dengan peer.
- Menemukan perubahan yang melemahkan atau mendukung thesis.

## 4.3 Example

User thesis:

> "Perusahaan X selalu memiliki pertumbuhan yang kuat."

Analytics dapat membantu memeriksa data historis:

```text
Claim:
"Pertumbuhan selalu kuat"
        ↓
Historical Analytics
        ↓
Growth: +12%, +8%, -4%, +15%
        ↓
Contradiction Found
        ↓
Verdict:
Partially Contradicted
```

## 4.4 Output for AI

Financial Analytics harus menghasilkan data terstruktur, bukan langsung membuat keputusan investasi.

Contoh internal structure:

```json
{
  "metric": "revenue_growth",
  "period": "historical",
  "trend": "volatile",
  "observations": [
    {
      "period": "P1",
      "value": 12
    },
    {
      "period": "P2",
      "value": 8
    },
    {
      "period": "P3",
      "value": -4
    }
  ],
  "potential_contradiction": true
}
```

AI kemudian menjelaskan:

```text
The claim that growth was consistently strong is not fully supported,
because the analyzed period includes a negative growth interval.
```

## 4.5 Configuration

```env
FINANCIAL_ANALYTICS_ENABLED=true
FINANCIAL_ANALYTICS_CACHE_TTL=3600
FINANCIAL_ANALYTICS_MAX_COMPARISONS=5
```

## 4.6 Analytics Rules

- Jangan menghitung ulang data yang sudah dicache.
- Maksimalkan processing dari data yang sudah diambil.
- Gunakan analytics setelah data retrieval, bukan dengan request tambahan yang tidak perlu.
- Batasi jumlah peer comparison dalam satu analisis.
- Simpan hasil intermediate analysis selama sesi pengguna.

---

# 5. IDX Companies

## 5.1 Role

IDX Companies adalah domain utama ThesisBreaker AI untuk MVP.

Fokus awal aplikasi:

```text
Indonesia Stock Exchange (IDX)
```

Hal ini membuat produk lebih fokus dan sesuai dengan konteks Indonesian financial markets.

## 5.2 Primary Use Cases

IDX Companies digunakan untuk:

- Company lookup.
- Symbol/ticker resolution.
- Company profile.
- Sector identification.
- Company classification.
- Peer comparison.
- Sector context.
- Performance context.

## 5.3 Company Resolution Workflow

User mungkin menulis:

```text
BBCA
```

atau:

```text
Bank Central Asia
```

atau:

```text
BCA
```

Sistem harus:

```text
User Input
    ↓
Company Resolver
    ↓
IDX Companies / Search
    ↓
Canonical Company
    ↓
Symbol
    ↓
Sectors API Data Retrieval
```

Contoh internal result:

```json
{
  "company_name": "Bank Central Asia",
  "symbol": "BBCA",
  "market": "IDX",
  "confidence": 0.98
}
```

## 5.4 Configuration

```env
IDX_MARKET_ENABLED=true
IDX_DEFAULT_MARKET=IDX
IDX_COMPANY_CACHE_TTL=86400
IDX_MAX_PEERS=5
```

## 5.5 MVP Scope

Untuk MVP:

- Fokus hanya IDX.
- Jangan membangun SGX dan KLSE pada versi pertama.
- Jangan menambahkan mining extension kecuali thesis demo memang membutuhkan dataset tersebut.
- Gunakan peer comparison maksimum 3–5 perusahaan.

---

# 6. Unified ThesisBreaker Workflow

Keempat feature di atas bekerja dalam satu workflow.

```text
┌───────────────────────────────┐
│          USER THESIS          │
│                               │
│ "Saya yakin X akan terus      │
│  tumbuh karena Y."            │
└───────────────┬───────────────┘
                ↓
┌───────────────────────────────┐
│      THESIS PARSER AGENT      │
│                               │
│ Extract:                      │
│ • Company                     │
│ • Claims                      │
│ • Assumptions                 │
│ • Time horizon                │
└───────────────┬───────────────┘
                ↓
       ┌────────┴─────────┐
       ↓                  ↓
┌──────────────┐   ┌──────────────────┐
│ IDX Companies│   │ Financial Search │
│ Resolution   │   │ Engine           │
└──────┬───────┘   └────────┬─────────┘
       └──────────┬──────────┘
                  ↓
┌───────────────────────────────┐
│     SECTORS FINANCIAL API     │
│                               │
│ • Reports                     │
│ • Screeners                   │
│ • Rankings                    │
│ • Performance                 │
│ • Evidence                    │
└───────────────┬───────────────┘
                ↓
┌───────────────────────────────┐
│      FINANCIAL ANALYTICS      │
│                               │
│ • Trend                       │
│ • Comparison                  │
│ • Peer context                │
│ • Outlier                     │
└───────────────┬───────────────┘
                ↓
       ┌────────┴────────┐
       ↓                 ↓
┌─────────────┐    ┌─────────────┐
│ BULL AGENT  │    │ BEAR AGENT  │
│ Supporting  │    │ Challenging │
│ Evidence    │    │ Evidence    │
└──────┬──────┘    └──────┬──────┘
       └─────────┬────────┘
                 ↓
┌───────────────────────────────┐
│    THESISBREAKER ENGINE       │
│                               │
│ • Contradictions              │
│ • Blind spots                 │
│ • Unsupported assumptions     │
│ • Evidence strength           │
└───────────────┬───────────────┘
                ↓
┌───────────────────────────────┐
│       FINAL AI REPORT         │
│                               │
│ 🟢 Supported                  │
│ 🟡 Partially Supported        │
│ 🔴 Contradicted               │
│ ⚪ Insufficient Evidence      │
└───────────────────────────────┘
```

---

# 7. Laravel Architecture

Recommended structure:

```text
app/
├── Actions/
│   └── Thesis/
├── Agents/
│   ├── ThesisParserAgent.php
│   ├── BullResearchAgent.php
│   ├── BearResearchAgent.php
│   └── ContradictionAgent.php
├── Services/
│   └── Sectors/
│       ├── SectorsApiService.php
│       ├── FinancialSearchService.php
│       ├── FinancialAnalyticsService.php
│       └── IdxCompanyService.php
├── DTOs/
│   ├── ThesisDTO.php
│   ├── ClaimDTO.php
│   └── EvidenceDTO.php
├── Jobs/
│   └── AnalyzeThesisJob.php
└── Http/
    └── Controllers/
        └── ThesisAnalysisController.php
```

## Service Responsibility

### `SectorsApiService`

Tanggung jawab:

- Authentication.
- HTTP requests.
- Error handling.
- Retry.
- Caching.
- Response normalization.

### `FinancialSearchService`

Tanggung jawab:

- Financial discovery.
- Company search.
- Query normalization.
- Candidate retrieval.

### `FinancialAnalyticsService`

Tanggung jawab:

- Trend calculation.
- Comparison.
- Peer context.
- Structured observations.

### `IdxCompanyService`

Tanggung jawab:

- Symbol resolution.
- Company metadata.
- Sector lookup.
- Peer identification.

---

# 8. React Frontend Configuration

React tidak boleh mengakses Sectors API secara langsung.

Arsitektur:

```text
React
  ↓
Laravel API
  ↓
ThesisBreaker Services
  ↓
Sectors API / Available Sectors Features
```

## Main Frontend Features

### Thesis Input

```text
Write your investment thesis...
```

### Company Detection

```text
Detected Company:
BBCA — Bank Central Asia
```

### Analysis Progress

```text
✓ Parsing thesis
✓ Resolving company
✓ Retrieving financial evidence
✓ Comparing historical data
✓ Challenging assumptions
✓ Building final report
```

### Final Result

```text
THESIS BREAK SCORE
72 / 100

🔴 2 claims contradicted
🟡 1 claim partially supported
🟢 3 claims supported
⚪ 1 claim lacks sufficient evidence
```

---

# 9. Credit Protection Rules

Credits adalah resource penting untuk hackathon.

## Mandatory Rules

1. Jangan call API dari frontend.
2. Jangan call endpoint yang sama berulang kali.
3. Gunakan Laravel cache.
4. Gunakan queue untuk analisis panjang.
5. Batasi maximum evidence per analysis.
6. Gunakan data yang sudah diambil untuk beberapa agent.
7. Jangan menjalankan Bull dan Bear Agent dengan request API terpisah jika evidence dapat dibagikan.
8. Simpan credits untuk testing dan demo akhir.

## Recommended Analysis Budget

Setiap thesis analysis harus mengikuti:

```text
Step 1: Resolve company
Step 2: Retrieve only required core data
Step 3: Reuse response
Step 4: Run analytics locally/server-side
Step 5: AI reasons over collected evidence
Step 6: Fetch additional evidence only if needed
```

Target:

```text
Minimal API requests
Maximum evidence reuse
Maximum credit efficiency
```

---

# 10. Error Handling

## API Error

Jika API gagal:

```json
{
  "status": "error",
  "message": "Financial data is temporarily unavailable.",
  "retryable": true
}
```

## Company Not Found

```json
{
  "status": "not_found",
  "message": "We could not confidently identify the company in your thesis."
}
```

## Insufficient Evidence

AI tidak boleh mengarang data.

Gunakan status:

```text
INSUFFICIENT EVIDENCE
```

Jangan mengubahnya menjadi:

```text
CONTRADICTED
```

jika data tidak cukup.

---

# 11. Financial Disclaimer

ThesisBreaker AI harus menampilkan disclaimer:

> ThesisBreaker AI provides financial information and analytical insights based on available data. It does not provide investment advice, recommendations, or instructions to buy or sell any financial instrument. Users remain responsible for their own financial decisions.

Bahasa Indonesia:

> ThesisBreaker AI menyediakan informasi dan analisis berdasarkan data yang tersedia. Aplikasi ini bukan merupakan nasihat investasi, rekomendasi, atau instruksi untuk membeli maupun menjual instrumen keuangan. Setiap pengguna tetap bertanggung jawab atas keputusan finansialnya sendiri.

---

# 12. Hackathon Compliance Checklist

## Sectors Integration

- [ ] Sectors data digunakan sebagai core data source.
- [ ] Produk kehilangan fungsi inti jika Sectors data dihapus.
- [ ] API/MCP digunakan dalam workflow nyata.
- [ ] Tidak ada API call dekoratif.

## Product

- [ ] User dapat memasukkan thesis.
- [ ] AI dapat mengekstrak claims.
- [ ] Sistem dapat mengambil data Sectors.
- [ ] Sistem dapat menganalisis evidence.
- [ ] Sistem dapat menemukan contradiction atau lack of evidence.
- [ ] Hasil dapat dilihat end-to-end.

## Security

- [ ] API key hanya di backend.
- [ ] `.env` tidak masuk GitHub.
- [ ] Repository publik tidak mengandung credential.
- [ ] Error log tidak mencetak API key.

## Credits

- [ ] Caching aktif.
- [ ] Duplicate requests dicegah.
- [ ] Demo credits disisakan.
- [ ] Endpoint usage diuji terlebih dahulu.
- [ ] Actual credit cost dicatat jika tersedia.

---

# 13. MVP Priority

## Phase 1 — Must Have

1. Thesis input.
2. IDX company detection.
3. Sectors Financial API integration.
4. Core company data retrieval.
5. Claim extraction.
6. Evidence analysis.
7. Contradiction detection.
8. Final report.
9. Financial disclaimer.

## Phase 2 — Strong Demo

1. Financial Search Engine.
2. Company comparison.
3. Rankings.
4. Performance analytics.
5. Bull vs Bear evidence.

## Phase 3 — Only If Time and Credits Allow

1. News/filings evidence.
2. Advanced peer analysis.
3. Multi-company thesis.
4. Historical thesis tracking.
5. Advanced analytics visualization.

---

# Final Configuration Principle

```text
Sectors Financial API
        =
Core factual evidence

Financial Search Engine
        =
Discovery and retrieval

Financial Analytics
        =
Context and data interpretation

IDX Companies
        =
Primary market universe

AI Agents
        =
Reasoning and thesis challenging

ThesisBreaker Engine
        =
Final contradiction detection
```

**The core rule for every implementation decision:**

> Retrieve only the Sectors data needed, reuse it across the analysis workflow, and let ThesisBreaker AI challenge assumptions using verifiable evidence rather than generating unsupported conclusions.
