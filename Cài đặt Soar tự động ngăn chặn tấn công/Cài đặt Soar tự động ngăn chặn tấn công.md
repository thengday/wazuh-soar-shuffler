**Các bước cài đặt Wazuh:**

Cấu hình máy ảo

![](Aspose.Words.295f68ff-7ceb-4f08-a36b-40e51627a3c7.001.png)

Ip máy Wazuh 192.168.0.117

![](Aspose.Words.295f68ff-7ceb-4f08-a36b-40e51627a3c7.002.png)

Cài đặt curl và tar (nếu chưa có):

**sudo apt-get update && sudo apt-get install curl tar -y**

![](Aspose.Words.295f68ff-7ceb-4f08-a36b-40e51627a3c7.003.png)

Tải script cài đặt Wazuh:

**curl -sO [**https://packages.wazuh.com/4.11/wazuh-install.sh**](https://packages.wazuh.com/4.11/wazuh-install.sh)**

![](Aspose.Words.295f68ff-7ceb-4f08-a36b-40e51627a3c7.004.png)

**Chạy lệnh cài đặt All-in-one:** Lệnh này sẽ cài đặt tất cả thành phần và tự tạo chứng chỉ SSL.

**sudo bash wazuh-install.sh -a**

![](Aspose.Words.295f68ff-7ceb-4f08-a36b-40e51627a3c7.005.png)

**Tạo file script integration**

sudo nano /var/ossec/integrations/custom-shuffle.py

**Khai báo Webhook đã tạo ở Soar ví dụ tấn công Bruth Force trên máy chạy Wazuh manager**

sudo nano /var/ossec/etc/ossec.conf 

Chèn đoạn code vào ngay trên </ossec\_config>

<command>

`    `<name>win\_route-null</name>

`    `<executable>route-null.cmd</executable>

`    `<expect>srcip</expect>

`    `<timeout\_allowed>yes</timeout\_allowed>

`  `</command>

`  `<active-response>

`    `<command>win\_route-null</command>

`    `<location>defined-agent</location>

`    `<agent\_id>002</agent\_id>

`  `</active-response> 

`  `<integration>

`    `<name>shuffle</name>

`    `<hook\_url><http://192.168.0.118:3001/api/v1/hooks/webhook_3d69368d-5e68-43bc>> 

`    `<rule\_id>60204</rule\_id>

`    `<alert\_format>json</alert\_format>

`  `</integration>

Link hook\_url là link của node webhook đổi từ https sang http và rule id là 60204 Multiple Windows Logon Failures

Command chặn tấn công Bruthforce 

Lưu ý đổi port 3443 đã copy từ webhook thành 3001

**Khởi động lại Wazuh**

sudo systemctl restart wazuh-manager

**Các bước cài đặt Shufle Soar:**

Cập nhật hệ thống

sudo apt-get update && sudo apt-get upgrade -y

Cài đặt Docker:

sudo apt-get install -y curl apt-transport-https ca-certificates software-properties-common

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb\_release -cs) stable"

sudo apt-get update

sudo apt-get install -y docker-ce docker-ce-cli containerd.io

**Cài đặt Docker Compose:**

sudo apt-get install docker-compose-plugin

**Cấu hình hệ thống:**

sudo sysctl -w vm.max\_map\_count=262144

echo "vm.max\_map\_count=262144" | sudo tee -a /etc/sysctl.conf

**Clone Git Shuffle:**

sudo mkdir -p /opt/shuffle

cd /opt/shuffle

sudo git clone https://github.com/Shuffle/Shuffle.git .

![](Aspose.Words.295f68ff-7ceb-4f08-a36b-40e51627a3c7.006.png)

https://192.168.0.118:3443/ tôi chạy trên local nên ip khác triển khai trên máy người khác thì thay ip cài xong thì chờ 1 lúc load database rồi đăng ký tài khoản admin rồi đăng nhập như bình thường giống shuffle.io

![](Aspose.Words.295f68ff-7ceb-4f08-a36b-40e51627a3c7.007.png)

Tạo Workflow mới 

![](Aspose.Words.295f68ff-7ceb-4f08-a36b-40e51627a3c7.008.png)

Kéo Webhook và Wazuh vào và nối chúng với nhau

![](Aspose.Words.295f68ff-7ceb-4f08-a36b-40e51627a3c7.009.png)

Note lại địa chỉ URL của Webhook để sửa vào file cấu hình của wazuh

Nối 2 Node HTTP ![](Aspose.Words.295f68ff-7ceb-4f08-a36b-40e51627a3c7.010.png)

Node Wazuh có Cấu hình như sau:

![](Aspose.Words.295f68ff-7ceb-4f08-a36b-40e51627a3c7.011.png)

Url <https://192.168.0.117:55000/security/user/authenticate> thay bằng IP máy Wazuh username và password là tài khoản của api user 

![](Aspose.Words.295f68ff-7ceb-4f08-a36b-40e51627a3c7.012.png)

Cấu hình node thứ 2 

Url: <https://192.168.0.117:55000/active-response?agents_list=$exec.all_fields.agent.id>

Body 

{

`  `"command": "win\_route-null0",

`  `"alert": {

`    `"data": {

`      `"srcip": "$exec.all\_fields.data.win.eventdata.ipAddress"

`    `}

`  `}

}

Headers

Content-Type: application/json

Authorization:Bearer $wazuh\_login.body.data.token

Node này lấy token từ node login rồi tiến hành chặn IP tấn công BruthForce có rule id 60204 

Ấn Save rồi chạy 

![](Aspose.Words.295f68ff-7ceb-4f08-a36b-40e51627a3c7.013.png)

Tiến hành tấn công 

Máy Tấn công có IP 192.168.0.99 tấn công vào máy Wazuh agent có ip 192.168.0.50

![](Aspose.Words.295f68ff-7ceb-4f08-a36b-40e51627a3c7.014.png) 

Sau khi đăng nhập nhiều lần hiện log trên Wazuh dashboard:

![](Aspose.Words.295f68ff-7ceb-4f08-a36b-40e51627a3c7.015.png)

Máy Attacker không tấn công được nữa hiện chứng chỉ không hoạt động

![](Aspose.Words.295f68ff-7ceb-4f08-a36b-40e51627a3c7.016.png)

Log trên Shuffle hiện agent id bị tấn công 

![](Aspose.Words.295f68ff-7ceb-4f08-a36b-40e51627a3c7.017.png)

Hiện IP của attacker và tên máy 

![](Aspose.Words.295f68ff-7ceb-4f08-a36b-40e51627a3c7.018.png)

Log của node chặn IP tấn công đã gửi lệnh cho agent id 002

![](Aspose.Words.295f68ff-7ceb-4f08-a36b-40e51627a3c7.019.png)

