# Session-06: Self-Hosted Photo Management with Immich

## Project Overview

For this lab, I installed **Immich on my Ubuntu VM using Docker Compose**. Immich is a self-hosted photo and video management application that allows me to store and manage my own photo library through a web interface.

After completing my Pi-hole and File Browser labs, I wanted to continue building services on my Ubuntu VM and learning more about Docker, networking, storage, and self-hosted applications.

For this project, I successfully deployed Immich on **port 2283**, accessed the web interface from my browser, created my account, logged in, and uploaded my first photo.

---

## Why Use Immich?

One reason I wanted to learn Immich is because it gives me another example of what **self-hosting** actually means.

Normally, photos and videos may be stored locally on a phone/computer or uploaded to a third-party cloud service. With Immich, I can run the application on infrastructure that I manage and use a web or mobile interface to access my photo library.

The basic idea looks like this:

```text
Phone / Computer
       |
       | Browser or Immich App
       |
       v
Ubuntu VM IP : 2283
       |
       v
Docker
       |
       v
Immich
       |
       +---- Photo / Video Library
       |
       +---- PostgreSQL Database
```

This makes Immich useful for learning about **self-hosted storage, Docker services, databases, networking, backups, and data ownership**.

It also helped me see the difference between simply storing a picture in a folder and running an application that manages an entire photo library.

Immich uses the uploaded photos and videos along with application data stored in its database to provide the photo-management experience.

---

## 1. Finding My Ubuntu VM's IP Address

I first opened my Ubuntu VM and checked its IP address.

I used:

```bash
ip address
```

This displayed the network interfaces and IP addresses assigned to my Ubuntu VM.

I needed the VM's IP address for two reasons:

1. To connect remotely to Ubuntu using SSH.
2. To access the Immich web interface from my browser.

---

## 2. Connecting to the VM with SSH

After identifying the correct IP address, I connected to my Ubuntu VM using SSH.

Example:

```bash
ssh <username>@<VM-IP-ADDRESS>
```

Once connected, I checked the files and directories already on my machine before creating the Immich project.

This allowed me to confirm where I was working before making changes to the server.

---

## 3. Creating the Immich Directory

I created a dedicated directory for Immich:

```bash
mkdir ~/immich
```

Then I moved into the directory:

```bash
cd ~/immich
```

Keeping the application in its own directory helps organize the Docker Compose configuration, environment variables, database files, and photo library.

---

## 4. Downloading the Immich Docker Compose File

Instead of manually creating the entire Docker Compose configuration, I downloaded the current release configuration provided by Immich:

```bash
wget -O docker-compose.yml https://github.com/immich-app/immich/releases/latest/download/docker-compose.yml
```

This downloaded the Docker Compose configuration and saved it as:

```text
docker-compose.yml
```

The Docker Compose file defines the services and containers Immich needs to operate.

---

## 5. Downloading the Environment File

Next, I downloaded Immich's example environment configuration:

```bash
wget -O .env https://github.com/immich-app/immich/releases/latest/download/example.env
```

This created:

```text
.env
```

The `.env` file contains environment variables used by the Docker Compose configuration.

Some of these variables control things such as:

```text
UPLOAD_LOCATION
DB_DATA_LOCATION
DB_PASSWORD
DB_USERNAME
DB_DATABASE_NAME
IMMICH_VERSION
```

This was another example of how Docker applications can separate the main Compose configuration from environment-specific settings.

---

## 6. Creating Storage Directories

I created directories for the Immich library and PostgreSQL database:

```bash
mkdir -p library postgres
```

These directories serve two different purposes.

```text
~/immich/
│
├── docker-compose.yml
├── .env
│
├── library/
│     └── Photo and video data
│
└── postgres/
      └── PostgreSQL database data
```

The `library` directory provides persistent storage for Immich's uploaded assets based on the configured upload location.

The `postgres` directory provides persistent storage for the PostgreSQL database.

This connected directly to something I learned during my File Browser lab:

> **The container can be replaceable, but the important data needs to persist.**

---

## Understanding Immich's Database

My File Browser lab helped me understand why the database portion of Immich matters.

Immich is not just storing pictures in a folder.

There are two different types of information involved:

```text
PHOTO / VIDEO DATA
        |
        └── Actual uploaded assets

DATABASE DATA
        |
        └── Information Immich needs
            to organize and manage
            the application
```

Immich uses PostgreSQL to store application information and metadata associated with the photo library.

This is why protecting only the photo directory would not represent a complete backup strategy. The photo/video files and the database both matter.

---

## 7. Starting Immich

After preparing the Immich directory and configuration, I started the Docker services.

```bash
docker compose up -d
```

The `-d` option runs the containers in **detached mode**, allowing them to continue running in the background while returning control of my terminal.

I could verify the containers with:

```bash
docker compose ps
```

---

## 8. Accessing Immich on Port 2283

