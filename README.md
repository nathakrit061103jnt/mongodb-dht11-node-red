# ตอนที่ 5 ดึงข้อมูลจาก MongoDB มาเเสดงที่ Node Red Dashboard UI

## 1. Show List Data

```HTML
    <ul>
       <li ng-repeat="item in msg.payload">{{item.name}}</li>
    </ul>
```

## 2. Show Data table view

```HTML
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
```

# ตอนที่ 6 นำข้อมูลจากNode Red บันทึกลงฐานข้อมูล MongDB บน localhost

## 1. function convertObjFucn ที่ใช้ convert ข้อมูลเป็น Json

```js
const data1 = msg.payload;
const key2 = msg.key2;

const obj = {
  data: data1,
  key2: key2,
};

msg.payload = obj;

return msg;
```

# ตอนที่ 7 นำข้อมูลจาก DHT11 มาบันทึกที่ฐานข้อมูล MongoDB

## 1. function การเเปลงข้อมูลจาก dht11 เป็น object เพื่อบันทึกลง MongoDB

```js
const humid = msg.payload.humid;
const tempc = msg.payload.tempc;

const d = new Date();
const timerecord = d.toString();

const jsonPayload = {
  tempc: tempc,
  humid: humid,
  timerecord: timerecord,
};

msg.payload = jsonPayload;

return msg;
```

## 2. arduino code mqtt-dht11

```c++

   #include <ESP8266WiFi.h>
    #include <Wire.h>
    #include <PubSubClient.h>

    #include "DHT.h"

    //เลือกขา Data
    #define DHTPIN D4

    //ระบุเซนเซอร์ที่จะเชื่อมต่อ
    // Uncomment whatever type you're using!
    #define DHTTYPE DHT11 // DHT 11
    //#define DHTTYPE DHT22 // DHT 22 (AM2302), AM2321
    //#define DHTTYPE DHT21 // DHT 21 (AM2301)

    DHT dht(DHTPIN, DHTTYPE);

    //เชื่อมไวไฟ
    #define wifi_ssid "JamesNathakrit"
    #define wifi_password "*1234567"
    #define mqtt_server "54.151.203.250"
    //#define mqtt_user "user"
    //#define mqtt_password "password"

    #define humid "humid"
    #define tempc "tempc"
    // #define tempf "sensor/tempf"

    //เชื่อมต่อไวไฟ
    WiFiClient espClient;
    PubSubClient client(espClient);

    void setup()
    {
    Serial.begin(115200);
    dht.begin();
    setup_wifi();
    client.setServer(mqtt_server, 1883);
    }

    String macToStr(const uint8_t *mac)
    {
    String result;
    for (int i = 0; i < 6; ++i)
    {
        result += String(mac[i], 16);
        if (i < 5)
        result += ':';
    }
    return result;
    }

    void setup_wifi()
    {
    delay(10);
    // We start by connecting to a WiFi network
    Serial.println();
    Serial.print("Connecting to ");
    Serial.println(wifi_ssid);

    WiFi.begin(wifi_ssid, wifi_password);

    while (WiFi.status() != WL_CONNECTED)
    {
        delay(500);
        Serial.print(".");
    }

    Serial.println("");
    Serial.println("WiFi connected");
    Serial.println("IP address: ");
    Serial.println(WiFi.localIP());
    }

    void reconnect()
    {
    // Loop until we're reconnected
    while (!client.connected())
    {
        Serial.print("Attempting MQTT connection...");

        // Generate client name based on MAC address and last 8 bits of microsecond counter
        String clientName;
        clientName += "esp8266-";
        uint8_t mac[6];
        WiFi.macAddress(mac);
        clientName += macToStr(mac);
        clientName += "-";
        clientName += String(micros() & 0xff, 16);
        Serial.print("Connecting to ");
        Serial.print(mqtt_server);
        Serial.print(" as ");
        Serial.println(clientName);

        // Attempt to connect
        // If you do not want to use a username and password, change next line to
        if (client.connect((char *)clientName.c_str()))
        {
        //if (client.connect((char*) clientName.c_str()), mqtt_user, mqtt_password)) {
        Serial.println("connected");
        }
        else
        {
        Serial.print("failed, rc=");
        Serial.print(client.state());
        Serial.println(" try again in 5 seconds");
        // Wait 5 seconds before retrying
        delay(5000);
        }
    }
    }

    void loop()
    {

    if (!client.connected())
    {
        reconnect();
    }
    client.loop();

    // Wait a few seconds between measurements.
    delay(10000);

    float h = dht.readHumidity();
    // Read temperature as Celsius (the default)
    float t = dht.readTemperature();
    // Read temperature as Fahrenheit (isFahrenheit = true)
    float f = dht.readTemperature(true);

    // Check if any reads failed and exit early (to try again).
    if (isnan(h) || isnan(t) || isnan(f))
    {
        Serial.println("Failed to read from DHT sensor!");
        return;
    }

    //เเสดงค่าเเละส่งค่าไฟที่ Server
    Serial.print("Temperature:");
    Serial.println(String(t).c_str());
    client.publish(tempc, String(t).c_str(), true);

    Serial.print("Humidity:");
    Serial.println(String(h).c_str());
    client.publish(humid, String(h).c_str(), true);
    }

```

