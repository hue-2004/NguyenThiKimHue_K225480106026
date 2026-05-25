# Môn: Phát triển ứng dụng với mã nguồn mở-TEE0421  
Nguyễn Thị Kim Huệ 
Lớp: 58KTPM  

**Bài tập 04:**    
# KHAI THÁC N8N ĐỂ TỰ ĐỘNG ĐĂNG BÀI LÊN WORDPRESS  
# 
## deadline : 23h59 ngày 25 tháng 5 năm 2026.  
## Link gửi bài: [Tại đây](https://docs.google.com/spreadsheets/d/1zftQMj748nRpS-_br4_jdHZocNVvo848zqxCGcTy4uU)  

### SỬ DỤNG KẾT QUẢ ĐÃ LÀM Ở BÀI TẬP 3, BỔ SUNG VÀO DOCKER COMPOSE ĐỂ CÓ THÊM SERVICE 8N8:  

1. SỬ DỤNG DOCKER TRÊN UBUNTU ĐỂ TẠO 1 file **docker-compose.yml** chứa:   
- Mariadb: sử dụng **image: mariadb:latest** để làm hệ quản trị csdl cho wordpress, thêm các biến môi trường: TZ: "Asia/Ho_Chi_Minh",   MARIADB_ROOT_PASSWORD, MARIADB_DATABASE, MARIADB_USER, MARIADB_PASSWORD (giá trị tuỳ ý)  
- Phpmyadmin: sử dụng **image: phpmyadmin:latest** để đăng nhập vào mariadb rồi tạo csdl trống (chỉ để xem, ko cần tạo bảng từ đây, wordpress sẽ làm hết),   khai báo biến môi trường: PMA_HOST: <tên service mariadb>, PMA_ARBITRARY: 1  
- WordPress: sử dụng **image: wordpress:latest**, truyền các tham số môi trường cho wordpress là các thông tin truy cập csdl mariadb, tạo bởi Phpmyadmin,   khai báo biến môi trường:  WORDPRESS_DB_HOST: <tên service mariadb>, WORDPRESS_DB_NAME, WORDPRESS_DB_USER, WORDPRESS_DB_PASSWORD (giá trị theo mariadb đã   khai báo)  
- Cloudflared: sử dụng **image: cloudflare/cloudflared:latest** , full command và token lấy từ dashboard của cloudflare, dùng AI chuyển sang dạng docker   compose  
- N8n : sử dụng **image: n8nio/n8n:latest**, nhớ truyền biến môi trường WEBHOOK_URL theo sub-domain đã add router cho cloudflared tunnel (ví dụ: WEBHOOK_URL=https://k58-n8n.tdh.io.vn/ )  

2. Yêu cầu: sau khi có 5 service này trong file docker-compose.yml :  
- pull các images về và chạy chúng (up -d)  
- Kiểm tra các service đã running ok (ko bị restart liên tục)  
- Cấu hình cloudflare tunnel add router để public wordpress lên sub-domain1 (dùng để truy cập wordpress)  
- Cấu hình cloudflare tunnel add router để public Phpmyadmin lên sub-domain2 (dùng để truy cập phpmyadmin)  
- Cấu hình cloudflare tunnel add router để public n8n này lên sub-domain3 (dùng để truy cập và cấu hình n8n)  
- Truy cập sub-domain2 để quan sát xem cơ sở dữ liệu chưa có bảng nào!  
- Truy cập sub-domain1 để cài đặt wordpress (làm theo hướng dẫn của wordpress)  
- Truy cập sub-domain2 để quan sát xem cơ sở dữ liệu có những bảng dữ liệu nào sau khi cài wp  
- Tạo 1 bài viết trong wordpress giới thiệu về bản thân sinh viên: thông tin cá nhân, sở thích, ... bài viết có thể chứa hình ảnh, âm thanh, video, ...  
- Tạo 1 bài viết trong wordpress giới thiệu về nhữn kiến thức mà em đã học được ở môn **Phát triển ứng dụng với mã nguồn mở**  
- Truy cập sub-domain3 để cấu hình n8n:  
  + tạo tài khoản admin : nhớ điền đúng email  
  + Send me a Licence key, bước này điền đủ thông tin, làm chậm sẽ thấy mục gửi License key về mail (n8n sẽ gửi email KEY cho dùng), check email để lấy KEY 
  + Activate License key: vào trang chủ => SETTING (góc dưới trái) => Usage and plan => Enter activation key: paste key từ email vào đây => Activate => sẽ nhận đc thông báo (góc dưới phải) Your Registered Community Edition has been successfully activated.  
  + Create workflow  (home page => overview => Create workflow)  
  + Add trigger node: tìm node: Telegram => OnMessage  ; cấu hình Credential: Set up Credential => cần Nhập Access Token  
    + Access Token thì lấy ở Telegram qua việc chát với @BotFather  
    + Cần chát với bot @BotFather để đẻ ra bot mới của riêng mình. bot này sẽ là nơi nhận lệnh (promt) để AI sinh html => n8n sẽ dùng html này để đăng bài   lên wp  
    + Sau khi tạo bot mới cần copy lấy Token, và chát lần đầu với bot mới này, nội dung bất kỳ (bước này quan trọng!)  
  + Add (nối tiếp vào sau node Telegram Trigger) node: AI Google Gemini => Message a model => Set up Credential => cần Nhập API KEY  
    + Lấy API KEY tại trang: https://aistudio.google.com  => https://aistudio.google.com/api-keys  
    + cần tạo project mới, sẽ lấy được API KEY  
    + Nhập API Key lên giao diện n8n  
    + kéo thả **nội dung đã chát** với bot của telegram (phía bên trái) vào **nội dung phần PROMPT** kết quả được {{ $json.message.text }}, cần gõ thêm vào sau {{ $json.message.text }} để promt dài hơn : vd ({{ $json.message.text }}. Kết quả sinh ra ở định dạng HTML+CSS để tôi dùng HTML+CSS này tạo bài viết cho wordpress.)  
    + Turn on Output Content as JSON : để kết quả trả về dạng json  
    + Có thể thử nghiệm các thành phần khác trong Options (add Options: System message, ...) => đưa ra cái nào đáng dùng?  
  + Add (nối tiếp vào sau node Message a model) node: Code in JavaScript  
    + Code js ở dạng này, có thể phải thay đổi tuỳ theo json AI trả về.  
```
// 1. lấy dữ liệu gốc  
const rawText = $input.first().json.content.parts[0].text;  

// 2. Chuyển đổi chuỗi (đã được bọc JSON) thành Object trong JavaScript  
const cleanData = JSON.parse(rawText);  

// 3. Trả về kết quả định dạng lại gọn gàng cho n8n sử dụng  
return {
  title: cleanData.post_title,  
  content: cleanData.post_content  
};  
```

  + Add (nối tiếp vào sau node Code in JavaScript) node: WordPress => Create a Post  
    + Set up Credential: vào wp tại url: https://sub-domain1/wp-admin  => vào mục Tài Khoản => chọn user đã tạo lúc setup wordpress => Mật khẩu ứng dụng => Nhập n8n và bấm "Thêm mật khẩu ứng dụng" => copy chuỗi 24 kí tự : Đây là mật khẩu ứng dụng => paste vào mục Password của n8n Credential  
    + Wordpress URL: điền giá trị https://sub-domain1/   (giá trị này cũng khai báo trong biến môi trường WEBHOOK_URL của n8n)  
    + Ignore SSL Issues (Insecure): TURN ON  
    + Cấu hình node Create a Post: bấm nút Execute previous nodes để thấy trường giá trị của node trước trả về, kéo nội dung phần title (bên trái) vào trường title, tương tự kéo nội dung content vào content  
    + Add field (Thêm thuộc tính): Status == Publish (bài đăng sẽ ở trạng thái xuất bản ngay lập tức, mặc định nó ở giá trị Draft bản nháp)  
+ PUBLISH flow (góc trên phải) Nút này thực hiện việc xuất bản flow <=> flow sẽ tự động thực thi khi thoả mãn điều kiện trigger  
   
+ Kết quả cuối cùng cần đặt được:  
  + từ điện thoại, chát với telegram bot  
  + nội dung chát được tự động gửi tới node Telegram trigger => Gửi tới Google Gemini Message a model (bản chất là gửi Prompt) : Nhận về json kết quả của   Prompt => Gửi sang node Code in JavaScript để tách tiêu đề và nội dung => gửi đến node WordPress để Create a Post(đăng bài) với tiêu đề và nội dung từ node trước gửi sang.  
  + f5 wordpress để thấy bài viết mới đã lên sóng.  

+ Chụp ảnh quá trình thao tác/cấu hình/các kết quả trung gian đạt được  
+ Nhận xét thành quả đạt được!!!  


demo kết quả cuối cùng:  

chát với bot:  

<img width="471" height="264" alt="image" src="https://github.com/user-attachments/assets/7c439503-63b4-4529-bbec-78fa1d4933d6" />  

flow automation của n8n (nhìn bên ngoài):  

<img width="1319" height="389" alt="image" src="https://github.com/user-attachments/assets/abbdc5af-952f-4d50-8fba-0cafc7334212" />  


bài tự động đăng trên wp:  

<img width="750" height="817" alt="image" src="https://github.com/user-attachments/assets/4f7c0cec-292f-4973-9eb0-1534189cdb18" />  

## update 5 services  
<img width="1612" height="869" alt="image" src="https://github.com/user-attachments/assets/b8feb730-f116-45e4-8b66-f6361a86a323" />  

## tạo tunnel  
<img width="1904" height="877" alt="image" src="https://github.com/user-attachments/assets/430c602c-3737-4fe5-b08c-9c7b889d0881" />   

## Sau khi tạo tunnel và chọn hệ điều hành docker nó sẽ sinh ra token  
<img width="1866" height="897" alt="image" src="https://github.com/user-attachments/assets/5afdaa95-3194-40c0-89e9-e7a229d7de34" />  

## file yml sau khi update  
<img width="1485" height="770" alt="image" src="https://github.com/user-attachments/assets/6eea0c04-6962-431e-a5f4-cf9fb227ea44" />   

## khởi chạy các container  
<img width="1474" height="748" alt="image" src="https://github.com/user-attachments/assets/c1dff5ef-9b38-4541-81a6-3431ec5c4ce8" />

## kiểm tra trạng thái hệ thống  
<img width="1470" height="745" alt="image" src="https://github.com/user-attachments/assets/2b73203b-5904-4445-9458-196e356dec1d" />  

## cấu hình cho wordpress  
<img width="1496" height="876" alt="image" src="https://github.com/user-attachments/assets/9627a674-fc62-41a1-aa81-942da63fdcc2" />

## cấu hình cho phpmyadmin
<img width="1648" height="832" alt="image" src="https://github.com/user-attachments/assets/435b4081-35ff-491e-904b-cd254d5de6ef" />  

## cấu hình cho n8n
<img width="1655" height="847" alt="image" src="https://github.com/user-attachments/assets/10dc9bf5-0c50-4808-afe1-38a5cfd7ec60" />   

## kết quả đạt được 
<img width="1850" height="738" alt="image" src="https://github.com/user-attachments/assets/a1bf72e6-28f1-417c-bea5-b20ffbb9e4e1" />  

## Truy cập https://pma.huek58.io.vn/ để quan sát xem cơ sở dữ liệu chưa có bảng nào! 
<img width="1839" height="832" alt="image" src="https://github.com/user-attachments/assets/df6e33fb-6bf3-458a-94b9-4eb00d10be94" />  

<img width="1842" height="780" alt="image" src="https://github.com/user-attachments/assets/5e175fc5-c3b3-448e-8a12-1355b5739288" />  

<img width="1842" height="780" alt="image" src="https://github.com/user-attachments/assets/7b0a19f9-0395-4a76-9c1b-6f4c5a625795" />

Truy cập https://pma.huek58.io.vn/ để cài đặt wordpress (làm theo hướng dẫn của wordpress)
<img width="1585" height="944" alt="image" src="https://github.com/user-attachments/assets/4dc571c3-8477-473d-87f1-e1c4d48495a5" />    

<img width="1651" height="960" alt="image" src="https://github.com/user-attachments/assets/1feff58f-78ba-4a06-bbbf-47f93a15ab45" />   

<img width="1586" height="779" alt="image" src="https://github.com/user-attachments/assets/71aa8021-c65d-4941-a59b-a454d4c9faf7" />  

## sau khi đăng nhập thành công sẽ ra trang chủ wordpress
<img width="1894" height="961" alt="image" src="https://github.com/user-attachments/assets/9faf0d8a-e1e4-4440-9ef3-9ae7c4198a81" />  

## Truy cập sub-domain2 để quan sát xem cơ sở dữ liệu có những bảng dữ liệu nào sau khi cài wp  
<img width="1840" height="968" alt="image" src="https://github.com/user-attachments/assets/13330af8-224e-4453-8804-350af0ccd1e2" />  

## bài viêt giới thiệu bản thân
<img width="1867" height="957" alt="image" src="https://github.com/user-attachments/assets/cfd9e27b-64e6-4124-adec-397f029eed02" />  

<img width="1867" height="962" alt="image" src="https://github.com/user-attachments/assets/968f705a-c4c7-4a3f-b003-a04f91a001f7" />  

<img width="1861" height="944" alt="image" src="https://github.com/user-attachments/assets/b3fe0778-188f-42a8-b6c2-ab8896c1d514" />  

<img width="1854" height="960" alt="image" src="https://github.com/user-attachments/assets/41df8ba6-efa1-4b1f-9795-eb62f1957425" />  

## truy cập http://n8n.huek58.io.vn KÍCH HOẠT LICENSE COMMUNITY CHO N8N  
<img width="1810" height="945" alt="image" src="https://github.com/user-attachments/assets/363f19a6-a1af-4185-80f4-a0c4cca0ca92" />  

<img width="1848" height="951" alt="image" src="https://github.com/user-attachments/assets/7077a744-80cf-4801-93ba-83f020b39d2e" />  

<img width="1905" height="964" alt="image" src="https://github.com/user-attachments/assets/09071825-5853-4672-b79c-7baa00790cfa" />  

<img width="1905" height="964" alt="image" src="https://github.com/user-attachments/assets/f9673f4e-f2a9-40f3-9d4d-a30c3cf6104d" />  

## Trên giao diện n8n: Chọn Settings (bánh răng góc dưới bên trái) -> Usage and plan -> Bấm Enter activation key -> Paste mã vào và bấm Activate. Bạn sẽ thấy thông báo kích hoạt thành công góc dưới bên phải.  

<img width="1869" height="970" alt="image" src="https://github.com/user-attachments/assets/7131eece-e52d-45b2-9311-c2a25e8cde1c" />   

<img width="1284" height="2778" alt="image" src="https://github.com/user-attachments/assets/18f4cfa6-9c98-4ebe-afe7-75185f6c4f9a" />  

## Tại trang chủ n8n, chọn Workflows -> Create workflow  
<img width="1886" height="965" alt="image" src="https://github.com/user-attachments/assets/fc38a9d3-1980-43f3-a581-0cb24f762403" />  

## Add trigger node: tìm node: Telegram => OnMessage  
<img width="1855" height="961" alt="image" src="https://github.com/user-attachments/assets/3e4e6b8e-7fd1-4696-97e7-463688d19f19" />   

## telegram bot token  
<img width="1823" height="957" alt="image" src="https://github.com/user-attachments/assets/75696e6b-723a-42e6-84aa-0aae725391eb" />  

<img width="1830" height="932" alt="image" src="https://github.com/user-attachments/assets/bda5578a-0a66-4d74-9293-209a43b19824" />  

## Bấm Listen for test event để n8n chờ. Lúc này, lấy điện thoại nhắn tin cho Bot Telegram một câu bất kỳ  
<img width="1792" height="810" alt="image" src="https://github.com/user-attachments/assets/2c2359f0-83dc-4b89-b945-40d82e4ab1df" />  

## Add (nối tiếp vào sau node Telegram Trigger) node: AI Google Gemini => Message a model => Set up Credential => cần Nhập API KEY
<img width="1860" height="957" alt="image" src="https://github.com/user-attachments/assets/8f59c6dc-5410-4a18-ad6f-dc005b63ba28" />  

## Lấy API KEY tại trang: https://aistudio.google.com => https://aistudio.google.com/api-keys
<img width="1860" height="975" alt="image" src="https://github.com/user-attachments/assets/8a29a7d6-d1ed-480a-836e-4feb225c7138" />   

<img width="1793" height="955" alt="image" src="https://github.com/user-attachments/assets/7c767110-88d3-4ebe-a9f0-0b3eee27d5b6" />  

 <img width="1859" height="939" alt="image" src="https://github.com/user-attachments/assets/a77c9905-2e8f-4e26-8fe6-bee9965e24c2" />

  <img width="1839" height="943" alt="image" src="https://github.com/user-attachments/assets/362a1fcd-e78e-4e3c-85c5-ef0b33d9ebb1" />

  <img width="1839" height="966" alt="image" src="https://github.com/user-attachments/assets/fec3463a-6425-4d3e-be8d-c64da56aadab" />  

  <img width="1853" height="973" alt="image" src="https://github.com/user-attachments/assets/94072231-ffaa-4977-8981-1e16413880c1" />  

  <img width="1836" height="945" alt="image" src="https://github.com/user-attachments/assets/fb5a9815-bcaf-4ed3-86b4-afde0d3e8434" />   

  <img width="1855" height="944" alt="image" src="https://github.com/user-attachments/assets/eaf1d06a-990c-456b-97cc-0bb374547ddd" />  

  <img width="1864" height="971" alt="image" src="https://github.com/user-attachments/assets/cba867f5-957f-48a9-b24f-df45b17fd4a1" />  

  <img width="1848" height="977" alt="image" src="https://github.com/user-attachments/assets/e5ae384e-05a7-4d5d-bb9f-fa6a1d20bbd8" />

<img width="1299" height="704" alt="image" src="https://github.com/user-attachments/assets/09fcf848-f87b-48c1-8abf-114829aa62d1" />  

<img width="1815" height="838" alt="image" src="https://github.com/user-attachments/assets/ca9090d2-00a9-4237-80a0-80be4d6dcffe" />  

<img width="1843" height="947" alt="image" src="https://github.com/user-attachments/assets/5cc4f847-34e2-4760-a588-b25f1d50c76b" />  

<img width="1822" height="970" alt="image" src="https://github.com/user-attachments/assets/d3e5928c-7939-490d-b5f4-42c9d6ef7640" />  

<img width="1855" height="988" alt="image" src="https://github.com/user-attachments/assets/4b9f91ad-ae1d-4d54-afbd-82794d7380c8" />  





















 


































