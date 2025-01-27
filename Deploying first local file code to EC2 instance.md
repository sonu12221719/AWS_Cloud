# Deploying a Frontend Project to AWS EC2

This guide provides a step-by-step process to deploy a local frontend project (HTML, CSS, JavaScript, or React) to an AWS EC2 instance.

---

## Prerequisites
- An **AWS EC2 instance** (Ubuntu) with a **public IP**.
- A **frontend project** on your local machine.
- **SSH key (.pem file)** downloaded when creating the EC2 instance.
- **Apache or Nginx installed** on EC2 (for serving static files).
- **Basic knowledge of the Linux terminal**.

---

## Step 1: Open Terminal and Navigate to Your Project
On your **local machine**, open a terminal or PowerShell and navigate to your project directory:
```bash
cd "C:\Users\Sonu Kumar\OneDrive\Desktop\ReactJs\vmFrontend"
```

---

## Step 2: Connect to Your EC2 Instance
Use SSH to connect to your AWS EC2 instance. Replace `your-key.pem` with your key file and `your-ec2-ip` with your instance's public IP.
```bash
ssh -i "C:/path/to/your-key.pem" ubuntu@your-ec2-ip
```

If you see a **permission denied** error, set proper permissions:
```bash
chmod 400 "C:/path/to/your-key.pem"
```
Then try connecting again.

---

## Step 3: Install Apache or Nginx on EC2 (If Not Installed)

For Apache:
```bash
sudo apt update
sudo apt install apache2 -y
sudo systemctl start apache2
sudo systemctl enable apache2
```
For Nginx:
```bash
sudo apt update
sudo apt install nginx -y
sudo systemctl start nginx
sudo systemctl enable nginx
```

Check if your server is running by visiting `http://your-ec2-ip` in a browser.

---

## Step 4: Upload Your Local Project to EC2
Use `scp` (Secure Copy Protocol) to transfer files. Run this from **your local machine**:
```bash
scp -i "C:/path/to/your-key.pem" -r "C:/path/to/your-frontend-project" ubuntu@your-ec2-ip:/var/www/html
```

If you get a **permission denied** error, log into your EC2 instance and change ownership:
```bash
sudo chown -R ubuntu:ubuntu /var/www/html
```
Then re-run the `scp` command from your local machine.

---

## Step 5: Verify Files on EC2
After transferring, SSH into your EC2 instance and check if the files are uploaded correctly:
```bash
ls -l /var/www/html
```

If the files are missing, ensure the `scp` path is correct and rerun the upload command.

---

## Step 6: Restart the Web Server
For Apache:
```bash
sudo systemctl restart apache2
```
For Nginx:
```bash
sudo systemctl restart nginx
```

---

## Step 7: Access Your Website
Open your browser and visit:
```bash
http://your-ec2-ip
```
If you see your frontend project, **congratulations! Your site is live on AWS EC2.** 🎉

---

## Troubleshooting
- **403 Forbidden**: Run `sudo chmod -R 755 /var/www/html`
- **Changes not reflecting**: Clear browser cache or restart Apache/Nginx
- **Site not loading**: Check your **EC2 security group settings** and allow HTTP (port 80)
- **Cannot connect via SSH**: Ensure port 22 is open in security groups

---

## Conclusion
You have successfully deployed a frontend project on AWS EC2! 🚀 If you have any issues, feel free to ask for help.

