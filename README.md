# 🎉 Automated Birthday Mail Sender - Documentation

This Power Automate cloud flow reads an Excel file containing employee data and sends a personalized birthday email with:

- Their name and nickname
- A custom message (randomized from 3 templates)
- A month-specific poster image

---

## 🧱 Table of Contents

1. [Requirements](#requirements)
2. [Excel File Structure](#excel-file-structure)
3. [Poster Image File Structure](#poster-image-structure)
4. [Variable Message File Structure](#variable-message-structure)
5. [OneDrive File Structure](#onedrive-structure)
6. [Step-by-Step: Creating the Flow](#step-by-step-creating-the-flow)
7. [Email Template with Randomized Message](#email-template-with-randomized-message)
8. [Editing & Customization](#editing--customization)

---

## ✅ Requirements

- Microsoft Power Automate
- Excel file hosted on OneDrive or SharePoint
- Poster images for each month
- Outlook (Office 365) for email sending
- Basic HTML/CSS knowledge for email design (optional)

---

## 📄 Excel File Structure

Create an Excel file with the following columns in a table:

| Name         | Nickname | Email              | Date of Birth | Birthday     |
|--------------|----------|--------------------|---------------|--------------|
| John Smith   | Johnny   | john@example.com   | 5/2/2025      |     05/02    |
| Fatima Rahim | Fati     | fatima@example.com | 4/26/2025     |     04/26    |

Ensure the data is inside a **formatted table**, e.g., named `Birthdays`.

---

## 🖼 Poster Image File Structure

You need to store 12 poster images in a folder, called "BDImage" in your OneDrive. Name them by month number (e.g., `01.png`, `02.png`, etc.).

---

## 📨 Variable Message File Structure

You need to store the messages which are to be used in the mail in a folder, called "BirthdayMessage", and create the messages by wrapping them in <p> HTML tags. Place the poster anywhere you want, preferably in the middle by enclosing it with:
```
<div class="center">
  <img src="cid:posterImage" alt="Birthday Poster" class="poster">
</div>
```
Name the .txt files with increasing numbers, starting from 1.


---

## 🗂 OneDrive File Structure

All the files related to this cloud flow will be stored in a folder called "AutomatedBirthdayMailSender". This is what the structure will look like:

```
OneDrive
└── AutomatedBirthdayMailSender
    ├── BirthdayList.xlsx
    └── BDImage
        ├── 01.png
        ├── 02.png
        ├── ...
        └── 12.png
    └── BirthdayMessage
        ├── 1.txt
        ├── 2.txt
        └── ...
```
---

## 🔧 Step-by-Step: Creating the Flow

### 🌀 0. Go to Microsoft Power Automate and click 'Create' on the sidebar

### 🌀 1. Start a **Scheduled Cloud Flow**

- Trigger: **Recurrence**
  - Frequency: `Day`
  - Interval: `1`
  - Time zone: Your preferred zone

### 📥 2. Read Excel Data

- Action: `List rows present in a table`
  - Location: Your Excel file
  - Table: `Birthdays`

### 🔁 3. Loop Over Rows

- Action: `Apply to each`
  - Input: Value from Excel table

Inside this loop:

### 🧮 4. Check for Today’s Birthday

- Action: `Condition`
  - Expression:
    ```plaintext
      items('Apply_to_each')?['Birthday'] == formatDateTime(utcNow(), 'MM/dd')
    ```

**If YES**, continue inside this branch:

---

### 🗓 5. Compose Month Number

- Action: `Compose` (Name: `MonthNumber`)
  - Expression:
    ```plaintext
    substring(items('Apply_to_each')?['Birthday'], 0, 2)
    ```
  - This will slice the Birthday column from the excel table and extract the month of their birthday.
 
### 📦 6. Concat Poster File Path

- Action: `Compose` (Name: `PosterFilePath`)
  - Expression:
    ```plaintext
    concat('/AutomatedBirthdayMailSender/BDImage/', outputs('MonthNumber'), '.png')
    ```
  - This will concatenate the output from the month number into a filepath for us to use to extract the poster later.

### 🎲 7. Random Message Selector

- Action: `Compose` (Name: `RandomSelector`)
  - Expression:
    ```plaintext
    rand(1, 4)
    ```
  - This will generate a number from 1 to 3, edit the final parameter based of how many message .txt files you have.
 
### 📦 8. Concat Message File Path

- Action: `Compose` (Name: `MessageFilePath`)
  - Expression:
    ```plaintext
    concat('/AutomatedBirthdayMailSender/BirthdayMessage/', outputs('RandomSelector'), '.txt')
    ```
  - This will concatenate the output from the random selector into a filepath for us to use to extract the message later.

 ### 🖼 9. Get Poster File Content (Using Path)

- Action: `Get file content using path` (Name: `Get Poster`)
  - Location: OneDrive for Business
  - File Path:
    ```plaintext
    @{outputs('PosterFilePath')}
    ```
  - This loads the randomized birthday message (.txt) content from the BirthdayMessage folder.

### 📂 10. Get Message File Content (Using Path)

- Action: `Get file content using path` (Name: `Get Message`)
  - Location: OneDrive for Business
  - File Path:
    ```plaintext
    @{outputs('MessageFilePath')}
    ```
  - This loads the image for the corresponding birthday month from the BDImage folder.

### 📧 11. Send Email

- Action: `Send an email (V2)`
  - To: `items('Apply_to_each')?['Email']`
  - Subject: `Happy Birthday, @{items('Apply_to_each')?['Nickname']}! 🎉`
  - Body: Use **HTML mode**. Here's an example:

```html
  <style>
    .container {
      font-family: Arial, sans-serif;
      padding: 20px;
      background-color: #f8f9fa;
      color: #333;
    }
    .card {
      max-width: 600px;
      margin: auto;
      background-color: #ffffff;
      border-radius: 10px;
      box-shadow: 0 4px 10px rgba(0, 0, 0, 0.1);
      padding: 30px;
    }
    .center {
      text-align: center;
    }
    .poster {
      max-width: 100%;
      height: auto;
      border-radius: 10px;
      margin-top: 20px;
      margin-bottom: 20px;
    }
    .footer {
      font-size: 12px;
      color: #777;
      text-align: center;
      margin-top: 40px;
    }
  </style>


  <div class="container">
    <div class="card">
      <div class="center">
        <h2>Happy Birthday, @{items('Apply_to_each')?['Nickname']}! 🎉</h2>
      </div>

      <p>Dear @{items('Apply_to_each')?['Full Name']},</p>


      

      @{body('Get_Message')}

      

      <p>Warm wishes,

      <strong>ICT Department</strong></p>
    </div>

    <div class="footer">
      This is an automated message sent to celebrate your special day with us 🎂
    </div>
  </div>
```
---
## ✉️ Email Template with Randomised Message

To make each birthday email feel more personal and less repetitive, the flow includes a **randomized HTML message block** loaded from `.txt` files in the `BirthdayMessage` folder.

Each message file should:
- Contain HTML-formatted text (e.g., wrapped in `<p>` tags)
- Include the poster image wrapped like this (recommended to place in the middle of the message):

```html
<div class="center">
  <img src="cid:posterImage" alt="Birthday Poster" class="poster">
</div>
```

The flow selects a message randomly using the following expression in a Compose action:
```
rand(1, 4)
```
---
## 🛠 Editing & Customization

### ✍️ To Edit Message Content:
- Go to OneDrive → `AutomatedBirthdayMailSender/BirthdayMessage`
- Open any `.txt` file.
- Edit the HTML content (you can add emojis, quotes, change tone, etc.).
- Save — no changes to the flow needed.

---

### ➕ To Add More Message Templates:
- Create a new `.txt` file (e.g., `4.txt`, `5.txt`, etc.) inside `BirthdayMessage`.
- Update the `rand(1, X)` formula in the **RandomSelector** step to match the new total count.

---

### 🖼 To Change Poster Images:
- Go to `AutomatedBirthdayMailSender/BDImage`.
- Replace images named `01.png` to `12.png` with new ones.
- Make sure filenames still match the month numbers.

---

### 💌 To Change Email Design:
- Open the **Send an email (V2)** action.
- Modify the HTML in the Body section.

**Customize:**
- Fonts
- Background colors
- Layout
- Branding (add logo or company footer)

---

### 🔒 Optional Enhancements:
- Add a condition to avoid sending on weekends.
- Log each sent email to a SharePoint list or Excel table.
- Include a fallback image if the monthly poster is missing.
