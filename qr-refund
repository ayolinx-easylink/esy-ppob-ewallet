# Standard QR Payment Refund Sequence Diagram

```mermaid
sequenceDiagram
    autonumber
    actor U as User
    participant I as Issuer
    participant S as Switcher
    participant A as Acquirer
    participant M as Merchant

    Note over U,M: QR Payment Refund Flow
    U->>M: Request refund for the original QR payment
    M->>A: Submit refund request (original transaction ID, amount, reason)
    A->>A: Validate merchant and refund request
    A->>A: Verify original transaction is PAID and refundable
    A->>A: Check duplicate request and refundable balance

    alt Refund request is valid
        alt Full refund
            A->>A: Set requested amount to the remaining refundable balance
        else Partial refund
            A->>A: Validate requested amount against the refundable balance
        end
        A->>S: Send refund authorization request
        S->>S: Validate message format and route refund
        S->>I: Forward refund request
        I->>I: Validate original payment reference and destination account

        alt Refund approved by issuer
            I->>I: Credit user's account or balance
            I-->>S: Return approved refund response
            S-->>A: Return approved refund response
            A->>A: Update status to REFUNDED or PARTIALLY_REFUNDED
            A-->>M: Refund status callback: APPROVED
            I-->>U: Send successful refund notification
            Note over I,A: Refund clearing and settlement are processed according to scheme rules
        else Refund declined by issuer
            I-->>S: Return declined response and reason code
            S-->>A: Return declined response and reason code
            A->>A: Update refund status to FAILED
            A-->>M: Refund status callback: FAILED
            I-->>U: Send failed refund notification
        end
    else Refund request is invalid
        A->>A: Reject request and record reason code
        A-->>M: Refund status callback: REJECTED
        M-->>U: Inform user that the refund request was rejected
    end
```

Note: This is a provider-agnostic QR refund flow. The Acquirer validates the original payment and refund eligibility, the Issuer credits the User after approval, and the Merchant receives the refund status callback from the Acquirer. Exact authorization, clearing, settlement, timeout, and reversal rules may vary by QR payment scheme and provider implementation.
