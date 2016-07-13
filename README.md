# Setting up gophish

Thanks to the developers this project: https://github.com/gophish/gophish


Download gophish and extract to /opt:
Look for your release at https://github.com/gophish/gophish/releases/
```
wget https://github.com/gophish/gophish/releases/download/v0.1.2/gophish_linux_64bit.zip
unzip gophish_linux_64bit.zip -d /opt
cd /opt/gophish_linux_64bit
```

Create self signed cert to enable https for admin web interface
```
openssl req -newkey rsa:2048 -nodes -keyout gophish.key -x509 -days 365 -out gophish.crt
```

Edit config.json file to enable ssl. I also wanted to listen on 0.0.0.0 rather than localhost so I can access from other systems.
```
{
	"admin_server" : {
		"listen_url" : "0.0.0.0:3333",
		"use_tls" : true,
		"cert_path" : "gophish.crt",
		"key_path" : "gophish.key"
	},
	"phish_server" : {
		"listen_url" : "0.0.0.0:80",
		"use_tls" : false,
		"cert_path" : "example.crt",
		"key_path": "example.key"
	},
	"smtp" : {
		"host" : "smtp.example.com:25",
		"user" : "username",
		"pass" : "password"
	},
	"db_path" : "gophish.db",
	"migrations_path" : "db/migrations/"
}
```

Add gophish user
```
useradd gophish
```

As root, create log file path and change ownership to gophish user. Change permisisons on /opt/gophish_linux_64bit/
```
mkdir /var/log/gophish
chown gophish.gophish /var/log/gophish
chown -R gophish.gophish /opt/gophish_linux_64bit/
```
Give privileges to the gophish binary to open port 80
```
setcap 'cap_net_bind_service=+ep' /opt/gophish_linux_64bit/gophish
```
create /usr/lib/systemd/system/gophish.service 
```
[Unit]
Description=Gophish is an open-source phishing toolkit 
Documentation=https://getgophish.com/documentation/
After=network.target

[Service]
WorkingDirectory='/opt/gophish_linux_64bit/'
User=gophish
Environment='STDOUT=/var/log/gophish/gophish.log'
Environment='STDERR=/var/log/gophish/gophish.log'
ExecStart=/bin/sh -c "/opt/gophish_linux_64bit/gophish >>${STDOUT} 2>>${STDERR}"
ExecReload=/bin/kill -HUP $MAINPID
Restart=on-failure
KillMode=process

[Install]
WantedBy=multi-user.target
Alias=gophish.service
```

Start and enable gophish service
```
systemctl start gophish
systemctl enable gophish
```

Visit https://localhost:3333 login with admin/gophish. Change password immediately