Immich's web server uses port **2283**.

After the containers were running, I entered my Ubuntu VM's IP address and port 2283 into my browser:

```text
http://<VM-IP-ADDRESS>:2283
```

This connected my browser to the Immich application running inside Docker on my Ubuntu VM.

Conceptually:

```text
Browser
   |
   | HTTP request
   v
Ubuntu VM
Port 2283
   |
   v
Immich Container
```

The Immich web interface successfully loaded.

---

## 9. Creating My Account and Logging In

After reaching the Immich web interface, I completed the initial account setup and successfully logged into the application.

This also introduced another security concept.

The web interface may be accessible over the network, but that does not mean everyone should automatically have access to the photo library.

Authentication helps answer:

> **Who are you?**

The application can then determine what that authenticated user is authorized to access.

This continues the authentication, authorization, and access-control concepts I started exploring during my File Browser lab.

---

## 10. Uploading My First Photo

After logging in, I tested the application by uploading my first photo.

The upload was successful, and I was able to see the photo inside my Immich library.

At that point I had verified the complete path:

```text
Ubuntu VM
    ↓
Docker
    ↓
Immich Services
    ↓
Port 2283
    ↓
Web Interface
    ↓
Authentication
    ↓
Photo Upload
    ↓
Persistent Storage
```

My Immich deployment was working.

---

# Why This Project Is Useful

What I liked about this lab is that the final product is something I can actually understand from a user's perspective.

I uploaded a photo through a website, but there was much more happening behind that simple action.

Behind the interface I had:

```text
Linux
   +
Docker
   +
Docker Compose
   +
Networking
   +
Port Mapping
   +
Environment Variables
   +
Persistent Storage
   +
PostgreSQL
   +
Authentication
   =
Self-Hosted Photo Management
```

This helped me understand how several technologies can work together to provide one application.

---

# Security and Backup Considerations

Self-hosting also means taking responsibility for protecting the service and its data.

Immich should not be treated as the only copy of important photos and videos.

A proper backup strategy should protect both the uploaded photo/video files and the application database. The database contains information Immich needs to understand and manage the library, while the actual media files are stored separately.

I would also need to consider authentication, network exposure, software updates, storage capacity, and secure remote access before treating this as a production service or making it available outside my private lab environment.

For now, this deployment is part of my Ubuntu and Docker learning environment.

---

# Commands Used During the Lab

### Check Network Information

```bash
ip address
```

### Create Immich Directory

```bash
mkdir ~/immich
```

### Enter Immich Directory

```bash
cd ~/immich
```

### Download Docker Compose Configuration

```bash
wget -O docker-compose.yml https://github.com/immich-app/immich/releases/latest/download/docker-compose.yml
```

### Download Environment Configuration

```bash
wget -O .env https://github.com/immich-app/immich/releases/latest/download/example.env
```

### Create Storage Directories

```bash
mkdir -p library postgres
```

### Start Immich

```bash
docker compose up -d
```

### Check Immich Containers

```bash
docker compose ps
```

### Access Immich

```text
http://<VM-IP-ADDRESS>:2283
```

---

# What I Learned

This lab gave me more hands-on experience with:

* Ubuntu Linux
* SSH
* Docker
* Docker Compose
* Self-hosted applications
* Port 2283
* Environment variables
* Persistent storage
* PostgreSQL
* Application databases
* Photo and video storage
* Authentication
* Network services
* Web-based applications
* Data ownership
* Backup planning

One of the biggest things I am starting to understand from these labs is that **running the container is only one part of deploying a service**.

I also need to understand:

**Where is the data stored?**

**What happens to the data if the container is recreated?**

**What information is stored in the database?**

**Who can access the application?**

**What port is the application using?**

**How would I protect and back up the data?**

Those questions are helping me understand what is actually happening behind the applications I use instead of only focusing on getting them to run.

---

# Final Result

✅ Identified my Ubuntu VM's IP address
✅ Connected to my VM using SSH
✅ Created a dedicated Immich directory
✅ Downloaded the current Immich Docker Compose configuration
✅ Downloaded and configured the `.env` file
✅ Created persistent library and PostgreSQL directories
✅ Started Immich using Docker Compose
✅ Successfully connected to Immich on port 2283
✅ Created my account and logged in
✅ Successfully uploaded my first photo
✅ Continued learning Docker persistence and application databases

---

## Final Reflection

This project helped connect several things I have been learning across my Ubuntu labs.

With Pi-hole, I learned more about **DNS, ports, Docker networking, and troubleshooting**.

With File Browser, I learned more about **permissions, UID/GID, least privilege, bind mounts, and persistence**.

Now with Immich, I am seeing how **Docker, networking, persistent storage, databases, authentication, and an actual application all work together**.

Uploading one photo looked simple from the browser, but understanding what had to happen behind that upload was the real purpose of this lab.

*Session-06 of my Ubuntu Lab.*
