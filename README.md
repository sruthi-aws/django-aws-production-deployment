# django-aws-production-deployment
Django application deployed on AWS EC2 using Gunicorn and Nginx
# Django Production Deployment on AWS EC2 using Gunicorn & Nginx

##  Project Overview

This project demonstrates the deployment of a Django web application on an **AWS EC2 Ubuntu server** using **Gunicorn** as the application server and **Nginx** as a reverse proxy.

The project covers the basic steps required to move a Django application from a local development environment to a production-style environment on AWS.

##  Architecture

```text
                    Internet
                       │
                       │ HTTP / HTTPS
                       ▼
                ┌───────────────┐
                │    Nginx      │
                │ Reverse Proxy │
                └───────┬───────┘
                        │
                        │ Proxy
                        ▼
                ┌───────────────┐
                │    Gunicorn   │
                │ WSGI Server   │
                └───────┬───────┘
                        │
                        ▼
                ┌───────────────┐
                │ Django App    │
                └───────┬───────┘
                        │
                        ▼
                ┌───────────────┐
                │ SQLite / DB   │
                └───────────────┘

                 AWS EC2 Ubuntu
```

##  Technologies Used

* **AWS EC2** – Cloud server for hosting the application
* **Ubuntu Linux** – Server operating system
* **Python** – Programming language
* **Django** – Web framework
* **Gunicorn** – Python WSGI application server
* **Nginx** – Web server and reverse proxy
* **Git & GitHub** – Source code and version control
* **Virtual Environment** – Isolated Python environment

##  Deployment Steps

### 1. Launch an AWS EC2 Instance

An Ubuntu EC2 instance was created to host the Django application.

The required inbound security group rules were configured to allow:

* SSH – Port `22`
* HTTP – Port `80`
* HTTPS – Port `443`

### 2. Connect to the EC2 Server

Connect to the server using SSH:

```bash
ssh -i your-key.pem ubuntu@<EC2-PUBLIC-IP>
```

### 3. Update the Server

```bash
sudo apt update
sudo apt upgrade -y
```

### 4. Install Required Packages

```bash
sudo apt install python3-pip python3-venv nginx git -y
```

### 5. Clone the Django Project

```bash
git clone <YOUR-GITHUB-REPOSITORY-URL>
cd <PROJECT-DIRECTORY>
```

### 6. Create a Virtual Environment

```bash
python3 -m venv venv
source venv/bin/activate
```

### 7. Install Python Dependencies

```bash
pip install -r requirements.txt
```

If a `requirements.txt` file is not available:

```bash
pip install django gunicorn
```

### 8. Configure Django

The Django `settings.py` file was configured for deployment.

Important settings include:

```python
ALLOWED_HOSTS = [
    "<EC2-PUBLIC-IP>",
    "localhost",
    "127.0.0.1",
]
```

### 9. Run Database Migrations

```bash
python manage.py migrate
```

### 10. Collect Static Files

```bash
python manage.py collectstatic
```

This collects Django static files into the configured static directory for serving in production.

### 11. Test Gunicorn

Gunicorn was used to serve the Django application.

```bash
gunicorn --bind 0.0.0.0:8000 <project_name>.wsgi:application
```

The application can then be tested using:

```text
http://<EC2-PUBLIC-IP>:8000
```

### 12. Create a Gunicorn Systemd Service

A systemd service was configured so Gunicorn can run as a background service and automatically restart when required.

Example:

```ini
[Unit]
Description=Gunicorn service for Django application
After=network.target

[Service]
User=ubuntu
Group=www-data
WorkingDirectory=/home/ubuntu/<project-directory>
ExecStart=/home/ubuntu/<project-directory>/venv/bin/gunicorn \
    --workers 3 \
    --bind unix:/home/ubuntu/<project-directory>/<project-name>.sock \
    <project-name>.wsgi:application

[Install]
WantedBy=multi-user.target
```

Enable and start the service:

```bash
sudo systemctl daemon-reload
sudo systemctl enable gunicorn
sudo systemctl start gunicorn
```

Check the service:

```bash
sudo systemctl status gunicorn
```

### 13. Configure Nginx

Nginx was configured as a reverse proxy in front of Gunicorn.

Example configuration:

```nginx
server {
    listen 80;
    server_name <EC2-PUBLIC-IP>;

    location /static/ {
        alias /home/ubuntu/<project-directory>/staticfiles/;
    }

    location /media/ {
        alias /home/ubuntu/<project-directory>/media/;
    }

    location / {
        proxy_pass http://unix:/home/ubuntu/<project-directory>/<project-name>.sock;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

Enable the configuration:

```bash
sudo ln -s /etc/nginx/sites-available/<project-name> /etc/nginx/sites-enabled/
```

Test the Nginx configuration:

```bash
sudo nginx -t
```

Restart Nginx:

```bash
sudo systemctl restart nginx
```

##  Request Flow

When a user accesses the application:

```text
User
  │
  ▼
AWS EC2 Public IP
  │
  ▼
Nginx :80
  │
  ▼
Gunicorn
  │
  ▼
Django Application
  │
  ▼
Response
```

Nginx handles incoming HTTP requests and forwards dynamic requests to Gunicorn. Gunicorn then communicates with the Django application and returns the response through Nginx to the user.

## 📂 Project Structure

```text
django-project/
│
├── manage.py
├── requirements.txt
├── Dockerfile
│
├── <django_project>/
│   ├── settings.py
│   ├── urls.py
│   ├── wsgi.py
│   └── ...
│
├── templates/
├── static/
├── staticfiles/
├── media/
└── README.md
```

## Verification

The deployment was verified by checking:

```bash
sudo systemctl status gunicorn
sudo systemctl status nginx
```

Nginx configuration:

```bash
sudo nginx -t
```

The Django application was then accessed through the EC2 public IP:

```text
http://<EC2-PUBLIC-IP>
```

##  Key Learning Outcomes

Through this project, I gained practical exposure to:

* Launching and configuring an AWS EC2 instance
* Connecting to Linux servers using SSH
* Deploying a Django application on AWS
* Working with Python virtual environments
* Configuring Django for production
* Using Gunicorn as a WSGI application server
* Configuring Nginx as a reverse proxy
* Serving Django static and media files
* Creating and managing Linux systemd services
* Troubleshooting deployment and server configuration issues
* Using Git and GitHub for project management

## Project Objective

The objective of this project was to understand the **end-to-end deployment of a Django application on AWS EC2** and gain practical experience with commonly used production deployment components such as **Linux, Gunicorn, Nginx, Git, and AWS EC2**.

## 👤 Author

**Sruthi**

AWS & DevOps | Cloud | Linux | Django | Cybersecurity

---
