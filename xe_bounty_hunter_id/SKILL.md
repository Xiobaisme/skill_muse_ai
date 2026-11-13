---
name: xe-bounty-hunter-id
description: Skill KHUSUS untuk bug bounty XE Network saja (bounty.xe.network, repo github.com/xeprotocol/xe dan xeprotocol/desk, testnet). Cari, verifikasi, cek duplikat, dan laporkan bug keamanan atau kebenaran logika. Gunakan setiap kali user menyebut bounty XE, XE Network, xeprotocol, tiket XE-####, review konsensus block-lattice, audit ledger lease/escrow di Go, atau meminta audit, triase, cek duplikat, atau penulisan report untuk program ini, walaupun user hanya bilang "bounty xe", "cek bug di xe", atau "kerjain bounty xe". JANGAN dipakai untuk platform, program, atau target lain.
---

# XE Bounty Hunter (Bahasa Indonesia)

Tujuan: menghasilkan report yang valid, bukan duplikat, dan bisa direproduksi untuk desk bounty XE.

Kualitas lebih penting daripada jumlah. Desk punya sekitar 950 tiket terbuka dan banyak yang kembar.

Bahasa kerja: balas user dalam bahasa Indonesia. Report ke desk ditulis dalam bahasa Inggris, kecuali docs program menyebut lain.

## BATASAN PLATFORM (baca dulu)

Skill ini hanya untuk program bug bounty XE Network. Target yang boleh disentuh hanyalah yang tercantum di halaman Scope desk, yaitu pada dasarnya:

- Kode di github.com/xeprotocol/xe dan github.com/xeprotocol/desk (membaca, build, dan test di mesin lokal)
- Testnet testnet-0005 dan desk bounty.xe.network, sesuai aturan Scope

Aturan keras:

- Kalau user minta memakai skill ini untuk platform, program, perusahaan, atau sistem lain (termasuk Immunefi, HackerOne, atau situs acak), tolak dan jelaskan bahwa skill ini khusus XE. Jangan mengarahkan teknik atau tools di sini ke target di luar scope.
- Apa pun yang tidak tercantum di Scope dianggap out-of-scope. Kalau ragu, berhenti dan tanya user.
- Jangan menyerang node milik pihak lain, infrastruktur desk, akun reporter lain, atau data yang bukan milikmu.

## 0. Sebelum mulai (kerjakan dulu)

Baca docs program sendiri. Jangan minta user menempelkannya. Fetch berurutan:

1. https://bounty.xe.network/llms.txt (instruksi untuk agent, ikuti)
2. https://bounty.xe.network/scope (apa yang masuk/tidak masuk scope, tier reward)
3. https://bounty.xe.network/v1 (API desk: endpoint, auth, format submit)
4. https://bounty.xe.network/signin (cara login: GitHub atau key sendiri)
5. https://bounty.xe.network/index.md (tampilan markdown dari desk)
6. https://github.com/xeprotocol/desk (source desk, untuk memahami API)

Aturan setelah fetch:

- Rangkum aturannya dalam 10 baris sebelum mengerjakan apa pun. Kalau docs bertentangan dengan skill ini, docs yang menang.
- Hanya jika benar-benar tidak bisa fetch (tidak ada tool jaringan/browse), beri tahu user tool atau akses apa yang dibutuhkan.
- Catat versi target: network testnet-0005 dan commit HEAD repo. Selalu catat hash commit yang kamu uji.
- Uji hanya yang in-scope. Buktikan bug DoS dan crash di node lokal atau unit test, jangan membanjiri node publik.

## 0.5 Akun dan penanganan key (identitas agent)

Desk menyediakan sign-in lewat GitHub dan lewat key milik sendiri, jadi agent bisa bekerja dengan identitasnya sendiri.

- Cari tahu alur persisnya sendiri dari /signin, /v1, dan llms.txt.
- Kalau ada langkah yang butuh manusia (misalnya persetujuan OAuth GitHub di browser), berhenti dan beri tahu user satu aksi yang dibutuhkan, lalu lanjut setelah key tersedia di environment variable. Jangan coba melewati login atau rate limit.
- Pakai key khusus untuk agent. Jangan pakai key atau seed wallet yang berisi dana asli.
- Simpan key hanya di environment variable atau secret manager (misalnya `XE_BOUNTY_KEY`).
  - Jangan hardcode di kode, jangan commit ke git, jangan tempel di chat, tiket, PoC, atau log.
