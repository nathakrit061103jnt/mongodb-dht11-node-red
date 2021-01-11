# ตอนที่ 5 ดึงข้อมูลจาก MongoDB มาเเสดงที่ Node Red Dashboard UI

## 1. Show List Data

    <ul>
       <li ng-repeat="item in msg.payload">{{item.name}}</li>
    </ul>

## 2. Show Data table view

    <!DOCTYPE html>
    <html lang="en">
            <head>
                <meta charset="utf-8">
                <meta name="viewport" content="width=device-width, initial-scale=1">
                <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/4.5.2/css/bootstrap.min.css">
                <script src="https://ajax.googleapis.com/ajax/libs/jquery/3.5.1/jquery.min.js"></script>
                <script src="https://cdnjs.cloudflare.com/ajax/libs/popper.js/1.16.0/umd/popper.min.js"></script>
                <script src="https://maxcdn.bootstrapcdn.com/bootstrap/4.5.2/js/bootstrap.min.js"></script>
            </head>
        <body>

            <div class="container">
                <h2>Product</h2>
                <table class="table">
                    <thead>
                    <tr>
                        <th>Name</th>
                        <th>Category</th>
                        <th>Price</th>
                    </tr>
                    </thead>
                    <tbody>
                    <tr ng-repeat="item in msg.payload">
                        <td>{{item.name}}</td>
                        <td>{{item.category}}</td>
                        <td>{{item.price}}</td>
                    </tr>
                    </tbody>
                </table>
            </div>

        </body>
    </html>

# ตอนที่ 6 นำข้อมูลจากNode Red บันทึกลงฐานข้อมูล MongDB บน localhost

## 1. function convertObjFucn ที่ใช้ convert ข้อมูลเป็น Json

    const data1 = msg.payload;
    const key2 = msg.key2;

    const obj = {
        "data": data1,
        "key2": key2
    }

    msg.payload = obj;

    return msg;

# ตอนที่ 7 นำข้อมูลจาก DHT11 มาบันทึกที่ฐานข้อมูล MongoDB

## 1. function การเเปลงข้อมูลจาก dht11 เป็น object เพื่อบันทึกลง MongoDB

    const humid = msg.payload.humid;
    const tempc = msg.payload.tempc;

    const d = new Date();
    const timerecord =  d.toString();

    const jsonPayload = {
        "tempc":tempc,
        "humid":humid,
        "timerecord":timerecord
    };

    msg.payload = jsonPayload;

    return msg;

# ตอนที่ 8 ใช้ Node-Red เเสดงข้อมูลจาก DHT11 ที่บันทึกลง MongoDB

## 1. Source code ที่ใช้เเสดงข้อมูล DHT11 ของ Node Dashboard template ui

    <!DOCTYPE html>
    <html lang="en">
        <head>
            <meta charset="utf-8">
            <meta name="viewport" content="width=device-width, initial-scale=1">
            <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/4.5.2/css/bootstrap.min.css">
            <script src="https://ajax.googleapis.com/ajax/libs/jquery/3.5.1/jquery.min.js"></script>
            <script src="https://cdnjs.cloudflare.com/ajax/libs/popper.js/1.16.0/umd/popper.min.js"></script>
            <script src="https://maxcdn.bootstrapcdn.com/bootstrap/4.5.2/js/bootstrap.min.js"></script>
        </head>
    <body>
    <div class="container">
        <h2>DHT11 Data</h2>
        <table class="table">
            <thead>
            <tr>
                <th>#</th>
                <th>humid</th>
                <th>tempc</th>
                <th>timerecord</th>
            </tr>
            </thead>
            <tbody>
            <tr ng-repeat="item in msg.payload">
                <td>{{$index + 1}}</td>
                <td>{{item.humid}}</td>
                <td>{{item.tempc}}</td>
                <td>{{item.timerecord}}</td>
            </tr>
            </tbody>
        </table>
    </div>

    </body>
    </html>

# ตอนที่ 11 node-red ดึงข้อมูลจาก MongoDB on Cloud && MongoDB Atlas

## 1. Source code ของ Node Dashboard template ui ที่ใช้เเสดงข้อมูลจาก MongoDB Atlas ที่อยู่บน Cloud

    <!DOCTYPE html>
    <html lang="en">
        <head>
            <meta charset="utf-8">
            <meta name="viewport" content="width=device-width, initial-scale=1">
            <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/4.5.2/css/bootstrap.min.css">
            <script src="https://ajax.googleapis.com/ajax/libs/jquery/3.5.1/jquery.min.js"></script>
            <script src="https://cdnjs.cloudflare.com/ajax/libs/popper.js/1.16.0/umd/popper.min.js"></script>
            <script src="https://maxcdn.bootstrapcdn.com/bootstrap/4.5.2/js/bootstrap.min.js"></script>
        </head>
    <body>

    <div class="container">
        <h2>DHT11 Data On Cloud</h2>
        <table class="table">
            <thead>
            <tr>
                <th>#</th>
                <th>id</th>
                <th>name</th>
            </tr>
            </thead>
            <tbody>
            <tr ng-repeat="item in msg.payload">
                <td>{{$index + 1}}</td>
                <td>{{item.id}}</td>
                <td>{{item.name}}</td>
            </tr>
            </tbody>
        </table>
    </div>

    </body>
    </html>

# ตอนที่ 14 ติดตั้งnode-red เเละตั้ง mqtt broker บน server aws ec2

## Script Command ที่ใช้ติดตั้ง

### install nodejs

    1. sudo apt update
    2. curl -sL https://deb.nodesource.com/setup_14.x | sudo bash -
    3. sudo apt -y install nodejs

### install npm

    1. sudo apt install npm

### check node version

    1. node  -v

### Node Development tools

    1. sudo apt -y install gcc g++ make

### install yarn

    1. curl -sL https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -
    2. echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee   /etc/apt/sources.list.d/yarn.list
    3. sudo apt update && sudo apt install yarn

### install node-red

    0. sudo apt-get update
    1. sudo npm install -g --unsafe-perm node-red
    2.  node-red

### Autostart on boot PM2

#### install PM2

    0. sudo apt-get update
    1. sudo npm install -g pm2
    2. pm2 start /usr/bin/node-red -- -v
    3. pm2 startup systemd

==================exmaple========================================
sudo env PATH=$PATH:/usr/bin /usr/lib/node_modules/pm2/bin/pm2 startup systemd -u ubuntu --hp /home/ubuntu
==================exmaple========================================

    4.export PM2_HOME="/root/.pm2"
    5. pm2 save

### chang password node-red

    1.  cd .node-red
    2.  node-red admin hash-pw
    3.  sudo nano settings.js
    4.  uncomment and paste hash key
    5.  Ctrl + O + Enter
    6.  Ctrl + x
    7.  sudo apt-get update

### install mqtt Broker on Ubuntu

1.  sudo apt-get install mosquitto
2.  sudo apt-get update
