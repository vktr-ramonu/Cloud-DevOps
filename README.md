# Setting Up a Web App in the Cloud ☁️

> **Building a complete CI/CD pipeline using AWS and VS Code**  

---

## 📋 Project Overview

This project sets up a Java-based web application on an AWS EC2 instance, connected and managed remotely through VS Code. It lays the foundation for the full CI/CD pipeline built across the 7-day challenge.

<img width="1419" height="454" alt="Screenshot 2026-03-02 130435" src="https://github.com/user-attachments/assets/4f141268-f38c-4476-85c7-5002222baf1d" />


**Tools & Services Used:** AWS EC2, IAM, Security Groups, Key Pairs, VS Code, Remote-SSH, Maven, Java (Amazon Corretto 8)

---

## 🚀 Steps

### Step 1 — Launch an EC2 Instance
- Log into the AWS Console and navigate to **EC2 → Instances → Launch Instance**
- Choose **Amazon Linux 2023 AMI** and **t2.micro** as the instance type
- Create a new key pair (e.g. `nextwork-keypair.pem`) and download it — keep this safe
- Configure the security group to allow **SSH on port 22** from your IP
- Launch the instance

---

### Step 2 — Set Up VS Code
- Download and install [Visual Studio Code](https://code.visualstudio.com/)
- Open VS Code and install the **Remote - SSH** extension (`Ctrl + Shift + X` → search "Remote - SSH")
- This extension allows you to connect directly to your EC2 instance and work on it as if it were your local machine

---

### Step 3 — Set Key Pair Permissions
- Locate your downloaded `.pem` file and restrict access to it so only you can read it
- Run these commands in your terminal (replace `USERNAME` with your Windows username):
```bash
icacls "nextwork-keypair.pem" /reset
icacls "nextwork-keypair.pem" /grant:r "USERNAME:R"
icacls "nextwork-keypair.pem" /inheritance:r
```
> ⚠️ This step is required — SSH will reject the connection if your key file has open permissions.

<img width="1701" height="979" alt="Screenshot 2026-03-02 152528" src="https://github.com/user-attachments/assets/cdefc6a8-35e9-4828-bac7-b679f6c7912b" />


---

### Step 4 — Connect VS Code to EC2 via SSH
- In VS Code, open the SSH config file and add your EC2 details:
```
Host my-ec2-instance
    HostName <EC2_PUBLIC_IPv4_DNS>
    User ec2-user
    IdentityFile ~/.ssh/nextwork-keypair.pem
```
- Click the **Remote - SSH** icon in the bottom left of VS Code → **Connect to Host** → select your instance
- Type `yes` when prompted to trust the host — this saves the server fingerprint to your `known_hosts` file
- You are now connected to your EC2 instance remotely inside VS Code

<img width="431" height="324" alt="image" src="https://github.com/user-attachments/assets/cd1efb92-be42-4d97-aeb6-b4e9e6c7d2f3" />


---

### Step 5 — Install Maven and Java
- In the VS Code terminal (now connected to EC2), run:
```bash
# Install Java (Amazon Corretto 8)
sudo dnf install -y java-1.8.0-amazon-corretto

# Install Maven
sudo dnf install -y maven
```
- Verify both installations:
```bash
java -version
mvn -v
```
> Maven handles compiling, linking, packaging, and testing the application. Java is required as the app runs on the JVM.

<img width="1417" height="350" alt="Screenshot 2026-03-02 174646" src="https://github.com/user-attachments/assets/0a086253-3083-459c-8f67-2e9985feae1f" />

<img width="1410" height="523" alt="Screenshot 2026-03-02 174703" src="https://github.com/user-attachments/assets/824ed227-01dc-4db9-9875-71d7e43463ce" />

---

### Step 6 — Generate the Web App with Maven
- Run the following Maven command to scaffold a Java web application:
```bash
mvn archetype:generate \
  -DgroupId=com.nextwork.app \
  -DartifactId=nextwork-web-project \
  -DarchetypeArtifactId=maven-archetype-webapp \
  -DinteractiveMode=false
```
- This automatically creates the full project folder structure for a Java web app

---

### Step 7 — Explore and Edit the Web App in VS Code
- In VS Code's file explorer, open the `nextwork-web-project` folder
- The key folders and files are:
  - `src/` — holds all source code
  - `src/main/webapp/` — web-facing files (HTML, CSS, JS)
  - `index.jsp` — the home page of the app (like `index.html` but supports dynamic Java content)
- Open `index.jsp` and edit the content to personalise your web app
- Save changes with `Ctrl + S`

---

## 📁 Project Structure
```
nextwork-web-project/
├── src/
│   └── main/
│       └── webapp/
│           └── index.jsp   ← Edit this to customize your web app
├── pom.xml                 ← Maven project config
```

---

## 🔗 Resources
- [AWS EC2 Documentation](https://docs.aws.amazon.com/ec2/)
- [VS Code Remote SSH](https://code.visualstudio.com/docs/remote/ssh)
- [Apache Maven](https://maven.apache.org/)
