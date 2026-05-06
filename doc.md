# Banking System Technical Report

## 1. Project Overview
This is a Java Swing desktop banking system application. It provides a simple GUI for admin login, account creation, deposit, withdrawal, account listing, and persistent storage via object serialization.

## 2. Technology Stack
- Java (standard edition)
- Java Swing for GUI
- Serialization for data persistence
- No build system configured; plain `javac` and `java` are expected

## 3. Project Structure
- `src/` — Java source code
  - `Application.java` — entry point
  - `Bank/` — domain models and business logic
  - `Exceptions/` — custom checked exceptions for validation
  - `Data/` — file persistence logic
  - `GUI/` — Swing-based UI windows
- `bin/` — compiled classes included in repository
- `data` — serialized data file used at runtime
- `README.md` — project description and launch instructions

## 4. Entry Point
- `src/Application.java`
  - Starts the app using `EventQueue.invokeLater`
  - Displays `GUIForm.login.frame`

## 5. Authentication
- `src/GUI/Login.java`
- Login is hardcoded:
  - Username: `admin`
  - Password: `admin`
- The UI even forces the username to `admin` before validating the password
- There is no real user management or secure authentication

## 6. Persistence
- `src/Data/FileIO.java`
- Uses Java object serialization to read/write a `Bank` object from/to the file named `data`
- Behavior:
  - On read failure, a new `Bank` is created
  - On exit from `Menu`, the current `FileIO.bank` object is serialized back to `data`
- Risks:
  - No file path configuration; uses relative path `data`
  - Exceptions are silently swallowed
  - No versioning or compatibility handling

## 7. Domain Model
### `Bank` (`src/Bank/Bank.java`)
- Stores up to 100 accounts in a fixed-size `BankAccount[]`
- Provides methods:
  - `addAccount(BankAccount acc)`
  - `addAccount(String, double, double)` for `SavingsAccount`
  - `addAccount(String, double, String)` for `CurrentAccount`
  - `addAccount(String, String, double, double)` for `StudentAccount`
  - `findAccount(String)`
  - `deposit(String, double)`
  - `withdraw(String, double)`
  - `display()` returns a `DefaultListModel<String>`

### `BankAccount` (`src/Bank/BankAccount.java`)
- Abstract base for accounts but is not declared `abstract`
- Fields:
  - `name`
  - `balance`
  - `min_balance`
  - `acc_num`
- Generates account number randomly from `10000` to `99999`
- Validates initial balance vs minimum balance in constructor
- `deposit` validates positive amount
- `withdraw` validates balance and minimum balance
- `toString` prints class type and basic account info

### `SavingsAccount` (`src/Bank/SavingsAccount.java`)
- Extends `BankAccount`
- Adds `rate = 0.05f`
- Adds `maxWithLimit`
- Overrides `withdraw()` to enforce `maxWithLimit`
- Adds `getNetBalance()` which calculates interest

### `CurrentAccount` (`src/Bank/CurrentAccount.java`)
- Extends `BankAccount`
- Enforces minimum balance of `5000`
- Stores `tradeLicenseNumber`

### `StudentAccount` (`src/Bank/StudentAccount.java`)
- Extends `SavingsAccount`
- Sets `min_balance = 100`
- Uses a fixed `maxWithLimit` of `20000`
- Stores `institutionName`
- Note: accesses `min_balance` from parent class though field is private in `BankAccount` — this compiles only if `min_balance` is package-private or protected. In source it is private, so this is a code issue.

## 8. GUI Architecture
### `GUIForm.java`
- Holds one instance of each window:
  - `Login`, `Menu`, `AddAccount`, `AddCurrentAccount`, `AddSavingsAccount`, `AddStudentAccount`, `DisplayList`, `DepositAcc`, `WithdrawAcc`
- Provides `UpdateDisplay()` to refresh the displayed account list window

### `Login.java`
- Login screen with username and password fields
- Hardcoded admin credentials
- Opens `GUIForm.menu` after successful login

