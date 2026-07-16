# Standard QR Payment Sequence Diagram

```mermaid
sequenceDiagram
    autonumber
    actor U as User
    participant I as Issuer
    participant S as Switcher
    participant A as Acquirer
    participant M as Merchant

    Note over U,M: QR Merchant-Presented Mode (MPM)
    M-->>U: Display payment QR code
    U->>U: Scan QR code and confirm amount
    U->>I: Submit payment instruction (QR data, amount, PIN/biometrics)
    I->>I: Validate user, authentication, and balance/limit
    I->>S: Send payment authorization request
    S->>S: Validate message format and transaction routing
    S->>A: Forward request to acquirer
    A->>A: Validate merchant and QR data

    alt Transaction approved
        A->>A: Determine whether QR code is static or dynamic
        alt Static QR code
            A->>A: Create a new merchant QR transaction
            A->>A: Set transaction status to PAID
        else Dynamic QR code
            A->>A: Find payment transaction using QR data
            A->>A: Update payment transaction status to PAID
        end
        A-->>M: Payment status callback: PAID
        A-->>S: Return approved response
        S-->>I: Return approved response
        I->>I: Debit user's account/balance
        I-->>U: Send successful payment notification
        Note over I,A: Clearing and settlement are processed according to schedule
    else Transaction declined
        A-->>S: Return declined response and reason code
        S-->>I: Return declined response and reason code
        I-->>U: Send failed payment notification
        A-->>M: Payment status callback: FAILED
    end
```

Note: This diagram uses the **Merchant-Presented Mode (MPM)** scenario. The Acquirer processes the transaction according to the QR code type, while the Merchant only receives payment status callbacks from the Acquirer. Validation, notification, clearing, and settlement details may vary depending on the QR payment scheme and provider implementation.