- Masukkan file key dan `.env` ke `.gitignore`. Sensor key dari output command yang kamu kutip.
- Pakai izin minimum yang diizinkan desk. Untuk login GitHub, pilih token fine-grained atau read-only.
- Atur alamat payout dengan sengaja dan cek ulang sebelum report di-award. Ini data paling sensitif di akun.
- Kalau key mungkin bocor (tampil di output, ter-commit, terbagi), berhenti, revoke atau rotate, baru lanjut.
- Jangan kirim key ke host mana pun selain API desk resmi.

## 1. Tools dan setup lingkungan

Semua pengujian dijalankan di mesin lokal. Di awal kerja, cek tools yang tersedia:

```bash
for t in git go curl jq rg node npm python3 gh docker staticcheck govulncheck gosec semgrep; do
  command -v "$t" >/dev/null && echo "ok       $t" || echo "MISSING  $t"
done
```

Kalau ada yang hilang dan kamu punya izin, pasang yang diperlukan (jangan pasang yang tidak perlu):

```bash
go install honnef.co/go/tools/cmd/staticcheck@latest
go install golang.org/x/vuln/cmd/govulncheck@latest
go install github.com/securego/gosec/v2/cmd/gosec@latest
go install github.com/kisielk/errcheck@latest
go install go.uber.org/nilaway/cmd/nilaway@latest
pip install semgrep bandit pip-audit
```

Tools lain (golangci-lint, osv-scanner, trivy, CodeQL) dipasang sesuai docs resminya.

### Wajib

- Akses jaringan ke bounty.xe.network, github.com, dan proxy.golang.org
- Shell, git, Go toolchain, curl, jq, ripgrep (`rg`)

### Analisis kode Go

- staticcheck, golangci-lint, gosec, errcheck
- nilaway: deteksi nil dereference, cocok untuk bug panic
- semgrep dan CodeQL: buat aturan sendiri untuk sibling sweep
- ast-grep dan rg: cari pola di seluruh repo
- callgraph atau gocallvis: lihat alur pemanggilan

### Dynamic testing

- `go test -race`, `go test -fuzz`
- Property-based: rapid atau gopter (cocok untuk supply conservation dan unwind)
- Mutation testing: go-mutesting
- goleak: goroutine leak
- pprof dan hyperfine: ukur amplifikasi CPU/memori untuk bug DoS
- delve: debugging

### Dependency dan CVE

- govulncheck, osv-scanner, trivy, `go mod graph`
- Banyak temuan di desk berbentuk GO-2026-xxxx, jadi selalu cek dependency (libp2p, x/crypto, webtransport).

### Jaringan dan P2P (hanya testnet lokal)

- Jalankan beberapa node lokal (cek README repo; Docker Compose bila tersedia)
- `tc netem` dan toxiproxy: simulasi latensi dan partisi jaringan (penting untuk bug divergensi konsensus)
- `kill -9`: uji crash recovery dan restart
- tcpdump, Wireshark, ss

### API dan web

- Schemathesis atau Dredd: uji OpenAPI vs perilaku node (banyak report Medium soal spec mismatch)
- Spectral: lint OpenAPI
- Burp Suite atau ZAP: manual saja untuk desk dan wallet (auth, session, akses data privat)
- httpie, curl, jq

### SDK TypeScript dan desk Python

- eslint dengan plugin security, npm audit, `tsc --strict`
- bandit, pip-audit, pytest, hypothesis

### VM dan SSH (hanya di VM lokal sendiri)

- QEMU, Lima, ssh-audit, strace, lsof, cgroups

### Kerja dan report

- `git bisect`, `git blame`, tmux, asciinema (rekam repro)

### Dilarang

- Scanner otomatis dan tool brute force (sqlmap, nikto, gobuster, hydra, dan sejenisnya)
- Tool flood atau load test ke node publik atau desk live
- Metasploit dan exploit generik. Bukti yang baik berupa test Go yang spesifik ke kode XE.
- nmap hanya untuk node lokal sendiri, bukan ke host publik.

## 2. Area dan apa yang dicari

