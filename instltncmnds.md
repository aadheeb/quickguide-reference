**php installation Methods**

- `sudo apt update && sudo apt upgrade`

- `sudo apt install software-properties-common`

- `sudo add-apt-repository ppa:ondrej/php -y`

- `sudo apt install php7.4`(We can specify version here)

- For common extrension we can use
 	``` 
 	sudo apt install php7.4 php7.4-cli php7.4-curl php7.4-bz2 php7.4-bcmath php7.4-gmp php7.4-json php7.4-xmlrpc php7.4-common php7.4-imap php7.4-xsl  php7.4-fpm php7.4-mbstring php7.4-zip php7.4-gd php7.4-intl php7.4-mysql php7.4-soap php7.4-xml
 	```


**Installing Nginx**

- `sudo apt update`

- `sudo apt install nginx`


**Installing Apache**

- `sudo apt update`

- `sudo apt install apache2`


**Setting up Certbot**

- `https://certbot.eff.org/`


**MySql Installation Method**

- `sudo apt update`

- `sudo apt install mysql-server`

- `sudo systemctl start mysql.service`

- Step 2 — Configuring MySQL

	- `sudo mysql`

	- `ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'password';`

	- `exit`

	Following that, you can run the mysql_secure_installation script without issue.

	- `mysql -u root -p`

	Then go back to using the default authentication method using this command:

	- `ALTER USER 'root'@'localhost' IDENTIFIED WITH auth_socket;`

	This will mean that you can once again connect to MySQL as your root user using the sudo mysql command.


	Run the security script with `sudo`:

	- `sudo mysql_secure_installation`

- Step 3 — Creating a Dedicated MySQL User and Granting Privileges

	- `mysql -u root -p`

	- `CREATE USER 'sammy'@'localhost' IDENTIFIED BY 'password';`

	- `GRANT ALL PRIVILEGES ON *.* TO 'sammy'@'localhost' WITH GRANT OPTION;`

	Then,Flush privileges

	- `FLUSH PRIVILEGES;`

- Step 4 — Testing MySQL
	
	- `systemctl status mysql.service`