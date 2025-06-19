# MarketPeak_Ecommerce Deployment Log

This README documents the journey I took to deploy the **MarketPeak_Ecommerce** project from local development to live deployment on AWS EC2, including the bumps I hit along the way—and how I solved them.

---

## 🛠️ 1. Setting Up Git & Initial Project

### 1.1 Initialized Git Repository
I started by creating a new project folder named `MarketPeak_Ecommerce` and initialized a Git repository in it:

```bash
mkdir MarketPeak_Ecommerce
cd MarketPeak_Ecommerce
git init
```
---

1.2 Obtained and Prepared the E-Commerce Template
The recommended template was "2130 Waso Strategy" from tooplate.com. I downloaded and extracted the files into my project directory.

1.3 Pushed Project to GitHub
I staged and committed the files, then created a GitHub repo with the same name. I linked it and pushed the contents:

```Bash
git remote add origin https://github.com/yJerriemiah/MarketPeak_Ecommerce.git
git add .
git commit -m "Initial commit with template"
git push -u origin main
```
---

☁️ 2. AWS EC2 Deployment
2.1 Launched EC2 Instance
I launched an Amazon Linux EC2 instance through the AWS Console.

2.2 Connected to EC2 Using PuTTY
I used PuTTY to SSH into the instance.

2.3 Updated & Prepared EC2 for Deployment
After connecting, I updated the machine:
```
sudo yum update -y
```

I generated an SSH key, added it to my GitHub SSH settings, and cloned the repo:
```
ssh-keygen
cat ~/.ssh/id_rsa.pub  # copied this to GitHub
git clone git@github.com:Jerriemiah/MarketPeak_Ecommerce.git
```
Then I installed Apache:
```
sudo yum install httpd -y
sudo systemctl start httpd
sudo systemctl enable httpd

```

2.4 Configured Apache to Serve My Website
To serve the website through Apache, I cleared the default web root and copied my site files into it:
```
sudo rm -rf /var/www/html/*
sudo cp -r ~/MarketPeak_Ecommerce/* /var/www/html/
```

🐛 Problem: "It works" page kept showing
Initially, the default Apache welcome page kept showing, even after copying my site files. I checked the contents of /var/www/html/ and found the issue: the template folder 2130_waso_strategy or MarketPeak_Ecommerce was still nested inside. Apache only serves content directly inside /var/www/html.

To fix it:
```
sudo mv /var/www/html/MarketPeak_Ecommerce/* /var/www/html/
sudo rm -rf /var/www/html/MarketPeak_Ecommerce
sudo systemctl restart httpd
```
After this, my site rendered properly in the browser via the instance’s public IP!

---

🔁 3. Continuous Integration & Deployment Workflow
3.1 Created a Development Branch
I wanted to simulate a basic CI/CD setup, so I created a development branch and made some changes:
```
git branch development
git checkout development
# edited the project title via VSCode
git add .
git commit -m "Add new features or fix bugs"
git push origin development
```
3.2 Pull Request and Merge
I opened a Pull Request on GitHub, reviewed the changes, then merged the development branch back into main:
```
git checkout main
git merge development
git push origin main
```
3.3 Pulled Changes into the EC2 Webserver
Back on the EC2 server, I pulled the new changes. But I got stuck for a bit—turns out, running git pull in /var/www/html/ failed with fatal: not a git repository. That made sense, because the original repo was cloned into /home/ec2-user.

I navigated back there and pulled:
```
cd ~/MarketPeak_Ecommerce
git pull origin main
```
After refreshing the browser—voila! My new changes showed up instantly.

---

💡 Lessons & Highlights
Apache serves content directly from /var/www/html, so nesting folders will block your site.

* in cp -r source/* destination/ means “only copy the contents,” not the directory itself.

You can pull from GitHub and redeploy quickly just by copying files and restarting Apache.

Troubleshooting is half the job—and good logging/sanity checks (ls -lah) help a ton.

---

🚀 Final Thoughts
This deployment taught me more than just uploading a site—it gave me hands-on experience with EC2, Linux, Git, basic CI/CD practices, and real-world debugging. 
Shoutout to Copilot (AI assistant) for guiding me through each bump!
