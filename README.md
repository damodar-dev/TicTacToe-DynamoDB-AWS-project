# 🎮 Tic-Tac-Toe Web App — AWS EC2 + DynamoDB + Python Flask
---

## 📌 Project Overview

A fully functional **Tic-Tac-Toe multiplayer web application** built on AWS Cloud.  
The application uses **AWS DynamoDB** as a NoSQL backend to store real-time game state,  
**Flask** to handle web requests, and runs on an **AWS EC2** instance — with secure  
access to DynamoDB via an **IAM Role** (no hardcoded credentials).

> ✅ Two players can create, join, and play games — all state managed in real-time via DynamoDB.

---

## 🏗️ Architecture

```
 Player 1 (Browser)          Player 2 (Browser)
        │                           │
        └──────────┬────────────────┘
                   │  HTTP (port 5000)
                   ▼
        ┌──────────────────────┐
        │    AWS EC2 Instance   │
        │   (Amazon Linux 2)    │
        │                       │
        │  Flask App (Python)   │
        │  GameController       │
        │  ConnectionManager    │
        └──────────┬────────────┘
                   │  IAM Role (Secure, No Hardcoded Credentials)
                   ▼
        ┌──────────────────────┐
        │     AWS DynamoDB      │
        │    Table: "Games"     │
        └──────────────────────┘
```

---

## ⚙️ Tech Stack

| Category | Tool / Service | Purpose |
|---|---|---|
| Cloud Compute | AWS EC2 (Amazon Linux 2) | Hosts the web application |
| Database | AWS DynamoDB | Stores game state & player data |
| Security | AWS IAM Role | Secure EC2 → DynamoDB access |
| Web Framework | Flask | Handles HTTP requests & renders UI |
| AWS SDK | Boto | Python SDK to interact with DynamoDB |
| Language | Python 2.7.18 | Application logic |
| Version Control | GitHub | Source code management |
| Region | ap-south-2 (Hyderabad) | AWS deployment region |
| App Port | 5000 | Flask application port |

---

## 🎯 Key Features

| Feature | Description |
|---|---|
| Create New Game | Player creates a game with a unique Game ID, specifying creator & invitee |
| Accept / Reject Invite | Invited player can accept or reject the game request |
| Real-time Board Updates | Each move is validated and instantly saved to DynamoDB |
| Turn Management | Automatic turn switching between Player 1 & Player 2 |
| Game Result Detection | Detects Win / Loss / Tie after every move |
| Persistent State | Full game state stored in DynamoDB — survives page refreshes |

---

## 🧩 Application Components

### 1. `ConnectionManager` Class
Manages the connection between the Flask app and DynamoDB.
- Sets up DynamoDB connection using IAM Role credentials
- Handles all read/write queries to the `Games` table
- Retrieves temporary credentials via EC2 Instance Metadata (IMDSv1 & v2)

### 2. `GameController` Class
Contains all the game business logic.
- `createNewGame()` — Initializes a new game entry in DynamoDB
- `acceptGame()` / `rejectGame()` — Updates game invite status
- `updateBoardAndTurn()` — Records player move & switches turn
- `checkForGameResult()` — Checks all win conditions after each move

### 3. DynamoDB `Games` Table Schema

| Attribute | Type | Description |
|---|---|---|
| GameId | String (PK) | Unique game identifier |
| Creator | String | Player who created the game |
| Invitee | String | Player who was invited |
| Status | String | PENDING / IN_PROGRESS / FINISHED |
| BoardState | String | Current state of the 3×3 board |
| Turn | String | Whose turn it is |
| Result | String | WIN / LOSS / TIE / IN_PROGRESS |

---

## 🛠️ Installation & Setup

### Step 1 — Launch EC2 Instance

- **AMI:** Amazon Linux 2
- **Region:** ap-south-2 (Hyderabad)
- **IAM Role:** Attach a role with `AmazonDynamoDBFullAccess` permission
- **Instance Metadata:** Set to **"V1 and V2"** *(not V2 only — required for Boto to retrieve temporary IAM credentials)*

**Security Group — Open these ports:**

| Port | Purpose |
|------|---------|
| 22 | SSH access |
| 5000 | Flask application |

---

### Step 2 — Install Development Tools & Dependencies

```bash
# Install essential build tools
sudo yum groupinstall -y "Development Tools"

# Install required system libraries for Python compilation
sudo yum install -y openssl-devel bzip2-devel libffi-devel
```

