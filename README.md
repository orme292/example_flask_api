# A Sample API with Flask

An example API built with Python using Flask (web framework), Marshmallow (data validation), and SQLAlchemy (database management)

You can get this running on your very own VM (like a Linode). You'll need:

* A public IP
* Debian 9 (or experience with some other distro)
* 30 Minutes

### Steps:

##### Install some python
```
sudo apt update && sudo apt upgrade
sudo apt install git nginx python3 python3-pip python-virtualenv nginx supervisor
```

##### Install some database
```
cd /tmp
wget https://dev.mysql.com/get/mysql-apt-config_0.8.13-1_all.deb
sudo dpkg -i mysql-apt-config*
```
*Select MySQL Server & Cluster and switch version to mysql-5.7*
```
sudo apt update
sudo apt install mysql-server
mysql_secure_installation
```
Create the **api-logger** database (note the backticks):
```
mysql -u root -p
CREATE DATABASE `api-logger`;
quit
```

##### Create an environment for your API
```
cd ~
mkdir api-root
cd ~/api-root
virtualenv -p python3 api-env
source api-env/bin/activate
```

##### Clone some API
```
cd ~/api-root/
git clone https://github.com/orme292/example_flask_api.git
```

##### Create a creds file
```
touch ~/api-root/example_flask_api/dbcreds.yaml
nano ~/api-root/example_flask_api/dbcreds.yaml
```
Add the following to the file and then save with Ctrl+O
```
db_user:  root
db_pass:  <password>
db_host:  127.0.0.1
db_port:  3036
db_name:  api-logger
```

##### Install the API dependencies
```
cd ~/api-root/example_flask_api/
pip3 install -r requirements.txt
```

##### Install Gunicorn
```
pip3 install gunicorn
```

##### Configure supervisor
```
sudo touch /etc/supervisor/conf.d/example_api.conf
sudo nano /etc/supervisor/conf.d/example_api.conf
```
Add the following lines to the file (replace username) and then save with Ctrl+O
```
[program:example_api]
directory=/home/<user>/api-root/example_flask_api/api
command=/home/<user>/api-root/api-env/bin/gunicorn --workers=4 --log-level=debug main_api:api -b localhost:8000
autostart=true
autorestart=true
stderr_logfile=/var/log/api/api.err.log
stdout_logfile=/var/log/api/api.out.log
```
Create the log files:
```
sudo mkdir /var/log/api
sudo touch /var/log/api/api.err.log
sudo touch /var/log/api/api.out.log
```
##### Start supervisor up
```
sudo supervisorctl reread
sudo service supervisor restart
sudo supervisorctl restart all
sudo supervisorctl status
```

##### Configure Nginx
```
sudo nano /etc/nginx/sites-enabled/default
```
Replace the entire Server { ... } block with the following and save with Ctrl+O:
```
server {
    listen 80;

    location / {
        proxy_pass http://127.0.0.1:8000;
    }
}
```
Check for errors and reload nginx:
```
sudo nginx -t
sudo systemctl restart nginx
```

Visit your IP address for the magic of denial!
```json
{
  "status": {
    "message": "900: Request Not Authorized.",
    "records": 0,
    "success": false
  }
}
```
To get started you need to initialize the database.

Download [Postman](https://www.getpostman.com/) to start querying the API.

All endpoints will be prefaced with your IP address for now. So, if your IP address is 257.100.2.10, then you would get data from the `events` endpoint here: `http://257.100.2.10/events`

Open Postman and create a new `POST` request to the following endpoint `/init`
Select `body` > `raw` and change type to `JSON (application/json)`. Enter the following JSON to initialize the DB:
```json
{
	"name": "init db",
	"location": "home",
	"description": "initialize the DB"
}
```
Click SEND and you should get the following back:
```json
{
    "dataset": {
        "created": "Day, DD MM YYYY HH:MM:SS TZD",
        "description": "initialize the DB",
        "id": 1,
        "location": "home",
        "name": "init db"
    },
    "status": {
        "message": "101: Created.",
        "records": 1,
        "success": true
    }
}
```

Success!

You can create new events at the `/events` endpoint, but you need to register a new account to get a token.

To register an account, send a `POST` request to the following endpoint: `/auth/register`.
Select `body` > `raw` and change type to `JSON (application/json)`.
Enter the following JSON to initialize the DB:
```json
{
	"username": "<username>",
	"email": "<email>"
}
```
Click SEND and you should get the following back:
```json
{
    "dataset": {
        "created": "Day, DD MM YYYY HH:MM:SS TZD",
        "email": "<email>",
        "id": 1,
        "role": 0,
        "token": "<token>",
        "username": "<username>"
    },
    "status": {
        "message": "101: Created.",
        "records": 1,
        "success": true
    }
}
```

Success! Take that token and add it to the `authorization` tab and change `type` to `Bearer Token` and paste your token in. Now you can create new events at the `/events/` endpoint by posting data like this:
```json
{
	"name": "test api",
	"location": "home",
	"description": "created a test event via postman"
}
```

View events
`GET` `/events`

View event by ID
`GET` `/events/<id>`

Delete event by ID
`DEL` `/events/<id>`

Update events by ID
`PATCH` `/events/<id>`
```json
{
 	"description": "eat dinner",
	"location": "france"
}
```
or
```json
{
  "name": "fight a bear"
}
