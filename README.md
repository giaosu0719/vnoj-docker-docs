# Hướng dẫn cài đặt hệ thống chấm điểm trực tuyến VNOJ sử dụng Docker
## Chuẩn bị
## Cài đặt Docker và Docker-Compose
Thực hiện trên Server
### Cài đặt Docker 
> Tham khảo cách cài đặt từ [document chính thức của Docker](https://docs.docker.com/engine/install/ubuntu/)

Set up Docker's apt repository.
```
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```
Install the Docker packages.
```
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io 
```
> Note: Hiện tại, chỉ có sudo mới có thể chạy các lệnh của Docker. Để các user khác cũng chạy được, cần thêm `sudo` vào trước các câu lệnh. Các lỗi như "docker: Got permission denied while trying to connect to the Docker daemon.." thường là do thiếu sudo trước câu lệnh.

### Cài đặt Docker-Compose
> Có thể tham khảo thêm tại [Install the Compose plugin](https://docs.docker.com/compose/install/linux/)
```
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```
## Cài đặt site
### Tải về mã nguồn VNOJ Docker
```
git clone --recursive https://github.com/VNOI-Admin/vnoj-docker.git
cd vnoj-docker/dmoj
```
Kể từ lúc này, các câu lệnh đằng sau sẽ có thư mục hiện hành là `/dmoj`
### Cấu hình môi trường để sử dụng Docker
Thay đổi các thông số cài đặt nhằm phù hợp với mục đích sử dụng và tăng tính bảo mật cho webserver.
Có 3 nơi mà bạn có thể chỉnh sửa:
1. `dmoj/environment/`

Nơi này chứa các biến môi trường để build Docker image.
> Note: Đổi tên các file .example tương ứng thành: mysql-admin.env, mysql.env, site.env
* mysql.env
```
MYSQL_DATABASE=dmoj
MYSQL_USER=dmoj
MYSQL_PASSWORD=deptraicogisai6969		#thay doi password
```
* mysql-admin.env
```
MYSQL_ROOT_PASSWORD=deptraicogisai6969		#thay doi password
```
* site.env
```
HOST=oj.togethercpp.xyz				#thay bang IP cua Server hoac Domain
SITE_FULL_URL=http://oj.togethercpp.xyz/				#thay bang IP cua Server hoac Domain
MEDIA_URL=http://oj.togethercpp.xyz/				#thay bang IP cua Server hoac Domain
DEBUG=0
SECRET_KEY=deptraicogisai6969		#thay doi bang ma khoa tuy y
```
2. `dmoj/nginx/conf.d/nginx.conf`

Cấu hình hình tên server_name thành IP của Server hoặc Domain

3. `dmoj/local_settings.py`

Hầu hết các thông số đã được cấu hình sẵn. Nếu muốn thêm tính năng nào thì bỏ dấu comment tính năng đó. 

Ví dụ: Để người dùng tự đăng ký tài khoản, tiến hành thêm các thông tin cấu hình email để thực hiện xác thực đăng ký tài khoản qua email (cần tạo mật khẩu ứng dụng cho email)

### Build Docker Image
Khởi tạo trước khi build
```
./scripts/initialize
```
Build Docker Image
```
sudo docker-compose build
```
Khởi động thành phần site để thực hiện cấu hình
```
sudo docker-compose up -d site
```
Khởi tạo dữ liệu cho Database
```
sudo ./scripts/migrate
```
Khởi tạo các file static
```
sudo ./scripts/copy_static
```
Load các dữ liệu cần thiết cho Website
```
sudo ./scripts/manage.py loaddata navbar
sudo ./scripts/manage.py loaddata language_small
sudo ./scripts/manage.py loaddata demo
```
### Sử dụng VNOJ Site
Quá trình cài đặt đến đây đã hoàn tất. Chạy câu lệnh bên dưới để khởi động tất cả các docker container.
```
sudo docker-compose up –d
```
Truy cập http://oj.togethercpp.xyz để kiểm tra kết quả (sử dụng IP Server hoặc Domain đã cấu hình từ trên thay thế cho `oj.togethercpp.xyz`).

Truy cập bài tập mẫu A+B và tiến hành upload testcase để kiểm tra.
## Cài đặt judge
Để đơn giản trong quá trình cài đặt, chúng ta sẽ tiếp tục cài đặt judge sử dụng Docker
> Note:<br>
	- Nếu cài đặt judge chạy trên Local Server, không cần cài đặt lại Docker.<br>
	- Nếu cài đặt judge trên Remote Judge, tiến hành cài đặt Docker theo hướng dẫn bên trên (không cần cài Docker-Compose)
> 
### Thiết lập cấu hình judge trên admin site
Truy cập http://oj.togethercpp.xyz/admin/judge/ (sử dụng IP Server hoặc Domain đã cấu hình từ trên thay thế cho `oj.togethercpp.xyz`).
Tạo các judge, lưu lại tên judge id và key (ví dụ ở đây tạo 03 judge là judge01, judge02, judge03)
### Tạo môi trường biên dịch 
Tải về môi trường biên dịch (thực hiện trên Local Server và Remote Judge)
```
git clone https://github.com/VNOI-Admin/judge-server
cd judge-server/.docker
sudo apt install make
sudo make judge-tiervnoj
```
Có thể thay thế `tiervnoj` bằng `tier1`, `tier2`, `tier3` (tier càng cao thì dung lượng càng lớn, càng được tích hợp nhiều ngôn ngữ hơn).

### Tạo judge trên Server
Tạo các file cấu hình tương ứng với mỗi judge có dạng là `judge_name.yml` (tên judge) và ghi những thông tin sau vào file:
```
id: <judge name>
key: <judge authentication key>
problem_storage_globs:
  - /problems/*
```
Ở đây, ta sẽ chạy 2 máy chấm `judge01` và `judge02` trên Local Server

Build Docker Image
```
sudo docker run \
    --name judge01 \
    --network="host" \
    -v /home/tcoj/vnoj-docker/dmoj/problems:/problems \
    --cap-add=SYS_PTRACE \
    -d \
    --restart=always \
    vnoj/judge-tiervnoj:latest \
    run -p 9999 -c /problems/judge01.yml localhost -A 0.0.0.0 -a 9111
```
> Note: <br>
	- Với mỗi judge, cần thay thế judge01 (judge name), judge01.yml (judge config), 9111 (PID) tương ứng khác nhau.<br>
	- Các judge chạy trên cùng Local Server phải có ID khác nhau (thay 9111 thành 9112, 9113, ...)
    - Thay thế đường dẫn đến file docker OJ VD: `/home/tcoj/vnoj-docker/dmoj/` -> `/home/deptraicogisai6969/vnoj-docker/dmoj/`

### Kiểm tra trạng thái của máy chấm
Mở Docker logs để kiểm tra kết quả cài đặt Judge
```
sudo docker logs -ft judge01
```
