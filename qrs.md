# Diagram Sequence Flow QR Standar

```mermaid
sequenceDiagram
    autonumber
    actor U as User
    participant I as Issuer
    participant S as Switcher
    participant A as Acquirer
    participant M as Merchant

    Note over U,M: QR Merchant Presented Mode (MPM)
    M-->>U: Menampilkan QR pembayaran
    U->>U: Scan QR dan konfirmasi nominal
    U->>I: Instruksi pembayaran (QR data, nominal, PIN/biometrik)
    I->>I: Validasi user, autentikasi, dan saldo/limit
    I->>S: Request otorisasi pembayaran
    S->>S: Validasi format dan routing transaksi
    S->>A: Teruskan request ke acquirer
    A->>A: Validasi merchant dan data QR

    alt Transaksi disetujui
        A->>A: Cek tipe QR static atau dynamic
        alt QR static
            A->>A: Buat transaksi QR merchant baru
            A->>A: Set status transaksi menjadi PAID
        else QR dynamic
            A->>A: Cari transaksi payment berdasarkan data QR
            A->>A: Update status transaksi payment menjadi PAID
        end
        A-->>M: Callback status payment: PAID
        A-->>S: Response approved
        S-->>I: Response approved
        I->>I: Debit rekening/saldo user
        I-->>U: Notifikasi pembayaran berhasil
        Note over I,A: Clearing dan settlement dilakukan sesuai jadwal
    else Transaksi ditolak
        A-->>S: Response declined + reason code
        S-->>I: Response declined + reason code
        I-->>U: Notifikasi pembayaran gagal
        A-->>M: Callback status payment: FAILED
    end
```

Catatan: diagram menggunakan skenario **Merchant Presented Mode (MPM)**. Acquirer memproses transaksi berdasarkan tipe QR, sedangkan Merchant hanya menerima callback status payment dari Acquirer. Detail validasi, notifikasi, clearing, dan settlement dapat berbeda sesuai skema QR dan implementasi penyelenggara.
