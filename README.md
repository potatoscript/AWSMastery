Here's the translation to Japanese along with the details on setting up Gunicorn:

---

## AWS EC2にDjangoプロジェクトをデプロイする

■ EC2はElastic Computeです。これは仮想サーバーです。

■ AWSは12ヶ月間の無料利用枠を提供しており、以下のサービスが無料で利用できます。

<img src="https://github.com/potatoscript/MyDocuments/blob/main/Free Tier.png?raw=true" width="400" height="200" />

### 1. **EC2インスタンスの起動**

1. **AWS Management Consoleにログイン:**
   - EC2ダッシュボードに移動します。

   <img src="https://github.com/potatoscript/MyDocuments/blob/main/EC2search.png?raw=true" width="550" height="200" />
   
2. **インスタンスを起動:**
   - 「Launch Instance」をクリックします。
   - <img src="https://github.com/potatoscript/MyDocuments/blob/main/Resources.png?raw=true" width="550" height="200" />
   - <img src="https://github.com/potatoscript/MyDocuments/blob/main/Launch instances.png?raw=true" width="550" height="200" />
   - インスタンスの名前とタグを設定します。
   - 「Ubuntu Server 20.04 LTS」などのAmazon Machine Image（AMI）を選択します。AMIはEC2で実行したいオペレーティングシステムです。
   - <img src="https://github.com/potatoscript/MyDocuments/blob/main/AMI.png?raw=true" width="400" height="200" />
   - インスタンスタイプ（例：無料利用枠対象のt2.micro）を選択します。
     - これは監視タイプのメモリで、仮想CPU1つを提供します。EC2の最も重要な構成です。
     - <img src="https://github.com/potatoscript/MyDocuments/blob/main/Instance Type.png?raw=true" width="550" height="200" />
   - インスタンスの詳細を設定し、ストレージを追加し、必要に応じてタグを追加します。
   - SSH（ポート22）とHTTP（ポート80）トラフィックを許可するようにセキュリティグループを構成します。
   - インスタンスを確認して起動します。
   - キーペア（.pemファイル）をダウンロードします（まだ持っていない場合）。
       - キーペアはサーバーに接続するための認証に使用されます。
       - <img src="https://github.com/potatoscript/MyDocuments/blob/main/Key%20pair.png?raw=true" width="600" height="200" />
       - <img src="https://github.com/potatoscript/MyDocuments/blob/main/Create%20key%20pair.png?raw=true" width="250" height="200" />
       - .pemファイルはUbuntuでSSH認証に使用されます。<br>
         Windowsユーザーの場合、Git Bashをダウンロードする必要があります。これにより、SSHおよびLinuxコマンドが使用可能になります。<br>
         .pemファイルは認証を簡単にします。
       - キーペアファイルは一度だけダウンロードされます。<br>
         このファイルを削除したり紛失したりすると、インスタンスに再接続できなくなります。
   - セキュリティグループロールを追加します。
   - <img src="https://github.com/potatoscript/MyDocuments/blob/main/add security group role.png?raw=true" width="450" height="200" />
   - ポート範囲を追加します。
   - <img src="https://github.com/potatoscript/MyDocuments/blob/main/port range.png?raw=true" width="450" height="200" />
   - インスタンスの起動
     -  <img src="https://github.com/potatoscript/MyDocuments/blob/main/After launch instances.png?raw=true" width="450" height="200" />
   - 次にインスタンスIDをクリックすると、次のページに移動します。
   - <img src="https://github.com/potatoscript/MyDocuments/blob/main/Instance ID.png?raw=true" width="450" height="200" />
     - これがあなたのウェブサイトがホストされるアドレスです。

### 2. **EC2インスタンスに接続する**
<img src="https://github.com/potatoscript/MyDocuments/blob/main/SSH client.png?raw=true" width="450" height="200" />
<img src="https://github.com/potatoscript/MyDocuments/blob/main/change permission.png?raw=true" width="450" height="200" />
1. **ターミナルを開く:**
   ```sh
   ssh -i /path/to/your-key-pair.pem ubuntu@your-ec2-public-dns
   ```
   このコマンドはインスタンスに接続します。
   <img src="https://github.com/potatoscript/MyDocuments/blob/main/connected to EC2 instance.png?raw=true" width="550" height="300" />