# ตอนที่ 8 ใช้ Node-Red เเสดงข้อมูลจาก DHT11 ที่บันทึกลง MongoDB

## 1. Source code ที่ใช้เเสดงข้อมูล DHT11 ของ Node Dashboard template ui

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <link
      rel="stylesheet"
      href="https://maxcdn.bootstrapcdn.com/bootstrap/4.5.2/css/bootstrap.min.css"
    />
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
```

# ตอนที่ 11 node-red ดึงข้อมูลจาก MongoDB on Cloud && MongoDB Atlas

## 1. Source code ของ Node Dashboard template ui ที่ใช้เเสดงข้อมูลจาก MongoDB Atlas ที่อยู่บน Cloud

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <link
      rel="stylesheet"
      href="https://maxcdn.bootstrapcdn.com/bootstrap/4.5.2/css/bootstrap.min.css"
    />
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
```

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

##### exmaple อันนี้เป็นตัวอย่างของผม ให้เพื่อเพื่อน ๆ เปลี่ยนเป็น path ของตัวเองด้วยนะครับ ==========

    sudo env PATH=$PATH:/usr/bin /usr/lib/node_modules/pm2/bin/pm2 startup systemd -u ubuntu --hp /home/ubuntu

##### exmaple ====================

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

# ep 16 สอน ติดตั้ง mqtt broker & node-red บน Docker-Compose

## link github code :

https://github.com/nathakrit061103jnt/docker-compose-mqtt-broker

## Install Docker Engine on Ubuntu

    1) sudo apt-get update
    2) sudo apt-get install \
        apt-transport-https \
        ca-certificates \
        curl \
        gnupg-agent \
        software-properties-common

    3)  curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add
    4)  sudo apt-key fingerprint 0EBFCD88
    5)  sudo add-apt-repository \
        "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
        $(lsb_release -cs) \
        stable"
    6)  sudo apt-get update
    7)  sudo apt-get install docker-ce docker-ce-cli containerd.io
    8)  apt-cache madison docker-ce

## 2. Install Docker-Compose

    0)  sudo apt-get update
    1)  sudo curl -L "https://github.com/docker/compose/releases/download/1.27.4/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
    2)  sudo chmod +x /usr/local/bin/docker-compose
    3)  sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
    4)  sudo docker-compose --version

## 3. Install Git

    0)  sudo apt-get update
    1)  sudo apt-get install git

## 4. Clone Project & docker-compose.yml file

    1) sudo git clone https://github.com/nathakrit061103jnt/docker-compose-mqtt-broker.git docker-compose-mqtt-broker
    2) cd docker-compose-mqtt-broker
    3) sudo docker-compose up -d

## Node-Red Port 1880

## Mqtt Broker eclipse-mosquitto Port 1883

## portainer Port 9000
