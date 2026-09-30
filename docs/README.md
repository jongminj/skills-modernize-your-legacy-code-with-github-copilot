# COBOL Student Account Program

This directory documents the COBOL account-management example in `src/cobol/`.
The current implementation manages one generic account; it does not store
student names, IDs, or multiple student accounts.

## Source Files

- [`main.cob`](../src/cobol/main.cob) contains the interactive entry point. Its
  `MAIN-LOGIC` paragraph displays a menu, reads a choice, and calls
  `Operations` with `TOTAL`, `CREDIT`, or `DEBIT`. Choosing 4 exits the loop;
  other choices display an error message.
- [`operations.cob`](../src/cobol/operations.cob) implements the account
  actions. It reads and displays the balance for `TOTAL`, prompts for an amount
  and updates the balance for `CREDIT`, and prompts for an amount and applies
  `DEBIT` when funds are sufficient. It communicates with `DataProgram` to
  read and write the balance.
- [`data.cob`](../src/cobol/data.cob) owns the balance in working storage. Its
  procedure accepts an operation and balance: `READ` copies the stored balance
  to the caller, and `WRITE` replaces the stored balance.

## Student Account Rules

- The balance starts at `1000.00` when `DataProgram` initializes.
- The balance and transaction amount use `PIC 9(6)V99`, allowing six whole
  digits and two implied decimal places (up to `999999.99`).
- A debit is applied only when the current balance is greater than or equal to
  the requested amount. Otherwise, the program displays an insufficient-funds
  message and leaves the balance unchanged.
- A credit adds the entered amount to the balance. There is no explicit check
  for positive amounts or for exceeding the field's maximum value. Debit input
  likewise has no explicit positivity validation.
- Only one account is represented. The balance is held in program working
  storage, not in a file or database, so no durable student account records
  are provided.

## Application Data Flow

```mermaid
sequenceDiagram
  actor User
  participant Main as MainProgram
  participant Ops as Operations
  participant Data as DataProgram

  Note over Data: Balance is initialized to 1000.00 in working storage

  loop Until the user chooses 4
    Main->>User: Display account menu
    User->>Main: Enter menu choice
    alt Choice 1: View balance
      Main->>Ops: CALL TOTAL
      Ops->>Data: CALL READ
      Data-->>Ops: Return current balance
      Ops->>User: Display current balance
    else Choice 2: Credit account
      Main->>Ops: CALL CREDIT
      Ops->>User: Prompt for credit amount
      User->>Ops: Enter amount
      Ops->>Data: CALL READ
      Data-->>Ops: Return current balance
      Ops->>Ops: Add amount to balance
      Ops->>Data: CALL WRITE with updated balance
      Data-->>Ops: Store updated balance
      Ops->>User: Display confirmation and new balance
    else Choice 3: Debit account
      Main->>Ops: CALL DEBIT
      Ops->>User: Prompt for debit amount
      User->>Ops: Enter amount
      Ops->>Data: CALL READ
      Data-->>Ops: Return current balance
      alt Balance is sufficient
        Ops->>Ops: Subtract amount from balance
        Ops->>Data: CALL WRITE with updated balance
        Data-->>Ops: Store updated balance
        Ops->>User: Display confirmation and new balance
      else Insufficient funds
        Ops->>User: Display insufficient-funds message
      end
    else Choice 4: Exit
      Main->>Main: Set continue flag to NO
      Main->>User: Display goodbye message
    else Invalid choice
      Main->>User: Display invalid-choice message
    end
  end
```