### 3. **環境を設定する**

- EC2を初めて起動する際に、次のコマンドを入力します。
   ```sh
    sudo su
   ```
- 次に、すべてのパッケージを更新するには、次のコマンドを使用します。
   ```sh
    yum update
   ```

1. **パッケージリストを更新する:**
   ```sh
   sudo apt update
   ```

2. **必要なパッケージをインストールする:**
   ```sh
   sudo apt install python3-pip python3-dev libpq-dev nginx curl
   ```

3. **virtualenvをインストールする:**
   ```sh
   sudo pip3 install virtualenv
   ```

4. **仮想環境を作成する:**
   ```sh
   mkdir ~/myproject
   cd ~/myproject
   virtualenv venv
   source venv/bin/activate
   ```

### 4. **Djangoプロジェクトをデプロイする**

1. **Djangoプロジェクトをクローンする:**
   ```sh
   git clone your-django-project-repo
   cd your-django-project
   ```

2. **依存関係をインストールする:**
   ```sh
   pip install -r requirements.txt
   ```

3. **Django設定を構成する:**
   - `settings.py`の`ALLOWED_HOSTS`を、EC2インスタンスの公開DNSを含むように更新します。
   - RDSのようなマネージドデータベースサービスを使用する場合、データベース設定を構成します。

4. **静的ファイルを収集する:**
   ```sh
   python manage.py collectstatic
   ```

5. **マイグレーションを適用する:**
   ```sh
   python manage.py migrate
   ```

### 5. **Gunicornの設定**

1. **Gunicornをインストールする:**
   ```sh
   pip install gunicorn
   ```

2. **Gunicornのsystemdサービスファイルを作成する:**
   ```sh
   sudo nano /etc/systemd/system/gunicorn.service
   ```

   次の内容を追加します:
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

3. **Gunicornを開始および有効化する:**
   ```sh
   sudo systemctl start gunicorn
   sudo systemctl enable gunicorn
   ```

### 6. **Nginxの設定**

1. **デフォルトのNginx設定を削除する:**
   ```sh
   sudo rm /etc/nginx/sites-enabled/default
   ```

2. **新しいNginx設定を作成する:**
   ```sh
   sudo nano /etc/nginx/sites-available/your-django-project
   ```

   次の内容を追加します:
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

