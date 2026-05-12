# Easylink PPOB Purchase Flow using Pivot Open Loop Wallet

## Overview

This document describes the asynchronous PPOB purchase flow using:

- Pivot Open Loop Wallet
- Easylink as Merchant
- PPOB Provider as Product Provider

In this architecture:

- User balance is maintained by Pivot
- Easylink does NOT maintain user wallet balance
- All payment deductions happen inside Pivot
- Easylink only processes orders after receiving payment callbacks

---

# Architecture Responsibility

| Component | Responsibility |
|---|---|
| Pivot | User wallet balance, PIN validation, payment processing |
| Easylink | Order management, PPOB orchestration |
| PPOB Provider | Product fulfillment |
| User | Initiates payment |

---

# PPOB Purchase Flow

```mermaid
sequenceDiagram
    autonumber

    participant U as User
    participant A as App Easylink
    participant W as Backend Easylink
    participant P as Pivot Wallet
    participant PPOB as Provider PPOB

    %% Create Order
    U->>A: Select PPOB Product
    A->>W: Purchase Request

    W->>W: Create Order (PENDING)

    %% Create Payment
    W->>P: Create Payment Merchant / Payment Link
    P-->>W: Return paymentLink

    W-->>A: Send paymentLink
    A-->>U: Redirect User to Pivot

    %% User Payment
    U->>P: Open paymentLink

    P->>P: Validate User Balance
    P->>P: User Input PIN
    P->>P: Deduct User Balance

    %% Async Callback
    P-->>W: Callback PAYMENT.SUCCESS

    W->>W: Validate Callback
    W->>W: Update Order = PAID

    %% Trigger PPOB
    W->>PPOB: Purchase Product
    PPOB-->>W: Success / Failed Response

    alt PPOB Success
        W->>W: Update Order = SUCCESS
        W-->>A: Success Notification
        A-->>U: Product Delivered
    else PPOB Failed
        W->>W: Update Order = FAILED

        %% Refund Process
        W->>P: Request Refund / Reversal (Account Transfer)
        P-->>W: Callback REFUND.SUCCESS

        W->>W: Update Order = REFUNDED

        W-->>A: Failed + Refund Notification
        A-->>U: Balance Refunded
    end
