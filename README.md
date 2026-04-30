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



# Secure Packages with AWS CodeArtifact 📦🔒

---

##  Overview

Set up AWS CodeArtifact as a private backup repository for Java dependencies, so Maven fetches packages securely through CodeArtifact instead of directly from the public internet — keeping the CI/CD pipeline reliable even if public packages go offline.

**Tools Used:** AWS EC2 · AWS CodeArtifact · IAM · Maven · GitHub · VS Code

---

##  Steps

**Step 1 — Create a CodeArtifact Repository**
In the AWS Console, create a repository under a domain. Set `maven-central-store` as the upstream repository so Maven Central acts as a fallback source.

<img width="1898" height="799" alt="Screenshot 2026-03-23 011643" src="https://github.com/user-attachments/assets/df9ec6d2-faad-48cf-8042-8e5317a47bba" />


**Step 2 — Create an IAM Policy**
Create a policy that grants EC2 permission to get authorization tokens, fetch repository endpoints, and read packages from CodeArtifact.

<img width="1887" height="855" alt="Screenshot 2026-03-23 014233" src="https://github.com/user-attachments/assets/e72a5385-e6f2-423e-8a67-82f6ec212d16" />


**Step 3 — Create and Attach an IAM Role to EC2**
Create a role, attach the policy from Step 2, then assign it to your EC2 instance via **EC2 → Actions → Security → Modify IAM Role**.

<img width="1908" height="864" alt="Screenshot 2026-03-23 020439" src="https://github.com/user-attachments/assets/112fef31-4c52-4c44-9326-a2eb0035b631" />

**Step 4 — Get the CodeArtifact Auth Token**
Run this in your VS Code terminal to generate a temporary access token:
```bash
export CODEARTIFACT_AUTH_TOKEN=`aws codeartifact get-authorization-token \
  --domain nextwork \
  --domain-owner <YOUR_ACCOUNT_ID> \
  --region <YOUR_REGION> \
  --query authorizationToken --output text`
```

**Step 5 — Configure settings.xml**
Create `~/.m2/settings.xml` using the snippet from **CodeArtifact → View connection instructions → mvn**. This file tells Maven where to find dependencies and how to authenticate with CodeArtifact.

<img width="767" height="458" alt="image" src="https://github.com/user-attachments/assets/5029fbe0-aade-4a2f-8fe2-83ec276b7a34" />


**Step 6 — Compile with Maven**
```bash
mvn -s ~/.m2/settings.xml compile
```
A successful run shows  in the logs and ends with `BUILD SUCCESS`. Packages will also appear in your CodeArtifact repository on the AWS Console.

<img width="1334" height="576" alt="image" src="https://github.com/user-attachments/assets/2c9ea014-cbc7-406e-8d58-3a1c653a1097" />

<img width="1884" height="825" alt="Screenshot 2026-03-28 132701" src="https://github.com/user-attachments/assets/db59c5db-1626-4b7a-b324-bb62b794e82e" />



---