3. **新しいNginx設定を有効にする:**
   ```sh
   sudo ln -s

以下は、Gunicornの設定に関する日本語での詳細な説明です。

### Gunicornの設定

Gunicorn (Green Unicorn) は、PythonのWSGI HTTPサーバーです。DjangoのようなPython Webアプリケーションを実行するのに非常に適しています。以下にGunicornの設定手順を説明します。

1. **Gunicornのインストール:**
   ```sh
   pip install gunicorn
   ```

2. **Gunicornのsystemdサービスファイルを作成する:**
   まず、`gunicorn.service`というsystemdサービスファイルを作成します。
   ```sh
   sudo nano /etc/systemd/system/gunicorn.service
   ```

   次の内容を追加します:
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

   **説明:**
   - `[Unit]` セクションでは、このサービスがネットワークが利用可能になった後に起動することを指定しています。
   - `[Service]` セクションでは、Gunicornが起動するための設定を行います。`User`と`Group`は、サービスが実行されるユーザーとグループを指定します。`WorkingDirectory`はDjangoプロジェクトのディレクトリを指定し、`ExecStart`はGunicornを起動するコマンドを指定します。
   - `[Install]` セクションでは、このサービスがマルチユーザーモードで有効にされることを指定しています。

3. **Gunicornを開始および有効化する:**
   次に、Gunicornサービスを開始し、ブート時に自動的に起動するように有効化します。
   ```sh
   sudo systemctl start gunicorn
   sudo systemctl enable gunicorn
   ```

これで、GunicornがDjangoプロジェクトを処理するために設定されました。次に、Nginxを設定して、Gunicornからのリクエストを処理するようにします。




## Deploying a Django project to AWS EC2 

■ EC2 is the Elastic Compute. It is a virtual Server

■ AWS isproviding the free tier for the 12 months and you will get the following for free

<img src="https://github.com/potatoscript/MyDocuments/blob/main/Free Tier.png?raw=true" width="400" height="200" />

### 1. **Launch an EC2 Instance**

1. **Log in to AWS Management Console:**
   - Navigate to the EC2 Dashboard.

   <img src="https://github.com/potatoscript/MyDocuments/blob/main/EC2search.png?raw=true" width="550" height="200" />
   
2. **Launch an Instance:**
   - Click on "Launch Instance."
   - <img src="https://github.com/potatoscript/MyDocuments/blob/main/Resources.png?raw=true" width="550" height="200" />
   - <img src="https://github.com/potatoscript/MyDocuments/blob/main/Launch instances.png?raw=true" width="550" height="200" />
   - Set the name and Tag for your instance
   - Choose an Amazon Machine Image (AMI), such as "Ubuntu Server 20.04 LTS.", AMI is operating system that you want to run into your EC2
   - <img src="https://github.com/potatoscript/MyDocuments/blob/main/AMI.png?raw=true" width="400" height="200" />
   - Select an instance type (e.g., t2.micro for free tier eligibility).
     - is a watched type memory one virtual CPU you will get and it is the most imporant configuration into the EC2
     - <img src="https://github.com/potatoscript/MyDocuments/blob/main/Instance Type.png?raw=true" width="550" height="200" />
   - Configure instance details, add storage, and add tags if necessary.
   - Configure the security group to allow SSH (port 22) and HTTP (port 80) traffic.
   - Review and launch the instance.
   - Download the key pair (.pem file) if you don't have one already.
       - A key pair is used for authentication to connect to your server.
       - <img src="https://github.com/potatoscript/MyDocuments/blob/main/Key%20pair.png?raw=true" width="600" height="200" />
       - <img src="https://github.com/potatoscript/MyDocuments/blob/main/Create%20key%20pair.png?raw=true" width="250" height="200" />
       - The .pem file is used with SSH for authentication on Ubuntu. <br>
         If you are a Windows user, you need to download Git Bash, which allows you to use SSH and Linux commands. <br>
         The .pem file makes authentication easier.
       - The key pair file will only be downloaded once.<br>
         If you delete or misplace this file, you will not be able to connect to your instance again.
   - Add security group role
   - <img src="https://github.com/potatoscript/MyDocuments/blob/main/add security group role.png?raw=true" width="450" height="200" />
   - Add the port range
   - <img src="https://github.com/potatoscript/MyDocuments/blob/main/port range.png?raw=true" width="450" height="200" />
   - Launching instance
     -  <img src="https://github.com/potatoscript/MyDocuments/blob/main/After launch instances.png?raw=true" width="450" height="200" />
   - Next click on the Instance ID and it will bring you to the following page
   - <img src="https://github.com/potatoscript/MyDocuments/blob/main/Instance ID.png?raw=true" width="450" height="200" />
     - This will be the address that your web site will be host
          

### 2. **Connect to Your EC2 Instance**
<img src="https://github.com/potatoscript/MyDocuments/blob/main/SSH client.png?raw=true" width="450" height="200" />
<img src="https://github.com/potatoscript/MyDocuments/blob/main/change permission.png?raw=true" width="450" height="200" />
1. **Open a terminal:**
   ```sh
   ssh -i /path/to/your-key-pair.pem ubuntu@your-ec2-public-dns
   ```<br>
   this command is going to connect with your instance
   <img src="https://github.com/potatoscript/MyDocuments/blob/main/connected to EC2 instance.png?raw=true" width="550" height="300" />

### 3. **Set Up the Environment**

- When you are going to boot your EC2 first time then just type command 
   ```sh
    sudo su
   ```
- Next update all the packages use the command
   ```sh
    yum update
   ```

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
