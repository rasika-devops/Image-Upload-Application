# 📤 Image Upload Application on AWS (EC2 + S3 + RDS)

A simple web application that lets a user submit their **name** and an **image**. The image is stored in an **Amazon S3 bucket**, and the user's name along with the image URL is saved in an **Amazon RDS (MySQL)** database. The web application is hosted on an **Amazon EC2** instance.

---

## 📌 Table of Contents

- [Project Overview](#-project-overview)
- [Architecture](#-architecture)
- [AWS Services Used](#-aws-services-used)
- [How It Works](#-how-it-works)
- [Prerequisites](#-prerequisites)
- [Setup Guide](#-setup-guide)
- [Database Schema](#-database-schema)
- [Usage](#-usage)
- [Verification](#-verification)
- [Security Best Practices](#-security-best-practices)
- [Future Improvements](#-future-improvements)

---

## 📖 Project Overview

This project demonstrates how three core AWS services work together in a real-world, three-tier style application:

| Layer | Service | Purpose |
|-------|---------|---------|
| Compute / Web | **Amazon EC2** | Hosts the HTML form and PHP backend |
| Storage | **Amazon S3** | Stores the uploaded image files |
| Database | **Amazon RDS (MySQL)** | Stores the user name and S3 image URL |

---

## 🏗️ Architecture

```
                 ┌──────────────┐
                 │     User     │
                 │   (Browser)  │
                 └──────┬───────┘
                        │  HTTP (port 80)
                        ▼
              ┌───────────────────┐
              │   Amazon EC2      │
              │ (Ubuntu + Apache  │
              │   + PHP)          │
              │ indexform.html    │
              │ upload.php        │
              └─────┬───────┬─────┘
                    │       │
      Upload image  │       │  Insert name + image URL
                    ▼       ▼   (port 3306)
          ┌────────────┐   ┌────────────────────┐
          │ Amazon S3  │   │ Amazon RDS (MySQL) │
          │  Bucket    │   │  database: mydb    │
          │ 24sept-    │   │  table: posts      │
          │ bucket     │   └────────────────────┘
          └────────────┘
```

**Region:** Asia Pacific (Mumbai) – `ap-south-1`

---

## ☁️ AWS Services Used

- **Amazon EC2** – Ubuntu instance running Apache and PHP
- **Amazon S3** – General purpose bucket (`24sept-bucket`) for image storage
- **Amazon RDS** – MySQL Community instance (`database-1`, class `db.t4g.micro`)
- **Amazon VPC / Security Groups** – Control network access between EC2, RDS and the internet
- **AWS IAM** – Permissions for EC2 to write to S3

---

## 🔄 How It Works

1. The user opens `http://<EC2_PUBLIC_IP>/indexform.html` in a browser.
2. The user enters their **Name** and selects an **Image**, then clicks **Upload Image**.
3. The form submits to `upload.php` on the EC2 instance.
4. `upload.php` uploads the image to the **S3 bucket**.
5. The S3 object URL is generated, e.g. `https://24sept-bucket.s3.ap-south-1.amazonaws.com/<file-name>`.
6. `upload.php` connects to **RDS MySQL** and inserts a new record (`name`, `url`) into the `posts` table.
7. A success message is displayed to the user.

---

## ✅ Prerequisites

- An AWS account
- Basic knowledge of EC2, S3 and RDS
- An EC2 key pair (`.pem`) to SSH into the instance
- Installed on EC2: **Apache**, **PHP**, **PHP MySQL extension**, **AWS SDK for PHP** (or AWS CLI), **MySQL/MariaDB client**

---

## 🛠️ Setup Guide

### 1️⃣ Create the S3 Bucket

1. Go to **S3 → Create bucket**.
2. Bucket name: `24sept-bucket` (must be globally unique).
3. Region: `ap-south-1` (Mumbai).
4. Configure public access / bucket policy as required for your use case (see [Security Best Practices](#-security-best-practices)).

### 2️⃣ Create the RDS MySQL Database

1. Go to **RDS → Create database**.
2. Engine: **MySQL Community**.
3. DB instance identifier: `database-1`
4. Instance class: `db.t4g.micro`
5. Master username: `admin` and set a strong password.
6. Port: `3306`
7. Public access: **Disabled** (only EC2 should reach the DB).
8. Attach a security group that allows **inbound MySQL/Aurora (3306)** from the EC2 security group.

### 3️⃣ Launch the EC2 Instance

1. Go to **EC2 → Launch instance**.
2. AMI: **Ubuntu**.
3. Allow inbound **SSH (22)** and **HTTP (80)** in the security group.
4. Attach an **IAM role** with permission to upload objects to the S3 bucket (e.g. `s3:PutObject` on `24sept-bucket/*`).

### 4️⃣ Install Required Packages on EC2

```bash
sudo apt update
sudo apt install -y apache2 php libapache2-mod-php php-mysql php-curl mysql-client unzip
sudo systemctl enable --now apache2
```

### 5️⃣ Deploy the Application Files

Place your files in the Apache web root:

```bash
cd /var/www/html
# indexform.html  -> upload form
# upload.php      -> backend (S3 upload + RDS insert)
```

### 6️⃣ Create the Database and Table

Download the RDS SSL certificate bundle and connect from EC2:

```bash
sudo mysql -u admin -p \
  -h <RDS_ENDPOINT> \
  --ssl-ca=global-bundle.pem
```

Then run:

```sql
CREATE DATABASE IF NOT EXISTS mydb;
USE mydb;

CREATE TABLE posts (
  id   INT AUTO_INCREMENT PRIMARY KEY,
  name VARCHAR(100) NOT NULL,
  url  VARCHAR(500) NOT NULL
);
```

### 7️⃣ Configure the Application

Update the configuration inside `upload.php` with your own values:

```php
$bucket   = "24sept-bucket";
$region   = "ap-south-1";

$db_host  = "<RDS_ENDPOINT>";
$db_user  = "admin";
$db_pass  = "<YOUR_DB_PASSWORD>";   // do NOT commit real passwords
$db_name  = "mydb";
```

---

## 🗄️ Database Schema

**Database:** `mydb`  **Table:** `posts`

| Column | Type | Description |
|--------|------|-------------|
| `id`   | INT (PK, AUTO_INCREMENT) | Unique record ID |
| `name` | VARCHAR(100) | Name entered by the user |
| `url`  | VARCHAR(500) | S3 URL of the uploaded image |

---

## 🚀 Usage

1. Open your browser and visit:
   ```
   http://<EC2_PUBLIC_IP>/indexform.html
   ```
2. Enter your **Name**.
3. Click **Choose File** and select an image (`.jpg`, `.jpeg`, `.png`).
4. Click **Upload Image**.
5. You will see a confirmation such as:
   ```
   Image uploaded successfully. Image path is: https://24sept-bucket.s3.ap-south-1.amazonaws.com/<file-name>
   New record created successfully
   Your file was uploaded successfully.
   ```

---

## 🔍 Verification

**1. Check the S3 bucket** – Open `24sept-bucket` in the S3 console and confirm the uploaded image appears under **Objects**.

**2. Check the RDS database** – From the EC2 instance:

```sql
USE mydb;
SELECT * FROM posts;
```

Example output:

```
+----+----------+---------------------------------------------------------------+
| id | name     | url                                                           |
+----+----------+---------------------------------------------------------------+
|  1 | Ninushka | https://24sept-bucket.s3.ap-south-1.amazonaws.com/image.jpg.jpeg |
|  2 | Rasika   | https://24sept-bucket.s3.ap-south-1.amazonaws.com/Teacher%27s%20Day%20invitation.png |
+----+----------+---------------------------------------------------------------+
```

---

## 🔐 Security Best Practices

- **Never hard-code or commit** DB passwords or AWS keys. Use environment variables, **AWS Secrets Manager**, or an **IAM role** attached to EC2.
- Keep RDS **not publicly accessible**; allow port `3306` only from the EC2 security group.
- Restrict SSH (port 22) to **your IP** only.
- Grant the EC2 IAM role **least-privilege** access (only the required S3 actions on the specific bucket).
- Validate uploaded files (allowed extensions, MIME type, max size) and sanitize file names.
- Use **prepared statements** in PHP to prevent SQL injection.
- Enable **HTTPS** (e.g. with a load balancer + ACM certificate) for production use.
- Connect to RDS using **SSL** (`--ssl-ca=global-bundle.pem`
## 🔮 Future Improvements

- Display uploaded images in a gallery page
- Add file type and size validation with user-friendly errors
- Use unique file names (timestamp/UUID) to avoid overwriting objects
- Serve images through **CloudFront**
- Use **pre-signed URLs** instead of public objects
- Move credentials to **AWS Secrets Manager**
- Add an Application Load Balancer and Auto Scaling for high availability

---



**Rasika Auti**

Built as a hands-on AWS project to learn how **EC2**, **S3** and **RDS** work together.

---

