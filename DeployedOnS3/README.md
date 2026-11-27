# 🌸 Kikō Japan – Static Website on Amazon S3

A multi-page travel website about Japan, hosted entirely on **Amazon S3 static website hosting**. There is no server to manage: the HTML and CSS files live in an S3 bucket and are served straight to the browser.

**Region:** Asia Pacific (Mumbai) – `ap-south-1`
**Bucket:** `japan-web-95`
**Live URL:** `http://<bucket-website-endpoint>` *(add your S3 website endpoint here)*

---

## 📄 Pages

| File | Description |
|------|-------------|
| `index.html` | Home page with hero banner and navigation cards |
| `gallery.html` | Photo gallery of scenes from the country |
| `video.html` | Video page |
| `culture.html` | Culture & Sound page |
| `about.html` | About the guide |
| `style.css` | Shared stylesheet for all pages |

---

## 🛠️ Tech Stack

- **HTML5** and **CSS3** for structure and styling
- **Amazon S3** for storage and static website hosting

---

## 🚀 Deployment Steps

1. **Create a bucket** in S3 (`japan-web-95`, region `ap-south-1`).
2. **Upload** all website files (`.html` and `.css`) to the bucket.
3. Go to **Properties → Static website hosting → Edit** and choose **Enable**.
4. Select **Host a static website** and set the **Index document** to `index.html`.
5. Go to **Permissions → Block public access** and turn it off for this bucket.
6. Add a **bucket policy** that allows public read access:

   ```json
   {
     "Version": "2012-10-17",
     "Statement": [
       {
         "Sid": "PublicReadGetObject",
         "Effect": "Allow",
         "Principal": "*",
         "Action": "s3:GetObject",
         "Resource": "arn:aws:s3:::japan-web-95/*"
       }
     ]
   }
   ```

7. Open the **website endpoint** (found at the bottom of the Static website hosting section) in your browser.

---

## 🖼️ Screenshots

| Screenshot | Preview |
|------------|---------|
| S3 bucket with website files | `![S3 Bucket](screenshots/s3-bucket.png)` |
| Static website hosting enabled | `![Hosting](screenshots/static-hosting.png)` |
| Home page | `![Home](screenshots/home.png)` |
| Gallery page | `![Gallery](screenshots/gallery.png)` |

---

## 💡 Key Learnings

- How S3 can host a complete website without any web server
- Setting the index document for static website hosting
- Why Block Public Access and a bucket policy matter for public content
- The difference between the S3 object URL and the website endpoint

---

## 🔮 Future Improvements

- Add a **CloudFront** distribution for HTTPS and faster loading
- Use a custom domain with **Route 53**
- Add an `error.html` page for broken links
- Make the layout fully responsive for mobile

---

## 👩‍💻 Author

**Rasika Auti**
