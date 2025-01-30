### **Amazon EC2 Storage: EBS, AMI, and Snapshots (Detailed Explanation + Practical + Project)**  

## **1. Introduction to EC2 Storage**  
Amazon Elastic Compute Cloud (EC2) provides virtual servers where you can run applications. However, for these servers to store data, AWS provides **Elastic Block Store (EBS), Amazon Machine Images (AMI), and Snapshots**.  

- **EBS (Elastic Block Store)**: Persistent storage volumes attached to EC2 instances.  
- **AMI (Amazon Machine Image)**: Pre-configured templates for launching EC2 instances.  
- **Snapshots**: Backups of EBS volumes that can be used to restore or create new volumes.  

---

## **2. EBS (Elastic Block Store) - Persistent Storage for EC2**  
### **2.1 What is EBS?**  
Amazon EBS provides **block-level** storage that is automatically replicated within an Availability Zone (AZ). It acts like a hard disk attached to an EC2 instance.  

### **2.2 Features of EBS**  
- **Persistent**: Data remains even after EC2 is stopped or terminated.  
- **Scalable**: Storage size can be increased without data loss.  
- **Secure**: Supports encryption.  
- **Fast Performance**: Optimized for low latency and high throughput.  

### **2.3 Types of EBS Volumes**  
| Type | Use Case | Performance | Cost |
|------|----------|-------------|------|
| **gp3 (General Purpose SSD)** | Web servers, small databases | Up to 16,000 IOPS | Low |
| **gp2 (General Purpose SSD)** | Boot volumes, dev/test | Up to 3,000 IOPS | Medium |
| **io2/io1 (Provisioned IOPS SSD)** | Large databases | Up to 256,000 IOPS | High |
| **st1 (Throughput Optimized HDD)** | Big data, data warehouses | High throughput | Lower |
| **sc1 (Cold HDD)** | Infrequent access | Low cost | Lowest |

---

### **2.4 Practical: Creating and Attaching an EBS Volume**
#### **Step 1: Create an EBS Volume**
1. Open AWS Management Console → Navigate to **EC2 Dashboard**  
2. Click **Elastic Block Store (EBS) → Volumes**  
3. Click **Create Volume**  
   - Choose volume type (**gp3/gp2** recommended for general use).  
   - Select **Size** (e.g., 10 GiB).  
   - Select the **Availability Zone** (must match your EC2 instance's AZ).  
   - Click **Create Volume**.  

#### **Step 2: Attach EBS to EC2**
1. Select the volume → Click **Actions → Attach Volume**.  
2. Choose the target **EC2 instance**.  
3. Note the device name (e.g., `/dev/xvdf`).  

#### **Step 3: Mount the EBS Volume in Linux**
1. Connect to EC2 instance via SSH:  
   ```bash
   ssh -i my-key.pem ubuntu@your-ec2-ip
   ```  
2. List attached disks:  
   ```bash
   lsblk
   ```  
3. Format the volume (if new, use ext4):  
   ```bash
   sudo mkfs -t ext4 /dev/xvdf
   ```  
4. Create a mount point:  
   ```bash
   sudo mkdir /data
   ```  
5. Mount the volume:  
   ```bash
   sudo mount /dev/xvdf /data
   ```  
6. Make the mount persistent after reboot by adding it to `/etc/fstab`:  
   ```bash
   echo "/dev/xvdf /data ext4 defaults,nofail 0 2" | sudo tee -a /etc/fstab
   ```  

---

## **3. AMI (Amazon Machine Image) - EC2 Templates**  
### **3.1 What is an AMI?**  
An AMI is a **pre-configured template** containing:  
- **OS** (Ubuntu, Amazon Linux, Windows, etc.)  
- **Pre-installed applications** (LAMP stack, database, etc.)  
- **User permissions and settings**  

### **3.2 Types of AMIs**
- **Public AMIs**: Provided by AWS or third parties (e.g., Ubuntu, Windows Server).  
- **Custom AMIs**: Created by users with specific configurations.  
- **AWS Marketplace AMIs**: Pre-configured AMIs for specific applications.  

---

### **3.3 Practical: Creating a Custom AMI**
#### **Step 1: Configure EC2 Instance**
1. Launch an **EC2 instance** and install necessary applications.  
   ```bash
   sudo apt update && sudo apt install apache2 -y
   echo "Welcome to my website" | sudo tee /var/www/html/index.html
   ```
2. Ensure Apache is running:  
   ```bash
   sudo systemctl start apache2
   sudo systemctl enable apache2
   ```

#### **Step 2: Create AMI**
1. In AWS EC2 Dashboard, select the instance → Click **Actions → Create Image**.  
2. Enter:  
   - **Image Name**: `MyWebServerAMI`  
   - **Description**: "Custom AMI with Apache installed".  
   - Click **Create Image**.  

#### **Step 3: Launch a New EC2 Instance from AMI**
1. Go to **AMIs** in EC2 Dashboard.  
2. Select `MyWebServerAMI` → Click **Launch Instance from Image**.  

---

## **4. Snapshots - Backing Up EBS Volumes**  
### **4.1 What is a Snapshot?**  
A snapshot is a **point-in-time backup** of an EBS volume.  
- Snapshots are **incremental** (only changed data is saved).  
- Can be used to **restore** or **create new EBS volumes**.  

### **4.2 Practical: Creating and Restoring a Snapshot**
#### **Step 1: Create a Snapshot**
1. Go to **EC2 Dashboard → Elastic Block Store (EBS) → Volumes**.  
2. Select your EBS volume → Click **Actions → Create Snapshot**.  
3. Provide a name and description → Click **Create Snapshot**.  

#### **Step 2: Restore from Snapshot**
1. Go to **Snapshots** in EC2 Dashboard.  
2. Select the snapshot → Click **Actions → Create Volume from Snapshot**.  
3. Select the Availability Zone → Click **Create Volume**.  
4. Attach the new volume to an EC2 instance and mount it as described in EBS steps.  

---

## **5. Real-world Project: Automated Backup System**
### **Project Goal:**  
Create a **backup system** that automatically takes snapshots of critical EBS volumes.  

### **Step 1: Create an IAM Role for EC2**
1. Go to **IAM → Roles** → Create role.  
2. Choose **AWS service → EC2**.  
3. Attach **AmazonEC2FullAccess** policy.  
4. Name it `EC2-Snapshot-Role`.  

### **Step 2: Attach Role to EC2**
1. Go to **EC2 Dashboard → Select Instance → Actions → Security → Modify IAM Role**.  
2. Select `EC2-Snapshot-Role` and attach it.  

### **Step 3: Automate Snapshots with a Shell Script**
1. SSH into EC2 and create a script:  
   ```bash
   nano backup.sh
   ```
2. Add the following script:
   ```bash
   #!/bin/bash
   VOLUME_ID="vol-xxxxxxxxxxxxxx"
   DESCRIPTION="Automated backup - $(date +%F)"
   aws ec2 create-snapshot --volume-id $VOLUME_ID --description "$DESCRIPTION"
   ```
3. Make it executable:  
   ```bash
   chmod +x backup.sh
   ```
4. Schedule it using **cron**:  
   ```bash
   crontab -e
   ```
   Add this line to run daily at midnight:  
   ```bash
   0 0 * * * /home/ubuntu/backup.sh
   ```

---

## **6. Summary**
- **EBS** provides persistent storage for EC2.  
- **AMI** creates reusable machine templates.  
- **Snapshots** back up and restore EBS volumes.  
- **Automated Backup** ensures data safety.  
