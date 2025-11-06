# Lăng Nguyễn Minh Lượng - K225480106044
# K58ktp - Môn phát triển ứng dụng trên nền web
# Nội dung bài tập 3:
Yêu cầu     : LẬP TRÌNH ỨNG DỤNG WEB trên nền linux
1. Cài đặt môi trường linux: SV chọn 1 trong các phương án
 - enable wsl: cài đặt docker desktop
 - enable wsl: cài đặt ubuntu
 - sử dụng Hyper-V: cài đặt ubuntu
 - sử dụng VMware : cài đặt ubuntu
 - sử dụng Virtual Box: cài đặt ubuntu
2. Cài đặt Docker (nếu dùng docker desktop trên windows thì nó có ngay)
3. Sử dụng 1 file docker-compose.yml để cài đặt các docker container sau: 
   mariadb (3306), phpmyadmin (8080), nodered/node-red (1880), influxdb (8086), grafana/grafana (3000), nginx (80,443)
4. Lập trình web frontend+backend:
 SV chọn 1 trong các web sau:

 4.1 Web thương mại điện tử
 - Tạo web dạng Single Page Application (SPA), chỉ gồm 1 file index.html, toàn bộ giao diện do javascript sinh động.
 - Có tính năng login, lưu phiên đăng nhập vào cookie và session
   Thông tin login lưu trong cơ sở dữ liệu của mariadb, được dev quản trị bằng phpmyadmin, yêu cầu sử dụng mã hoá khi gửi login.
   Chỉ cần login 1 lần, bao giờ logout thì mới phải login lại.
 - Có tính năng liệt kê các sản phẩm bán chạy ra trang chủ
 - Có tính năng liệt kê các nhóm sản phẩm
 - Có tính năng liệt kê sản phẩm theo nhóm
 - Có tính năng tìm kiếm sản phẩm
 - Có tính năng chọn sản phẩm (đưa sản phẩm vào giỏ hàng, thay đổi số lượng sản phẩm trong giỏ, cập nhật tổng tiền)
 - Có tính năng đặt hàng, nhập thông tin giao hàng => được 1 đơn hàng.
 - Có tính năng dành cho admin: Thống kê xem có bao nhiêu đơn hàng, call để xác nhận và cập nhật thông tin đơn hàng. chuyển cho bộ phận đóng gói, gửi bưu điện, cập nhật mã COD, tình trạng giao hàng, huỷ hàng,...
 - Có tính năng dành cho admin: biểu đồ thống kê số lượng mặt hàng bán được trong từng ngày. (sử dụng grafana)
 - backend: sử dụng nodered xử lý request gửi lên từ javascript, phản hồi về json.

 4.2 Web IOT: Giám sát dữ liệu IOT.
 - Tạo web dạng Single Page Application (SPA), chỉ gồm 1 file index.html, toàn bộ giao diện do javascript sinh động.
 - Có tính năng login, lưu phiên đăng nhập vào cookie và session
   Thông tin login lưu trong cơ sở dữ liệu của mariadb, được dev quản trị bằng phpmyadmin, yêu cầu sử dụng mã hoá khi gửi login.
   Chỉ cần login 1 lần, bao giờ logout thì mới phải login lại.
 - hiển thị giá trị mới nhất của các thông số đang giám sát, khi click vào thì hiển thị đồ thị lịch sử quá trình thay đổi (gọi grafana iframe để hiển thị)
 - backend: Sử dụng nodered để đọc dữ liệu từ các cảm biến (có thể dùng api online để lấy dữ liệu theo giời gian thực), 
   nodered sẽ lưu dữ liệu mới nhất (dạng update) vào cơ sở dữ liệu mariadb (sử dụng phpmyadmin để tạp table và quản trị lần đầu)
   nodered sẽ lưu dữ liệu (insert) vào influxdb để lưu giá trị lịch sử, để cho grafana dùng để hiển thị biểu đồ.
5. Nginx làm web-server
 - Cấu hình nginx để chạy được website qua url http://fullname.com  (thay fullname bằng chuỗi ko dấu viết liền tên của bạn)
 - Cấu hình nginx để http://fullname.com/nodered truy cập vào nodered qua cổng 80, (dù nodered đang chạy ở port 1880)
 - Cấu hình nginx để http://fullname.com/grafana truy cập vào grafana qua cổng 80, (dù grafana đang chạy ở port 3000)
# -----BÀI LÀM-----
Mở cmd chạy lệnh wsl --install

<img width="1111" height="625" alt="image" src="https://github.com/user-attachments/assets/d3b3879f-e060-4684-88fa-907baa027f22" />

tải docker desktop => vào cài đặt bật ubuntu => ally

<img width="1235" height="712" alt="image" src="https://github.com/user-attachments/assets/a0acd42e-9e2b-4eb0-a4f2-25c4fdae6fc4" />
sau khi tạo user và pw thì được giao diện ubuntu như này 

<img width="1108" height="622" alt="image" src="https://github.com/user-attachments/assets/40738b93-8720-4243-aca2-210275855cb2" />