### `Menu.java`
- Main dashboard after login
- Reads data on startup via `FileIO.Read()`
- Offers buttons for:
  - Add Account
  - Deposit
  - Withdraw
  - Display Account List
  - Exit
- On exit, writes data with `FileIO.Write()` and terminates

### `AddAccount.java`
- Launcher window for account type selection
- Opens one of the account creation windows

### `AddCurrentAccount.java`
- Adds a current account with fields:
  - Name
  - Balance
  - Trade License Number
- Requires minimum balance of `5000`
- Calls `FileIO.bank.addAccount(name, bal, trlic)`
- Adds new account string to `DisplayList.arr`

### `AddSavingsAccount.java`
- Adds a savings account with fields:
  - Name
  - Balance
  - Maximum withdraw limit
- Requires minimum balance of `2000`
- Calls `FileIO.bank.addAccount(name, bal, maxw)`
- Adds new account string to `DisplayList.arr`

### `AddStudentAccount.java`
- Adds a student account with fields:
  - Name
  - Balance
  - Institution Name
- Requires balance >= `100`
- Calls `FileIO.bank.addAccount(name, bal, insname)`
- Duplicate account add logic exists due to repeated call in confirm block

### `DisplayList.java`
- Shows accounts in a scrollable `JList`
- Refreshes content from `FileIO.bank.display()`

### `DepositAcc.java`
- Deposits to an account by number
- Handles exceptions:
  - `InvalidAmount`
  - `AccNotFound`

### `WithdrawAcc.java`
- Withdraws from an account by number
- Handles exceptions:
  - `MaxBalance`
  - `AccNotFound`
  - `MaxWithdraw`
  - `InvalidAmount`

## 9. Key Issues and Observations
### Security and business logic
- Login credentials are insecure and hardcoded
- Username field is ignored and forced to `admin`
- There is no user or session separation
- All account operations use a global static `FileIO.bank`

### Code quality and bugs
- `StudentAccount` sets `min_balance` though the parent field is private; this is inconsistent with proper OOP encapsulation
- `AddStudentAccount` adds the same account twice under confirm due to duplicate `FileIO.bank.addAccount(...)` call
- `AddSavingsAccount` shows wrong minimum limit warning text (`Minimum Limit 5000`) while requiring `balance < 2000`
- `DisplayList.arr` is static and reused across windows, creating possible stale UI state
- Silent exception handling in `FileIO.Read()` and `FileIO.Write()` hides errors
- `Bank.addAccount` uses a fixed array of 100 accounts with no bounds check before insertion
- `BankAccount.withdraw` condition `amount < balance` disallows withdrawing exactly the full balance, which may be unexpected

### Persistence and usability
- The app depends on a relative file named `data` in the working directory
- No separate `data` folder or path configuration is used
- The app writes data only on exit from `Menu` and not after every transaction

## 10. Compilation and Run Instructions
### Requirements
- Java JDK installed
- `javac` and `java` on PATH

### Compile
```powershell
javac -d bin src\Application.java src\Bank\*.java src\Exceptions\*.java src\Data\FileIO.java src\GUI\*.java
```

### Run
```powershell
java -cp bin Application
```

## 11. Recommendations
- Replace hardcoded login with a real authentication mechanism
- Avoid static global state (`FileIO.bank`) and use dependency injection or a controller
- Replace fixed-size `BankAccount[]` with `ArrayList<BankAccount>`
- Improve exception handling and surface errors to the user or logs
- Refactor account classes to use `abstract` and proper access modifiers
- Add validation for numeric parsing to prevent runtime exceptions
- Persist data immediately after transactions or add manual save/load controls
- Use a build tool like Maven or Gradle for dependency and compile management

## 12. Summary
This project is a basic Java Swing banking demo with:
- Single admin login
- Three account types
- Deposit/withdraw operations
- Account list display
- Serialized storage

The implementation is functional but fragile, with hardcoded credentials, poor error handling, UI state management issues, and several code quality concerns. Improvements should focus on security, architecture, persistence reliability, and clean separation between UI and business logic.
