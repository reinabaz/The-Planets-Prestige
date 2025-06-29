# The Planet's Prestige Challenge - BlueTeamLab Walkthrough

## Overview

This walkthrough covers the analysis of a suspicious email in BlueTeamLab's "The Planet's Prestige Challenge." The scenario involves investigating a ransom email with attachments that contain hidden clues leading to the attacker's identity and location.

**Environment:** Ubuntu VM (recommended for safe analysis of suspicious files)

**Note:** This is a retired challenge.

## Initial Setup

1. Download the email file from the challenge
2. Extract using the provided password: `btlo`
3. Open the extracted file in a text editor (Sublime Text used in this analysis)

## Email Header Analysis

### Key Headers Identified

**Recipient Information:**
- `Delivered-To: themajoronearth@gmail.com` - Shows who received the email
  
![first one](https://github.com/user-attachments/assets/6f42438b-ed4f-4eae-83b2-1460335c4aed)

**Email Routing:**
- Multiple `Received:` headers trace the email's path through various SMTP servers
- The first received header represents the server closest to the recipient

![closest to the recepient](https://github.com/user-attachments/assets/cf6facb4-2472-411a-b2d1-a5e8068df3b4)

- The last received header shows the server closest to the sender

**Authentication and Security:**
- `ARC` headers (Authenticated Received Chain) provide email authentication chain

![arc headers](https://github.com/user-attachments/assets/354dbba7-da4e-4f7f-88cb-61bf2bcb26cd)

- `Return-Path: <billjobs@microapple.com>` - Address used for bounce-backs
- `Reply-To: negeja3921@pashter.com` - Address used when recipient clicks reply

### Suspicious Indicators

**SPF Failure:**
```
spf=fail (google.com: domain of billjobs@microapple.com does not designate 93.99.104.210 as permitted sender)
```
This indicates that microapple.com doesn't authorize IP address 93.99.104.210 to send emails on its behalf.

**Domain Mismatch:**
The Reply-To domain (`pashter.com`) differs from the From domain (`microapple.com`), which is a red flag for email spoofing.

**Email Service Identification:**
From the Received headers: `from localhost (emkei.cz [93.99.104.210])`, we can identify `emkei.cz` as the email service used by the attacker.

## Content Analysis

### Email Structure
- `Content-Type: multipart/mixed; boundary=BOUND_600FB98E0DCEE8.49207210`
- Contains multiple parts with different content types

### Base64 Encoded Message

The email body is base64 encoded

![First Base64 Encoding](https://github.com/user-attachments/assets/d1f5f971-5253-4890-917f-6c96168d2bb2)

After decoding using CyberChef:

![First Base64 Decoding](https://github.com/user-attachments/assets/d3d3ec61-13ca-461f-b508-ae24851994c9)

```
Hi TheMajorOnEarth,

The abducted CoCanDians are with me including the President's daughter. Dont worry. They are safe in a secret location.
Send me 1 Billion CoCanDs💰 in cash💸 with a spaceship🚀 and my autonomous bots will safely bring back your citizens.

I heard that CoCanDians have the best brains in the Universe. Solve the puzzle I sent as an attachment for the next steps.

I'm approximately 12.8 light minutes away from the sun and my advice for the puzzle is 

"Don't Trust Your Eyes"

Lol😄

See you Major. Waiting for the Cassshhhhh💰
```

## Attachment Analysis

### File Type Identification

The attachment claims to be `PuzzleToCoCanDa.pdf`, but we need to verify using file signatures:

![2nd content type](https://github.com/user-attachments/assets/725605af-dd46-4fb0-b695-8840aeed0468)

1. Decode the base64 attachment
2. Convert to hex to examine file signature (Also using CyberChef)
3. First couple of bytes: `50 4b 03 04`
4. Using Gary Kessler's File Signature Database, this identifies as a **ZIP file**

![claimed pdf real extension](https://github.com/user-attachments/assets/44d176f1-a8c7-47f1-8dbc-055b02a57bc8)

**Important:** Never trust file extensions alone - always verify using file signatures (magic numbers).

### ZIP File Contents

After extracting the ZIP file (remember to use a VM for safety), several files are discovered:
Make sure to check for hidden items (On windows: View → Check off hidden items, On Ubuntu: Ctrl+H)

To view a file in it’s hex form, I’m using ghex on ubuntu ( You can use HxD if you’re using windows)

![found files](https://github.com/user-attachments/assets/7f6c88a5-cd14-48c2-9ceb-2e55c6acbf6a)

(Also use Gary Kessler's File Signature Database to find the file type)

#### DaughtersCrown
- File signature: `FF D8 FF E0`

![Daughter's Crown File Type](https://github.com/user-attachments/assets/360b8c8b-f0da-46d5-a3c6-b834ccb1a4e8)

- File type: **JPEG image**

![DaughtersCrown File type](https://github.com/user-attachments/assets/88124405-4068-41c7-a313-8350381a7f91)

#### GoodJobMajor
- File signature: `25 50 44 46`

![GoodJobMajor pdf](https://github.com/user-attachments/assets/13c1c5a9-e99b-4199-ae8c-8f7cde1111d4)

- File type: **PDF document**

#### Money.xlsx
- File signature indicates Microsoft Office Open XML Format

![Money excel file](https://github.com/user-attachments/assets/09b5886b-ae79-418c-a5da-a6295b62f755)

- File type: **Excel spreadsheet**

### Hidden Content Discovery

**Excel File Analysis:**
1. The spreadsheet appears mostly empty in Sheet1 and Sheet3
2. To reveal hidden content: Select all → Right-click → Clear → Format
3. Sheet3 contains hidden base64 encoded text

![Money hiddent conent](https://github.com/user-attachments/assets/205abccd-ba8e-4c75-b339-477cd137cb0a)

5. Decoding reveals: `"The Martian Colony, Beside Interplanetary Spaceport."`

![Hidden content decoding](https://github.com/user-attachments/assets/f1b4484d-d349-48a2-8558-305d298dc00c)


### Metadata Analysis

Using `exiftool` to examine file metadata:

```bash
exiftool -r /path/to/extracted/files/*
```

Key finding: **Author: Pestero Negeja** - This reveals the attacker's full name.

## Challenge Questions & Answers

### 1. What is the email service used by the malicious actor?
**Answer:** `emkei.cz`
**Explanation:** Found in the Received headers showing the originating server.

### 2. What is the Reply-To email address?
**Answer:** `negeja3921@pashter.com`
**Explanation:** Listed in the Reply-To header field.

### 3. What is the filetype of the received attachment which helped to continue the investigation?
**Answer:** `.zip`
**Explanation:** File signature analysis revealed the true file type despite the .pdf extension.

### 4. What is the name of the malicious actor?
**Answer:** `Pestero Negeja`
**Explanation:** Discovered through metadata analysis using exiftool on the extracted files.

### 5. What is the location of the attacker in this Universe?
**Answer:** `The Martian Colony, Beside Interplanetary Spaceport.`
**Explanation:** Hidden base64 encoded message found in the Excel spreadsheet.

### 6. What could be the probable C&C domain to control the attacker's autonomous bots?
**Answer:** `pashter.com`
**Explanation:** Domain from the Reply-To address, likely used for command and control communications.

## Key Takeaways

- **File signature analysis** is crucial for identifying true file types
- **Email header analysis** reveals routing information and authentication failures
- **Hidden content** can be embedded in various file formats
- **Metadata analysis** often contains valuable investigative information
- **Always use isolated environments** when analyzing suspicious files

## Tools Used

- **CyberChef** - For encoding/decoding operations
- **exiftool** - For metadata analysis
- **ghex** (Ubuntu) / **HxD** (Windows) - For hex analysis
- **Gary Kessler's File Signature Database** - For file type identification

---

