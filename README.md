## ตอนที่ 5 ดึงข้อมูลจาก MongoDB มาเเสดงที่ Node Red Dashboard UI

# 1. Show List Data

    <ul>
       <li ng-repeat="item in msg.payload">{{item.name}}</li>
    </ul>

# 2. Show Data table view

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

## ตอนที่ 6 นำข้อมูลจากNode Red บันทึกลงฐานข้อมูล MongDB บน localhost

# 1. function convertObjFucn ที่ใช้ convert ข้อมูลเป็น Json

    const data1 = msg.payload;
    const key2 = msg.key2;

    const obj = {
        "data": data1,
        "key2": key2
    }

    msg.payload = obj;

    return msg;