## 🔗 Resources
- [AWS CodeArtifact Docs](https://docs.aws.amazon.com/codeartifact/)



# Continuous Integration with AWS CodeBuild 🏗️⚙️

---

##  Overview

Set up AWS CodeBuild to automatically compile and package the Java web app into a deployable WAR file every time code is pushed to GitHub. This is the CI (Continuous Integration) stage of the DevOps pipeline — no more manual builds.

**Tools Used:** AWS CodeBuild · AWS S3 · IAM · GitHub · Maven · VS Code

---

##  Steps

**Step 1 — Create an S3 Bucket for Build Artifacts**
In the AWS Console, create an S3 bucket to store the compiled output files from each build. This bucket acts as the handoff point between CodeBuild and later deployment stages.

**Step 2 — Create the buildspec.yml File**
In your project root, create a `buildspec.yml` file. This is the blueprint that tells CodeBuild exactly what to do during the build:
```yaml
version: 0.2

phases:
  install:
    commands:
      - echo Installing Maven dependencies...
  pre_build:
    commands:
      - echo Logging into CodeArtifact...
      - export CODEARTIFACT_AUTH_TOKEN=`aws codeartifact get-authorization-token \
          --domain nextwork \
          --domain-owner <YOUR_ACCOUNT_ID> \
          --region <YOUR_REGION> \
          --query authorizationToken --output text`
  build:
    commands:
      - echo Build started on `date`
      - mvn -s settings.xml compile
  post_build:
    commands:
      - echo Build completed on `date`
      - mvn -s settings.xml package

artifacts:
  files:
    - target/nextwork-web-project.war
  discard-paths: yes
```
Commit and push this file to GitHub:
```bash
git add buildspec.yml
git commit -m "Add buildspec.yml for CodeBuild"
git push origin main
```

**Step 3 — Create a CodeBuild IAM Role**
- Go to **IAM → Roles → Create Role** → select **CodeBuild** as the trusted service
- Attach these policies:
  - `AmazonS3FullAccess` — to upload artifacts to S3
  - `codeartifact-*name*-consumer-policy` — to fetch packages from CodeArtifact (created in Project 3)
- Name the role

**Step 4 — Create the CodeBuild Project**
- Go to **CodeBuild → Create build project**
- Fill in the settings:
  - **Project name:** `vktr-devops-cicd`
  - **Source:** GitHub → connect your repo
  - **Environment:** Amazon Linux, standard runtime, latest image
  - **Service role:** `codebuild-vktr-codebuild-cicd-service-role`
  - **Buildspec:** Use a buildspec file (CodeBuild will find `buildspec.yml` automatically)
  - **Artifacts:** Amazon S3 → select your bucket from Step 1
- Click **Create build project**

**Step 5 — Run the Build**
- Click **Start build** in your CodeBuild project
- Monitor progress in the **Build logs** tab — you'll see each phase execute in real time
- A successful build ends with `BUILD SUCCESS` and uploads a `.war` file to your S3 bucket

<img width="1670" height="669" alt="Screenshot 2026-04-15 165444" src="https://github.com/user-attachments/assets/df9d64c1-2abf-4091-b27b-e906ef066694" />


**Step 6 — Verify the Artifact in S3**
- Go to **S3 → your artifacts bucket**
- Confirm `nextwork-web-project.war` is present
- This WAR file is the packaged web app, ready for deployment in the next project

<img width="1822" height="533" alt="Screenshot 2026-04-15 172437" src="https://github.com/user-attachments/assets/e776e551-b46f-4ac3-8bcf-6d160f92f9de" />


---

## 🔄 CI Flow

```
Push code to GitHub
       ↓
CodeBuild triggers automatically
       ↓
Fetches dependencies via CodeArtifact
       ↓
Compiles & packages app → .war file
       ↓
Uploads artifact to S3
```

---

## 🔗 Resources
- [AWS CodeBuild Docs](https://docs.aws.amazon.com/codebuild/)
- [buildspec.yml Reference](https://docs.aws.amazon.com/codebuild/latest/userguide/build-spec-ref.html)



# Deploy a Web App with AWS CodeDeploy 


---

## 📋 Overview

Use AWS CodeDeploy to automatically deploy the WAR file (built by CodeBuild in Project 4) to an EC2 instance running Apache Tomcat. This is the CD (Continuous Deployment) stage of the pipeline.

**Tools Used:** AWS CodeDeploy · EC2 · IAM · S3 · Apache Tomcat · GitHub · VS Code

---

## 🚀 Steps

**Step 1 — Create IAM Roles**
Two roles are needed:

- **CodeDeploy Service Role** — allows CodeDeploy to access EC2:
  - Go to **IAM → Roles → Create Role** → select **CodeDeploy**
  - Attach `AWSCodeDeployRole` policy
 
<img width="1900" height="676" alt="Screenshot 2026-04-19 195023" src="https://github.com/user-attachments/assets/1690b5dd-0938-4aa1-a07d-8a9f3425fa47" />


- **EC2 Instance Role** — allows EC2 to read from S3:
  - Create another role → select **EC2** as trusted entity
  - Attach `AmazonS3ReadOnlyAccess` and `codeartifact-nextwork-consumer-policy`
  - Name it `EC2-instance-nextwork-cicd`
  - Attach to your EC2 instance via **EC2 → Actions → Security → Modify IAM Role**

**Step 2 — Create the appspec.yml File**
In the root of your project, create `appspec.yml` — this tells CodeDeploy where to place files and what scripts to run:
```yaml
version: 0.0
os: linux
files:
  - source: /target/nextwork-web-project.war
    destination: /var/lib/tomcat9/webapps/
hooks:
  BeforeInstall:
    - location: scripts/install_dependencies.sh
      timeout: 300
      runas: root
  ApplicationStop:
    - location: scripts/stop_server.sh
      timeout: 300
      runas: root
  ApplicationStart:
    - location: scripts/start_server.sh
      timeout: 300
      runas: root
```

Create the supporting scripts in a `scripts/` folder, then push everything to GitHub:
```bash
git add appspec.yml scripts/
git commit -m "Add CodeDeploy appspec and deployment scripts"
git push origin main
```

**Step 3 — Install CodeDeploy Agent on EC2**
SSH into your EC2 instance (via VS Code Remote-SSH) and run:
```bash
sudo apt update
sudo apt install ruby wget -y
wget https://aws-codedeploy-<YOUR_REGION>.s3.amazonaws.com/latest/install
chmod +x ./install
sudo ./install auto
sudo systemctl start codedeploy-agent
sudo systemctl status codedeploy-agent
```
You should see `active (running)` in the status output.

**Step 4 — Create a CodeDeploy Application**
- Go to **CodeDeploy → Applications → Create application**
- **Application name:** 
- **Compute platform:** `EC2/On-premises`
- Click **Create application**

**Step 5 — Create a Deployment Group**
Inside your CodeDeploy application:
- Click **Create deployment group**
- **Group name:** `nextwork-web-deploy-group`
- **Service role:** `NextWorkCodeDeployRole`
- **Deployment type:** In-place
- **Environment:** Amazon EC2 Instances → tag your instance (e.g. `Name: nextwork-devops-yourname`)
- **Agent configuration:** Choose **Now and schedule updates** (14 days)
- **Deployment settings:** `CodeDeployDefault.AllAtOnce`
- **Load balancer:** Uncheck (not needed for this project)
- Click **Create deployment group**

**Step 6 — Create and Run a Deployment**
- In your deployment group, click **Create deployment**
- **Revision type:** Amazon S3
- **Revision location:** paste the S3 URI of your `.zip` artifact from Project 4
- **Revision file type:** `.zip`
- Click **Create deployment**

<img width="1892" height="822" alt="Screenshot 2026-04-21 121152" src="https://github.com/user-attachments/assets/70d77978-e717-4dec-8a38-fd69ed00c308" />

<img width="1842" height="595" alt="Screenshot 2026-04-21 121655" src="https://github.com/user-attachments/assets/b448f629-bdd6-4096-abee-9b2feb662fa8" />

<img width="1374" height="379" alt="Screenshot 2026-04-21 125049" src="https://github.com/user-attachments/assets/9a92bbbc-b1c3-4de1-90f3-b4eb788f450f" />


The deployment takes ~30 seconds. Once complete, open your EC2 instance's public IP in a browser — your Java web app should be live! 🎉

---

## 🔄 CD Flow

```
WAR file stored in S3 (from CodeBuild)
            ↓
CodeDeploy picks up the artifact
            ↓
Runs appspec.yml hooks on EC2
            ↓
WAR deployed to Tomcat → App goes live
```

---

## 🔗 Resources
- [AWS CodeDeploy Docs](https://docs.aws.amazon.com/codedeploy/)
- [appspec.yml Reference](https://docs.aws.amazon.com/codedeploy/latest/userguide/reference-appspec-file.html)


# Build a CI/CD Pipeline with AWS CodePipeline 🔁

---

##  Overview

Tie everything together using AWS CodePipeline to fully automate the entire workflow — from a GitHub code push all the way to a live deployment on EC2. No more manual builds or deployments; every change triggers the pipeline automatically.

**Tools Used:** AWS CodePipeline · CodeBuild · CodeDeploy · GitHub · S3 · EC2 · VS Code

---

##  Steps

**Step 1 — Create the Pipeline**
- Go to **CodePipeline → Create pipeline**
- **Pipeline name** 
- **Pipeline type:** V2
- **Execution mode:** Superseded *(ensures latest push always takes priority)*
- **Service role:** Create a new service role (auto-generated name is fine)
- Click **Next**

**Step 2 — Add the Source Stage (GitHub)**
- **Source provider:** GitHub
- Click **Connect to GitHub** and authorize AWS
- **Repository:** `cjoose repository`
- **Branch:** `main`
- **Detection option:** Amazon CloudWatch Events *(auto-triggers pipeline on every push)*
- **Output artifact format:** CodePipeline Default
- Click **Next**

**Step 3 — Add the Build Stage (CodeBuild)**
- **Build provider:** AWS CodeBuild
- **Region:** match your CodeBuild project region
- **Project name:** `choose project name` 
- Click **Next**

**Step 4 — Add the Deploy Stage (CodeDeploy)**
- **Deploy provider:** AWS CodeDeploy
- **Application name:** `choose application name` *(created in Project 5)*
- **Deployment group:** `choose deployment group`
- Click **Next** → review → **Create pipeline**

The pipeline triggers immediately after creation — watch all three stages run in real time.

**Step 5 — Test the Full Pipeline**
Make a small change to your app and push it:
```bash
# Edit your web app page
cd ~/nextwork-web-project
# Make a change to src/main/webapp/index.jsp in VS Code

git add .
git commit -m "Test pipeline trigger"
git push origin main
```
Head to **CodePipeline** and watch the stages progress:
- ✅ **Source** — picks up your GitHub commit
- ✅ **Build** — CodeBuild compiles and packages the app
- ✅ **Deploy** — CodeDeploy pushes it live to EC2

**Step 6 — Verify the Live App**
Once all stages show green, open your EC2 instance's public IP in a browser — your updated app should be live within minutes. 🎉

**Step 7 — Test Rollback (Optional)**
If a bad deployment goes through:
- Go to **CodePipeline → your pipeline → Deploy stage**
- Click **Start rollback**
- Wait ~30 seconds — CodeDeploy reverts to the previous working version automatically

---

## 🔄 Full CI/CD Flow

```
Push code to GitHub
        ↓
CodePipeline detects change (CloudWatch Events)
        ↓
CodeBuild — fetches from CodeArtifact, compiles, packages WAR
        ↓
Artifact stored in S3
        ↓
CodeDeploy — pulls artifact, runs appspec.yml, deploys to EC2
        ↓
App is live 🚀
```

---

## 🔗 Resources
- [AWS CodePipeline Docs](https://docs.aws.amazon.com/codepipeline/)