chạy lệnh 
nano docker-compose.yml
```
version: "3.8"

services:
  # =============================
  # 🗄️ MariaDB Database
  # =============================
  mariadb:
    image: mariadb:latest
    container_name: mariadb
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: 123456
      MYSQL_DATABASE: iotdb
      MYSQL_USER: root
      MYSQL_PASSWORD: 123456
    ports:
      - "3306:3306"
    volumes:
      - ./db:/var/lib/mysql
    networks:
      - backend

  # =============================
  # 🧮 phpMyAdmin (DB GUI)
  # =============================
  phpmyadmin:
    image: phpmyadmin:latest
    container_name: phpmyadmin
    restart: always
    environment:
      PMA_HOST: mariadb
      PMA_PORT: 3306
    ports:
      - "8080:80"
    depends_on:
      - mariadb
    networks:
      - backend

  # =============================
  # ⏱️ InfluxDB (Time-Series DB)
  # =============================
  influxdb:
    image: influxdb:latest
    container_name: influxdb
    restart: always
    ports:
      - "8086:8086"
    environment:
      - DOCKER_INFLUXDB_INIT_MODE=setup
      - DOCKER_INFLUXDB_INIT_USERNAME=admin
      - DOCKER_INFLUXDB_INIT_PASSWORD=12345678       # >= 8 ký tự
      - DOCKER_INFLUXDB_INIT_ORG=iotorg
      - DOCKER_INFLUXDB_INIT_BUCKET=iotdata
      - DOCKER_INFLUXDB_INIT_ADMIN_TOKEN=my-super-token
    volumes:
      - ./influxdb_data:/var/lib/influxdb2
    networks:
      - backend

  # =============================
  # ⚙️ Node-RED (Logic & API)
  # =============================
  nodered:
    image: nodered/node-red:latest
    container_name: nodered
    restart: always
    ports:
      - "1880:1880"
    environment:
      - TZ=Asia/Ho_Chi_Minh
    volumes:
      - ./nodered_data:/data
    depends_on:
      - mariadb
      - influxdb
    networks:
      - frontend
      - backend

  # =============================
  # 📊 Grafana (Dashboard)
  # =============================
  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    restart: always
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_USER=admin
      - GF_SECURITY_ADMIN_PASSWORD=12345678
      - GF_INSTALL_PLUGINS=marcusolsson-json-datasource
    volumes:
      - ./grafana_data:/var/lib/grafana
    depends_on:
      - influxdb
    networks:
      - frontend
      - backend

  # =============================
  # 🌐 Nginx (Frontend Web)
  # =============================
  nginx:
    image: nginx:latest
    container_name: nginx
    restart: always
    ports:
      - "8088:80"
    volumes:
      - ./nginx/conf.d:/etc/nginx/conf.d
      - ./nginx/www:/usr/share/nginx/html
    depends_on:
      - nodered
      - grafana
    networks:
      - frontend
      - backend

# =============================
# 🌐 Network Configuration
# =============================
networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge
```
để tải  mariadb (3306), phpmyadmin (8080), nodered/node-red (1880), influxdb (8086), grafana/grafana (3000), nginx (80,443)

 giao diện login

<img width="1366" height="768" alt="image" src="https://github.com/user-attachments/assets/de2a7892-a338-4ee4-9cb5-f2e62e0166bc" />

 giao diện influxdb

<img width="1342" height="715" alt="image" src="https://github.com/user-attachments/assets/262f1efd-1453-418f-858f-b475ad3945f7" />

 giao diện phpmyadmin

<img width="1366" height="768" alt="image" src="https://github.com/user-attachments/assets/8370033b-afa4-4e75-bb7e-db7c7bb2910a" />

  giao diện nginx
  
<img width="1314" height="685" alt="image" src="https://github.com/user-attachments/assets/013601ec-096d-4565-b25f-918bd0a9459f" />

