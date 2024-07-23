Deploying a Django project to AWS EC2 involves several steps, including setting up the EC2 instance, configuring the environment, and deploying the Django application. Here is a step-by-step guide to help you through the process:

### 1. **Launch an EC2 Instance**

1. **Log in to AWS Management Console:**
   - Navigate to the EC2 Dashboard.
   
2. **Launch an Instance:**
   - Click on "Launch Instance."
   - Choose an Amazon Machine Image (AMI), such as "Ubuntu Server 20.04 LTS."
   - Select an instance type (e.g., t2.micro for free tier eligibility).
   - Configure instance details, add storage, and add tags if necessary.
   - Configure the security group to allow SSH (port 22) and HTTP (port 80) traffic.
   - Review and launch the instance.
   - Download the key pair (.pem file) if you don't have one already.

### 2. **Connect to Your EC2 Instance**

1. **Open a terminal:**
   ```sh
   ssh -i /path/to/your-key-pair.pem ubuntu@your-ec2-public-dns
   ```

### 3. **Set Up the Environment**

1. **Update the package lists:**
   ```sh
   sudo apt update
   ```

2. **Install necessary packages:**
   ```sh
   sudo apt install python3-pip python3-dev libpq-dev nginx curl
   ```

3. **Install virtualenv:**
   ```sh
   sudo pip3 install virtualenv
   ```

4. **Create a virtual environment:**
   ```sh
   mkdir ~/myproject
   cd ~/myproject
   virtualenv venv
   source venv/bin/activate
   ```

### 4. **Deploy Your Django Project**

1. **Clone your Django project:**
   ```sh
   git clone your-django-project-repo
   cd your-django-project
   ```

2. **Install dependencies:**
   ```sh
   pip install -r requirements.txt
   ```

3. **Configure your Django settings:**
   - Update `ALLOWED_HOSTS` in `settings.py` to include your EC2 instance's public DNS.
   - Configure database settings if using a managed database service like RDS.

4. **Collect static files:**
   ```sh
   python manage.py collectstatic
   ```

5. **Apply migrations:**
   ```sh
   python manage.py migrate
   ```

### 5. **Configure Gunicorn**

1. **Install Gunicorn:**
   ```sh
   pip install gunicorn
   ```

2. **Create a Gunicorn systemd service file:**
   ```sh
   sudo nano /etc/systemd/system/gunicorn.service
   ```

   Add the following content:
   ```ini
   [Unit]
   Description=gunicorn daemon
   After=network.target

   [Service]
   User=ubuntu
   Group=www-data
   WorkingDirectory=/home/ubuntu/myproject/your-django-project
   ExecStart=/home/ubuntu/myproject/venv/bin/gunicorn --access-logfile - --workers 3 --bind unix:/home/ubuntu/myproject/your-django-project.sock your_django_project.wsgi:application

   [Install]
   WantedBy=multi-user.target
   ```

3. **Start and enable Gunicorn:**
   ```sh
   sudo systemctl start gunicorn
   sudo systemctl enable gunicorn
   ```

### 6. **Configure Nginx**

1. **Remove the default Nginx configuration:**
   ```sh
   sudo rm /etc/nginx/sites-enabled/default
   ```

2. **Create a new Nginx configuration:**
   ```sh
   sudo nano /etc/nginx/sites-available/your-django-project
   ```

   Add the following content:
   ```nginx
   server {
       listen 80;
       server_name your-ec2-public-dns;

       location = /favicon.ico { access_log off; log_not_found off; }
       location /static/ {
           root /home/ubuntu/myproject/your-django-project;
       }

       location / {
           include proxy_params;
           proxy_pass http://unix:/home/ubuntu/myproject/your-django-project.sock;
       }
   }
   ```

3. **Enable the new Nginx configuration:**
   ```sh
   sudo ln -s /etc/nginx/sites-available/your-django-project /etc/nginx/sites-enabled
   ```

4. **Restart Nginx:**
   ```sh
   sudo systemctl restart nginx
   ```

### 7. **Access Your Application**

- Open your browser and navigate to your EC2 instance's public DNS to see your Django application live.

### Additional Considerations

- **Domain Name:** If you have a domain name, update the Nginx server block's `server_name` directive.
- **HTTPS:** Set up an SSL certificate using Let's Encrypt for secure HTTPS connections.
