# Instruction how to create an Backend AMI and configure it

## Launch an instance:

Launch an instance with Amazon Linux 2 AMI. Choose security group with port 22, 80 opened (app is working on 80 port and we would need 22 port to ssh into instance and make app work). Then choose key which you have and select IAM role for this instance which will allow instance to fetch credentials securely from AWS Secrets Manager (create if you don’t have that role).

## SSH into instance:

Clone repository with `git clone ssh_clone_url_for_the_repo`  or scp repo from your local machine with:

```html
scp -i location/of/key -r reviews-app-23c-team2 [ec2-user@](mailto:ec2-user@54.221.132.19)IP_ADRESS:/home/ec2-user
```

Than in ec2 install the required packages (for application to work):

```html
sudo pip3 install -r requirements.txt
```

In addition install mysql (we would need it when connecting to RDS):

```html
sudo yum install mariadb -y
```

## Create a systemd service:

Run: 

```html
sudo yum install -y systemd
```

Create a service:

```html
sudo vi /etc/systemd/system/reviews-api.service
```

Paste this code inside:

```
[Unit]
Description=My service
After=multi-user.target

[Service]
Type=simple
Restart=always
ExecStart=/usr/bin/python3 /home/ec2-user/reviews-app-23c-team2/api-backend/app.py

[Install]
WantedBy=multi-user.target
```

Reload daemons:

```html
sudo systemctl daemon-reload
```

Enable your service:

```html
sudo systemctl enable reviews-api.service
```

Start your service:

```html
sudo systemctl start reviews-api.service
```

## Test if application is working:

To test it you need to create Secret in Secret Manager with name exactly like in the code (`your_secret_name_here`) or to change that name in app.py with `sudo vi app.py` . Also you would need keys: DB_Username, DB_Password, DB_Endpoint and DB_Name and some values for them.

You can then check status of application with:

```html
sudo systemctl status reviews-api.service
```

If everything is working fine, the output should look like this:

![Screenshot 2023-10-16 at 20.40.50.png](Instruction%20how%20to%20create%20an%20Backend%20AMI%20and%20confi%206cf1ef03063b498ea140690d4841a9d0/Screenshot_2023-10-16_at_20.40.50.png)

## Connect to RDS:

In order for app to work with RDS instance you should know such information as DB Username, DB Password, DB Endpoint and DB Name. So you should have access to this information in Secret Manager. Once you have access you can run some commands in CLI like this:

```html
curl -X POST -H "Content-Type: application/json" -d '{"product_name": "Product Name"}' [http://localhost:80/product](http://localhost/product)
```

Or this:

```html
curl -X GET [http://localhost:80/product](http://localhost/product)
```

If everything works properly then you should have output with response.

## Creating AMI from EC2 instance:

If you tested that your app is working well you can save your AMI. Just stop instance, wait till it will be completely stoped and then press Actions → Image and templates → Create image

Name your Image and create it. That’s all.