---

### Step 3 — Install Python 2.7.18

```bash
# Navigate to source directory
cd /usr/src

# Download Python 2.7.18
sudo wget https://www.python.org/ftp/python/2.7.18/Python-2.7.18.tgz

# Extract the archive
sudo tar xzf Python-2.7.18.tgz

# Build and install
cd Python-2.7.18
sudo ./configure --enable-optimizations
sudo make altinstall

# Verify the installation
python2.7 -V
```

> ℹ️ `altinstall` is used to avoid replacing the system's default Python binary.

---

### Step 4 — Install pip for Python 2.7

```bash
# Download the pip installer for Python 2.7
wget https://bootstrap.pypa.io/pip/2.7/get-pip.py

# Install pip using Python 2.7
python2.7 get-pip.py
```

---

### Step 5 — Install Required Python Packages

```bash
# Install Flask (web framework)
pip2.7 install Flask

# Install Boto (AWS SDK for Python 2)
pip2.7 install boto

# Install configparser (to read config.ini)
pip2.7 install configparser
```

---

### Step 6 — Install Git & Clone the Repository

```bash
# Install Git
sudo yum install git -y

# Navigate to home directory and clone the project
cd /home/ec2-user/
git clone https://github.com/damodar-dev/TicTacToe-DynamoDB-AWS-project.git

# Move into the project directory
cd TicTacToe-DynamoDB-AWS-project
```

---

### Step 7 — Create the Configuration File

Create a `config.ini` file inside the project directory:

```bash
vi config.ini
```

Add the following content:

```ini
[Settings]
TableName = Games
Region = ap-south-2

[dynamodb]
region = ap-south-2
endpoint = dynamodb.ap-south-2.amazonaws.com
```

> ⚠️ Update `region` and `endpoint` values if you are deploying in a different AWS region.

---

### Step 8 — Attach IAM Role to EC2

1. Go to **AWS Console → EC2 → Select your instance**
2. Click **Actions → Security → Modify IAM Role**
3. Attach a role with **`AmazonDynamoDBFullAccess`** policy
4. Click **Update IAM Role**

> ✅ The IAM Role allows the app to access DynamoDB securely — no hardcoded AWS Access Keys needed.

---

### Step 9 — Run the Application

```bash
# Make sure you are inside the project directory
cd /home/ec2-user/TicTacToe-DynamoDB-AWS-project

# Start the Flask application
python2.7 application.py \
  --config config.ini \
  --mode service \
  --endpoint dynamodb.ap-south-2.amazonaws.com \
  --serverPort 5000
```

---

### Step 10 — Access the Application

Open your browser and enter:

```
http://<EC2-PUBLIC-IP>:5000/
```

> ⚠️ Make sure **port 5000** is open in your EC2 Security Group inbound rules before accessing.

---

## 🔁 Game Workflow

```
1. Player 1 opens the app → Creates a new game (Status: PENDING)
              ↓
2. Player 2 receives the invite → Accepts it (Status: IN_PROGRESS)
              ↓
3. Players take turns → Each move saved instantly to DynamoDB
              ↓
4. After every move → checkForGameResult() runs automatically
              ↓
5. WIN / LOSS / TIE detected → Game status updated to FINISHED ✅
```

---

## ✅ Results & Outcomes

- ✅ Multiplayer Tic-Tac-Toe running live on AWS EC2
- ✅ Real-time game state stored and retrieved from DynamoDB
- ✅ Secure AWS access via IAM Role — no hardcoded credentials
- ✅ Automatic game result detection (Win / Loss / Tie) after every move
- ✅ Persistent game state that survives page refreshes
- ✅ Clean architecture — Flask routes, GameController, and ConnectionManager separated

---

## 🧠 Key Learnings

- AWS DynamoDB table design for real-time stateful applications
- Python Flask web framework and routing
- Boto SDK for DynamoDB read/write operations
- IAM Role-based secure access from EC2 to DynamoDB
- EC2 Instance Metadata (IMDSv1/v2) for temporary credential retrieval
- NoSQL data modeling for multiplayer game state management
- Configuring and running Python 2.7 on Amazon Linux 2

---

## 👨‍💻 Author

**S Damodararao**  
DevOps & Cloud Engineer  
🔗 [LinkedIn](https://www.linkedin.com/in/sdamodararao/)  
🐙 [GitHub](https://github.com/damodar-dev)


