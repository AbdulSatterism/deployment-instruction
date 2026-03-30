# 🌐 সহজ হোস্টিং গাইড (React + Node) — একদম নতুন VPS থেকে শুরু

## ✅ ধাপ ০: আগে ডোমেইন সেট করে নিই (Hostinger)

১. Hostinger এ যান 👉 [https://hpanel.hostinger.com](https://hpanel.hostinger.com)  
২. ডোমেইন লিস্টে যান → **DNS Zone** এ ক্লিক করুন  
৩. এখন খেয়াল করুন:  
 🔍 যদি **@, api বা dashboard** নামে পুরনো কোনো A রেকর্ড থাকে —  
 👉 **সেগুলো আগে ডিলিট করে দিন।**

> ⚠️ একই নামে একাধিক A রেকর্ড থাকলে DNS কাজ করবে না।

৪. এবার নিচের মতো নতুন তিনটা A রেকর্ড যোগ করুন (আপনার VPS IP বসিয়ে):

| নাম         | টাইপ | IP দিবেন |
| ----------- | ---- | -------- |
| `@`         | A    | VPS_IP   |
| `api`       | A    | VPS_IP   |
| `dashboard` | A    | VPS_IP   |

এগুলো দিয়ে আপনার:

-   `yourdomain.com`
-   `api.yourdomain.com`
-   `dashboard.yourdomain.com`

সবগুলো সঠিকভাবে আপনার VPS-এ পয়েন্ট করবে ✅

---

## ✅ ধাপ ১: VS Code দিয়ে VPS এ ঢুকি

১. VS Code ওপেন করুন  
২. এক্সটেনশন অপশনে যান (`Ctrl + Shift + X`)  
৩. লিখুন **Remote - SSH**, তারপর **Install** করুন  
৪. এবার `Ctrl + Shift + P` চাপুন → লিখুন:

```
Remote-SSH: Connect to Host
```

৫. যখন অ্যাড করতে বলে, তখন লিখুন:

```
ssh root@YOUR_VPS_IP
```

৬. Enter দিন, তারপর কিছুক্ষণের মধ্যে VPS এ ঢুকে যাবেন 🎉

---

## ✅ ধাপ ২: VPS এ দরকারি জিনিস ইনস্টল করে নেই

টার্মিনাল খুলে নিচের কমান্ডটা একবারেই কপি করে চালান:

```bash
apt update && apt upgrade -y
apt install -y nginx curl unzip git tmux nodejs npm certbot python3-certbot-nginx
systemctl enable nginx
systemctl start nginx
```

👉 এতে সার্ভারের সব দরকারি প্যাকেজ ইনস্টল হবে  
👉 NGINX চালু হয়ে থাকবে সব সময়

---

## ✅ ধাপ ৩: নিজের প্রজেক্ট গুলো আপলোড করি (Drag & Drop)

VS Code এ গিয়ে `/var/www/` ফোল্ডার ওপেন করুন।

এখন আপনার লোকাল পিসি থেকে নিচের ফোল্ডারগুলো ধরে ধরে এখানে ড্র্যাগ করে ছেড়ে দিন:

| প্রজেক্ট            | VPS-এ কোথায় দিবেন    |
| ------------------- | -------------------- |
| ফ্রন্টএন্ড (React)  | `/var/www/frontend`  |
| ব্যাকএন্ড (Node.js) | `/var/www/backend`   |
| ড্যাশবোর্ড (React)  | `/var/www/dashboard` |

তারপর প্রতিটা ফোল্ডারে ঢুকে টার্মিনাল খুলে রান করুন:

```bash
npm install
```

---

## ✅ ধাপ ৪: প্রজেক্টগুলো চালু করে দেই (tmux দিয়ে)

টার্মিনালে নিচের কমান্ডগুলো একে একে চালান:

```bash
tmux new -s frontend
cd /var/www/frontend
npm start
# Ctrl + B, তারপর D চাপুন (detach)

tmux new -s backend
cd /var/www/backend
npm start
# Ctrl + B, তারপর D

tmux new -s dashboard
cd /var/www/dashboard
npm start
# Ctrl + B, তারপর D
```

👉 এতে আপনি টার্মিনাল বন্ধ করলেও সার্ভার চালু থাকবে।

---

## ✅ ধাপ ৫: এখন NGINX কনফিগার করি

### 📁 `/etc/nginx/sites-available/` ফোল্ডারে যান (VS Code দিয়ে)

---

### ⚠️ একটা জিনিস খুবই গুরুত্বপূর্ণ

🔥 **`default` নামের যে ফাইলটা আছে, সেটা ডিলিট করে দিন।**  
🛑 **কিছুতেই অন্য কোনো ফাইল ডিলিট করবেন না।**

**কেন?**  
এই ফাইলটা থাকলে, আপনার সাবডোমেইন কাজ না করে nginx-এর default পেইজ দেখাবে।

---

### 🛠 এরপর ৩টা ফাইল বানান:

-   `root`
-   `api`
-   `dashboard`

প্রতিটাতে নিচের মতো কনফিগ লিখে ফেলুন (সাবডোমেইন ও port নিজের মতো করে বসান):

```nginx
server {
    listen 80;
    server_name (subdomain.)yourdomain.com;

    location / {
        proxy_pass http://localhost:(port);
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

টার্মিনালে রান করুন:

```bash
sudo ln -s /etc/nginx/sites-available/root /etc/nginx/sites-enabled/
sudo ln -s /etc/nginx/sites-available/api /etc/nginx/sites-enabled/
sudo ln -s /etc/nginx/sites-available/dashboard /etc/nginx/sites-enabled/
```

---

### 🔁 এবার NGINX রিলোড দিন

টার্মিনালে রান করুন:

```bash
nginx -t
systemctl reload nginx
```

---

## ✅ ধাপ ৬: SSL বসাই (HTTPS ফ্রি সার্টিফিকেট)

টার্মিনালে নিচেরটা চালান:

```bash
certbot --nginx -d yourdomain.com -d api.yourdomain.com -d dashboard.yourdomain.com
```

👉 যেটা জিজ্ঞেস করে, সেখানে **redirect HTTP to HTTPS** অপশনটা সিলেক্ট করে দিন।

---

## 🎉 সব হয়ে গেছে!

| URL                                | কী দেখাবে |
| ---------------------------------- | --------- |
| `https://yourdomain.com`           | Main app  |
| `https://api.yourdomain.com`       | Backend   |
| `https://dashboard.yourdomain.com` | Dashboard |

---

## ⚙️ দরকারি কমান্ড

### 🧵 TMUX

```bash
tmux new -s name        # নতুন সেশন
tmux ls                 # সব সেশন দেখতে
tmux attach -t name     # পুরানো সেশনে ঢুকতে
tmux kill-session -t name # সেশন বন্ধ করতে
```

### 🌐 NGINX

```bash
nginx -t                 # config ঠিক আছে কি না দেখতে
systemctl reload nginx  # NGINX রিলোড দিতে
```

### 🔐 CERTBOT (SSL)

```bash
certbot renew --dry-run   # অটো রিনিউ ঠিকঠাক হচ্ছে কি না দেখতে


✅ Step 2: Install Necessary Software in VPS
Copy and run:

apt update && apt upgrade -y
apt install -y nginx curl unzip git certbot python3-certbot-nginx

curl -fsSL https://deb.nodesource.com/setup_24.x | bash -
apt install -y nodejs
npm i -g pm2

systemctl enable nginx
systemctl start nginx
👉 This installs all required packages & starts NGINX.

✅ Step 3: Upload Your Projects (Drag & Drop)
In VS Code, open /var/www/ folder.

Drag your project folders here:

Project	Upload To
Frontend (React)	/var/www/frontend
Backend (Node.js)	/var/www/backend
Dashboard (React)	/var/www/dashboard
Then go inside each folder and run:

npm install
✅ Step 4: Run Projects Using PM2
cd /var/www/frontend
pm2 start npm --name frontend -- start

cd /var/www/backend
pm2 start npm --name backend -- start

cd /var/www/dashboard
pm2 start npm --name dashboard -- start
Save and enable autostart:

pm2 save
pm2 startup
```
