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
    activate I
    I->>I: Validasi user, autentikasi, dan saldo/limit
    I->>S: Request otorisasi pembayaran
    activate S
    S->>S: Validasi format dan routing transaksi
    S->>A: Teruskan request ke acquirer
    activate A
    A->>M: Validasi merchant dan transaksi
    activate M
    M-->>A: Hasil validasi merchant
    deactivate M

    alt Transaksi disetujui
        A-->>S: Response approved
        deactivate A
        S-->>I: Response approved
        deactivate S
        I->>I: Debit rekening/saldo user
        I-->>U: Notifikasi pembayaran berhasil
        A-->>M: Notifikasi pembayaran diterima
        deactivate I
        Note over I,A: Clearing dan settlement dilakukan sesuai jadwal
    else Transaksi ditolak
        A-->>S: Response declined + reason code
        deactivate A
        S-->>I: Response declined + reason code
        deactivate S
        I-->>U: Notifikasi pembayaran gagal
        A-->>M: Status transaksi gagal
        deactivate I
    end
```

Catatan: diagram menggunakan skenario **Merchant Presented Mode (MPM)**. Detail validasi, notifikasi, clearing, dan settlement dapat berbeda sesuai skema QR dan implementasi penyelenggara.