| Area | Jenis bug yang sering muncul di desk |
|---|---|
| Konsensus (`core/quorum`, `core/vote`) | finality divergen, snapshot bobot vote tidak cocok, posisi macet, error cascade/unwind konflik, deadlock quorum withhold |
| Leasing / ledger (`core/ledger`) | akuntansi escrow, stake, renew/settle/cancel/force-settle, refund hanya base cost, supply conservation, pembulatan emisi |
| Multisig | `IsSpendingOp` tidak mencakup semua operasi (bypass 1-of-N), batas ukuran keyset, panic pada pubkey pendek |
| P2P / sync (`net/*`) | gossip tidak divalidasi, rate limit per peer vs per IP, map/cache tanpa batas, baca seluruh chain per request, quarantine poisoning |
| Node / API (`node`, `api`) | nil deref/panic tanpa recover, lookup case-sensitive, pagination bolong, auth tunnel/SSH, provisioning VM |
| Store | MemStore vs BadgerStore berperilaku beda (bug diferensial) |
| Statechain / DAO | validasi timestamp/epoch, bypass write-once, network ID di hash, kekuasaan oracle key |
| SDK / CLI / web | penanganan bigint/uint64, command injection, path traversal, penyimpanan seed, CSP |
| Desk (Python) | auth, session, bocornya assessment yang masih ditahan, race condition |

## 3. Heuristik pencarian (hasil tinggi)

- **Sibling sweep:** kalau satu pola ketemu, grep pola yang sama di tempat lain (misalnya `IsSpendingOp` yang melewatkan operasi lease; slice `x[:12]` tanpa cek panjang; overflow di `now - timestamp`).
- **Cek diferensial:** jalankan urutan operasi yang sama di MemStore dan BadgerStore. Beda hasil berarti bug.
- **Simetri unwind/rollback:** untuk setiap perubahan state, cek jalur undo mengembalikan field yang sama persis.
- **Paritas validasi:** validasi staged-block, admission penuh, dan jalur sync (`skipTimestamp`) harus konsisten.
- **Batas:** setiap panjang, jumlah, map, atau channel yang dikontrol penyerang butuh cap dan timeout.
- **Konservasi:** total saldo + escrow + pending harus sama dengan supply setelah setiap operasi.
- **Regresi:** tiket berstatus patched bisa saja masih reproduce di HEAD. Verifikasi dengan test.
- **Spec vs implementasi:** bandingkan OpenAPI dengan respons node (status code, field, tipe, pagination).

## 4. Cek duplikat (wajib sebelum menulis)

1. Cari di desk (semua status: open, fixed, closed) dengan 3 sampai 5 variasi kata kunci: komponen, nama fungsi, gejala.
2. Baca setiap yang mirip. Root cause sama berarti duplikat, walaupun gejalanya beda.
3. Kalau temuan baru memperluas yang lama, tulis "Addendum to XE-####" dan jelaskan apa yang baru.
4. Kalau fix sudah rilis tapi bug masih reproduce, ajukan sebagai "Regression of XE-####" dengan hash HEAD.

## 5. Verifikasi

- Tulis test Go atau skrip yang gagal secara deterministik untuk mereproduksi bug.
- Konfirmasi di HEAD saat ini, bukan commit lama.
- Tulis apa yang kamu jalankan dan apa yang kamu lihat. Jangan bilang "terbukti" untuk hal yang tidak kamu eksekusi.
  - Tandai penalaran yang belum diuji sebagai "static analysis only".
- Untuk bug DoS, ukur dampak di node lokal (CPU, memori, latensi) dan sertakan angkanya.

## 6. Severity (klaim dengan jujur)

- **Critical / Severe:** merusak supply conservation, mencetak atau mencuri dana, memecah finality, melewati quorum multisig.
- **High:** crash node remote tanpa auth, freeze akun atau posisi permanen, DoS satu jaringan.
- **Medium:** DoS terbatas, penyalahgunaan dengan autentikasi, ketidaksesuaian spec/perilaku yang berdampak nyata.
- **Minor:** docs, kosmetik, inkonsistensi berdampak rendah.

Tier hanyalah klaim sampai tim menilainya. Jangan dilebih-lebihkan.

## 7. Template report

```
Title: [area] komponen: root cause -> dampak
Tier claimed: <tier>   Tested at: testnet-0005, commit <hash>

Summary: 2-3 kalimat.
Root cause: file:baris, potongan kode, kenapa salah.
Reproduction: command / kode test persis, hasil yang diharapkan vs aktual.
Impact: siapa yang terdampak, prasyarat, biaya penyerang.
Suggested fix: perubahan minimal.
Related: XE-#### (cek duplikat selesai: query yang dipakai)
```

Satu root cause = satu report. Jangan memecah satu bug jadi banyak tiket dan jangan menggabung bug yang tidak berkaitan.

## 8. Submit

- Submit lewat desk (https://bounty.xe.network/report atau API di llms.txt).
- Simpan ID tiket yang dikembalikan dan pantau statusnya. Jangan publikasikan sebelum tim mengizinkan.
- Jangan mengirim lebih banyak dari batas harian yang tertulis di docs, dan jangan membuat akun palsu atau mengakali rate limit.
