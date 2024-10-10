# shudong_share_sql_injection
shudong share sql injection
## BUG_Author:
YiKai Wang

## Affected version:
latest(<=2.4.7)

## Vendor:
https://github.com/HFO4/shudong-share

## Software:
https://github.com/HFO4/shudong-share/archive/refs/heads/master.zip

## Vulnerability File:
includes/create_share.php
views/user_shares.php

## Description:
A critical second-order SQL Injection vulnerability has been discovered in the open-source project Shudong-Share.  This severe flaw allows a regular user to insert malicious SQL statements into the database through the ‘fkey’ parameter in ‘includes/create_share.php’.  Subsequently, in ‘views/user_shares.php’, the stored ‘fkey’ value is directly concatenated into SQL queries, leading to a second-order SQL injection, which could potentially result in unauthorized data access or manipulation.

## Vulnerability Verification:
Firstly, by accessing /includes/create_share.php ,use POST method,the following payload is provided, which utilizes a UNION-based SQL injection technique.
```ftype=open&fname=test&fkey=1' union select 1, (select version()), 'a'  ,'a', 'a', 'a', 'a', 'a' #```
![sql1](https://github.com/user-attachments/assets/494ecfc9-631c-4a73-bcaa-77d6fd42d138)
![sql1_res](https://github.com/user-attachments/assets/026c1a0d-d6c4-4039-9796-723ef2ad139a)

Next, by accessing /views/user_shares.php, the information obtained through the SQL injection can be seen in the original filename.
![sql2](https://github.com/user-attachments/assets/45eacded-6392-4a49-8c3f-9e4606864253)

These steps confirm the presence of a second-order SQL Injection vulnerability in the Shudong-Share project, as the payload executed successfully and sensitive data was extracted from the database.

## Code Audit:

Firstly, in the file includes/create_share.php, the script retrieves the POST parameters fkey, ftype, and fname.  When ftype is "open", a sharing link is created, and an entry is inserted into the sd_ss table.  At this point, the fkey parameter is not subjected to any filtering operations, which could be a potential security risk.

![create_share1](https://github.com/user-attachments/assets/f7fc0d13-aec9-4a21-869b-843f934822b7)
![create_share2](https://github.com/user-attachments/assets/87585dbf-db66-447b-8d1f-6fd01d37c4d1)


Next, in views/user_shares.php, the script initially checks if the user is a visitor.  This can be bypassed by simply registering a user on the frontend.  Subsequently, the script fetches all public sharing links associated with the user.  The fkey parameter, which we previously inserted, is used directly in a query to retrieve information from the sd_file table without proper sanitization.  This allows for an SQL injection attack, where an attacker can manipulate the fkey to alter the SQL query.

To exploit this vulnerability, an attacker can use a UNION injection to control the third column of the result set, which is the ming field.  By doing so, they can reflect and extract sensitive information from the database.

![userShares](https://github.com/user-attachments/assets/cde22e76-188d-47df-97fa-a66049db22a1)

