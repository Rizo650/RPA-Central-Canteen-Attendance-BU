# GA Central Canteen Attendance per BU - UiPath RPA

This RPA project automates the process of retrieving **canteen attendance data per Business Unit (BU)** and sending it via **WhatsApp**. It pulls the data directly from a **centralized database**, constructs a formatted message, and sends it using WhatsApp Web automation.

Built with **REFramework**, it supports error handling, logging, and notification via email for success or failure.

---

## Project Description

The automation fetches attendance data from a connected database and groups it by business unit (BU). A clean, readable message is constructed for each BU and delivered via WhatsApp using browser automation.

All components are modular and placed in the `Modular/` folder for maintainability and scalability.

---

## Key Features

- Connects to database to retrieve attendance records
- Groups attendance data per Business Unit (BU)
- Sends formatted attendance messages via **WhatsApp Web**
- Sends **email alerts** on success or failure
- Modular design with dedicated workflows for each function
- Structured error handling using **REFramework**

---

## Project Structure

| Folder/File                         | Description                                                  |
|-------------------------------------|--------------------------------------------------------------|
| Main.xaml                           | Main entry point for the automation                         |
| Modular/GetFromDatabase.xaml        | Retrieves attendance data from the database                 |
| Modular/ConstructMessage.xaml       | Formats attendance data into readable WhatsApp messages     |
| Modular/WhatsApp_OpenAndSendMsg.xaml | Sends message via WhatsApp Web                             |
| Modular/EmailSuccess.xaml           | Sends success email notification                            |
| Modular/EmailError.xaml             | Sends error notification email with log or context          |
| Framework/                          | REFramework core components                                 |
| Data/Config.xlsx                    | Configuration: DB connection, BU mappings, email settings   |
| project.json                        | Project metadata and dependencies                           |
| README.md                           | Project documentation                                        |

---

## Process Workflow (Detailed)

### 1. **Initialization**
- The REFramework starts by reading configurations from `Config.xlsx` and/or **Orchestrator Assets**.
- It sets up:
  - Database connection string
  - Email settings (SMTP/Outlook)
  - Logging path and output directory
- Logging is initialized with a timestamp and robot details.

---

### 2. **Retrieve Attendance Data**
- **Workflow:** `Modular/GetFromDatabase.xaml`
- Function:
  - Establishes connection to the target database
  - Executes a SQL query to retrieve today's attendance records
  - Returns a `DataTable` containing:
    - Employee Name
    - NIK (Employee ID)
    - Business Unit (BU)
    - Attendance timestamp

- **If DataTable is empty:**
  - A `BusinessRuleException` is thrown
  - The process stops
  - An error email is sent using `EmailError.xaml`

---

### 3. **Construct WhatsApp Message**
- **Workflow:** `Modular/ConstructMessage.xaml`
- Function:
  - Groups the retrieved data by Business Unit
  - For each BU, constructs a message like:

    ```
    GA Attendance - [BU Name]
    ----------------------------
    1. John Doe (NIK: 123456)
    2. Jane Smith (NIK: 789012)
    Total: 2 present
    ```

  - All messages are stored in a list or dictionary for sending later

---

### 4. **Send Message via WhatsApp**
- **Workflow:** `Modular/WhatsApp_OpenAndSendMsg.xaml`
- Function:
  - Launches WhatsApp Web in a browser (Chrome/Edge)
  - For each BU:
    - Searches for contact/group by name
    - Pastes and sends the formatted message using simulated typing/clicks
  - If a contact is not found:
    - The BU is added to the error list
    - The process continues with the next BU

---

### 5. **Success Email Notification**
- **Workflow:** `Modular/EmailSuccess.xaml`
- Function:
  - Sends a confirmation email to the GA stakeholder
  - Subject:  
    ```
    ✅ SUCCESS - Attendance per BU sent via WhatsApp
    ```
  - Body includes:
    - List of BUs successfully processed
    - Attendance count per BU
    - Timestamp of the process
    - Optional: attached log summary (e.g., .txt)

---

### 6. **Error Handling & Notification**
- If any step fails:
  - REFramework catches the `SystemException` or `BusinessException`
  - **Workflow:** `Modular/EmailError.xaml` is triggered
  - Function:
    - Sends a failure email with subject:
      ```
      ❌ FAILED - Attendance BU Automation Error
      ```
    - Email content includes:
      - Exception message
      - Last executed step
      - Screenshot (if available)
  - All errors are also logged locally or in Orchestrator (if connected)

---

## Optional Enhancements

- Export backup data to Excel (for archive use)
- Send a summary WhatsApp message to admin for monitoring
- Add shift-based validation (e.g., only process attendance between 11:00–13:00)
- Integrate with Microsoft Teams or Telegram as a WhatsApp alternative

---

## How to Run

> Recommended via **Orchestrator schedule** (daily, per shift, etc.)

### Manual Execution:

1. Open project in **UiPath Studio**
2. Update `Config.xlsx`:
   - Database connection string or secure asset
   - WhatsApp message format or BU grouping
3. Run `Main.xaml`
4. Monitor output via WhatsApp and email

---

## Exception Handling

- **Business Exceptions**:
  - No attendance records found
  - Invalid BU mapping

- **System Exceptions**:
  - Database connection error
  - WhatsApp Web timeout
  - UI element not found

All handled using:
- Retry logic (REFramework)
- Logging and screenshots
- Email alert via `EmailError.xaml`

---

## Requirements

- UiPath Studio 2022.10+
- WhatsApp Web access (via Chrome/Edge)
- Database access (SQL Server, MySQL, etc.)
- SMTP/Outlook for email notifications
- Configured Orchestrator (recommended)

---

## Contact

For questions, suggestions, or contributions:

- **Email:** fadillah650@gmail.com  
- **LinkedIn:** [Enrico Naufal Fadilla](https://linkedin.com/in/enrico-naufal-fadilla-54338a256)