code file index.html
```
k<!DOCTYPE html>
<html lang="vi">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Web IoT - Giám sát dữ liệu | Lăng Nguyễn Minh Lượng</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      background-color: #f4f6f8;
      margin: 0;
      padding: 0;
    }

    /* --- LOGIN --- */
    #loginSection {
      display: flex;
      justify-content: center;
      align-items: center;
      height: 100vh;
      background-color: #f5f5f5;
    }

    .login-container {
      background-color: white;
      padding: 30px;
      border-radius: 10px;
      box-shadow: 0 4px 10px rgba(0,0,0,0.1);
      width: 350px;
      text-align: center;
    }

    .login-container h2 {
      color: #007bff;
    }

    input {
      width: 90%;
      padding: 10px;
      margin: 10px 0;
      border: 1px solid #ccc;
      border-radius: 5px;
    }

    button {
      width: 100%;
      padding: 10px;
      background-color: #007bff;
      border: none;
      color: white;
      font-weight: bold;
      border-radius: 5px;
      cursor: pointer;
    }

    button:hover {
      background-color: #0056b3;
    }

    .error {
      color: red;
      margin-top: 10px;
    }

    /* --- DASHBOARD --- */
    #dashboardSection {
      display: none;
    }

    header {
      background-color: #007bff;
      color: white;
      padding: 15px;
      text-align: center;
      font-size: 20px;
      font-weight: bold;
      position: relative;
    }

.logout {
  background-color: red;
  color: white;
  border: none;
  padding: 6px 12px;
  border-radius: 5px;
  cursor: pointer;
  position: absolute;
  top: 10px;
  right: 10px;
  width: auto;           /* không kéo dài */
  min-width: 90px;       /* đặt kích thước tối thiểu vừa nút */
  text-align: center;    /* căn giữa chữ */
  font-weight: bold;
               }

    .container {
      padding: 20px;
      text-align: center;
    }

    .card {
      display: inline-block;
      background: white;
      border-radius: 10px;
      box-shadow: 0 3px 10px rgba(0,0,0,0.1);
      padding: 20px;
      margin: 15px;
      width: 200px;
    }

    iframe {
      width: 90%;
      height: 400px;
      border: none;
      margin-top: 30px;
    }
  </style>
</head>
<body>

  <!-- LOGIN SECTION -->
  <section id="loginSection">
    <div class="login-container">
      <h2>Web IoT - Giám sát dữ liệu<br>(Lăng Nguyễn Minh Lượng)</h2>
      <input type="text" id="username" placeholder="Tên đăng nhập">
      <input type="password" id="password" placeholder="Mật khẩu">
      <button onclick="login()">Đăng nhập</button>
      <p id="error" class="error"></p>
    </div>
  </section>

  <!-- DASHBOARD SECTION -->
  <section id="dashboardSection">
    <header>
      Hệ thống giám sát dữ liệu IoT 
      <button class="logout" onclick="logout()">Đăng xuất</button>
    </header>

    <div class="container">
      <h2 id="welcome"></h2>

      <div class="card">
        <h3>Nhiệt độ</h3>
        <p id="temp">-- °C</p>
      </div>

      <div class="card">
        <h3>Độ ẩm</h3>
        <p id="humidity">-- %</p>
      </div>

      <div class="card">
        <h3>Nồng độ CO₂</h3>
        <p id="co2">-- ppm</p>
      </div>

      <iframe id="grafanaFrame" src="http://localhost:3000/d/your-dashboard-id" title="Grafana Dashboard"></iframe>
    </div>
  </section>

  <script>
    // ===== LOGIN FUNCTION =====
    async function login() {
      const username = document.getElementById("username").value.trim();
      const password = document.getElementById("password").value.trim();
      const errorEl = document.getElementById("error");

      if (!username || !password) {
        errorEl.textContent = "Vui lòng nhập đầy đủ thông tin!";
        return;
      }

      try {
        const res = await fetch("http://localhost:1880/login", {
          method: "POST",
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify({ username, password })
        });

        const result = await res.json();

        if (result.success) {
          sessionStorage.setItem("user", JSON.stringify(result.user));
          showDashboard(result.user.username);
        } else {
          errorEl.textContent = "Sai tài khoản hoặc mật khẩu!";
        }
      } catch (err) {
        console.error(err);
        errorEl.textContent = "Không kết nối được tới máy chủ Node-RED!";
      }
    }

    // ===== DASHBOARD FUNCTIONS =====
    function showDashboard(username) {
      document.getElementById("loginSection").style.display = "none";
      document.getElementById("dashboardSection").style.display = "block";
      document.getElementById("welcome").textContent = `Xin chào, ${username}!`;
      fetchData();
      setInterval(fetchData, 5000);
    }

    async function fetchData() {
      try {
        const res = await fetch("http://localhost:1880/api/sensors/latest");
        const data = await res.json();
        document.getElementById("temp").textContent = data.temperature + " °C";
        document.getElementById("humidity").textContent = data.humidity + " %";
        document.getElementById("co2").textContent = data.co2 + " ppm";
      } catch (e) {
        console.log("Không lấy được dữ liệu từ Node-RED.");
      }
    }

    function logout() {
      sessionStorage.clear();
      document.getElementById("dashboardSection").style.display = "none";
      document.getElementById("loginSection").style.display = "flex";
    }

    // ===== AUTO LOGIN IF SESSION EXISTS =====
    window.onload = function() {
      const user = JSON.parse(sessionStorage.getItem("user"));
      if (user) showDashboard(user.username);
    };
  </script>
</body>
</html>
```



# 5

<img width="1296" height="707" alt="image" src="https://github.com/user-attachments/assets/a4a06715-0121-4514-a7da-72476bea5551" />





<img width="1366" height="768" alt="image" src="https://github.com/user-attachments/assets/722a5049-27f6-44c8-9214-9dcb6b80f9b7" />


<img width="1366" height="768" alt="image" src="https://github.com/user-attachments/assets/d7f81480-c63e-43d0-90eb-c04e0d073275" />

<img width="1366" height="768" alt="image" src="https://github.com/user-attachments/assets/2fe7f3ec-bbec-422f-9b01-ba9a031bc7a9" />
