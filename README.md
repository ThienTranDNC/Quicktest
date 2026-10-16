# QuickTest DNC – Hướng dẫn cài đặt trên hosting PHP + MySQL

Hệ thống thi trắc nghiệm trực tuyến nhiều môn – Trường Công nghệ số & Trí tuệ nhân tạo, Trường Đại học Nam Cần Thơ.

Tài liệu này dành cho bản chạy trên **hosting thông thường**, loại có cPanel, DirectAdmin hoặc Plesk, chạy **PHP + MySQL**. Ví dụ dưới đây dùng tên miền **https://giaolang.edu.vn**.

**Phiên bản 2.0 – nhiều giáo viên:**
- Thành viên tự đăng ký bằng email + mật khẩu. Tài khoản @nctu.edu.vn dùng nút Google.
- Giáo viên đăng ký xong phải chờ quản trị viên duyệt.
- **Mỗi giáo viên chỉ thấy và quản lý câu hỏi, đề, kết quả của mình.**
- Có **file mẫu** để soạn đề bằng Excel.
- Giao diện đăng nhập mới.

**Đang chạy bản cũ (1.x)?** Làm theo **mục 8** để nâng cấp mà không mất dữ liệu.

Mã nguồn viết bằng PHP thuần nên không cần Composer, không cần Node.js và không cần quyền SSH: chỉ cần tải tệp lên rồi chạy trình cài đặt trên trình duyệt.

---

## 1. Yêu cầu hosting

| Hạng mục | Yêu cầu |
|---|---|
| PHP | **8.0 trở lên** (khuyến nghị 8.2 hoặc 8.3) |
| Tiện ích PHP | `pdo_mysql`, `zip`, `xmlreader`, `simplexml`, `curl`, `mbstring` (hầu hết hosting bật sẵn) |
| CSDL | **MySQL 5.7 / 8.0** hoặc **MariaDB 10.3** trở lên |
| Web server | Apache hoặc LiteSpeed (đọc được tệp `.htaccess`). Nginx vẫn chạy được, xem mục 9 |
| Tên miền và SSL | `giaolang.edu.vn` trỏ về hosting, có **HTTPS**. Hầu hết hosting cấp miễn phí qua AutoSSL hoặc Let's Encrypt |
| Kết nối ra ngoài | Hosting phải gọi được `https://oauth2.googleapis.com` để đăng nhập Google. Hosting thông thường đều cho phép |

Trình cài đặt tự kiểm tra các yêu cầu trên và báo mục nào chưa đạt.

---

## 2. Chuẩn bị trên hosting

1. **Gắn tên miền:** trỏ `giaolang.edu.vn` về hosting, thường là đổi nameserver hoặc bản ghi A ở nơi mua tên miền. Sau đó bật **SSL** cho tên miền trong cPanel → *SSL/TLS Status* (hoặc *Let's Encrypt*).
2. **Tạo CSDL:** vào cPanel → *MySQL® Databases* (DirectAdmin: *MySQL Management*).
   - Tạo CSDL, ví dụ `cpuser_quicktest`.
   - Tạo tài khoản CSDL, ví dụ `cpuser_qt`, đặt mật khẩu mạnh.
   - **Add User to Database**, chọn **ALL PRIVILEGES**.
   - Ghi lại 3 thông tin: tên CSDL, tài khoản, mật khẩu. Máy chủ CSDL thường là `localhost`.
3. **Chọn phiên bản PHP:** cPanel → *Select PHP Version* (hoặc *MultiPHP Manager*), chọn PHP 8.2 hoặc 8.3. Nếu trình cài đặt báo thiếu tiện ích nào, bật nó ở tab *Extensions*.

---

## 3. Tạo OAuth Client trên Google (để đăng nhập bằng tài khoản trường)

Bước này do quản trị Google Workspace của trường thực hiện, hoặc tài khoản có quyền tạo dự án trên Google Cloud.

1. Vào https://console.cloud.google.com, tạo dự án, ví dụ `QuickTest DNC`.
2. Mở phần màn hình đồng ý OAuth: **APIs & Services → OAuth consent screen** (giao diện mới: **Google Auth Platform → Branding / Audience**).
   - **User type / Audience: Internal**: chỉ tài khoản @nctu.edu.vn đăng nhập được.
   - Tên ứng dụng: `QuickTest DNC`. Phạm vi (scopes): `openid`, `email`, `profile`.
3. **Credentials → Create credentials → OAuth client ID** (giao diện mới: **Clients → Create client**).
   - Application type: **Web application**.
   - **Authorized redirect URIs:** `https://giaolang.edu.vn/auth.php?a=callback`
4. Ghi lại **Client ID** và **Client secret**.

> Tên các mục trên Google Cloud Console có thể thay đổi theo thời gian. Nếu không thấy đúng tên, tìm chữ "OAuth" trong ô tìm kiếm của Console.
>
> Đăng nhập Google **không bắt buộc**. Chưa có Client ID thì thành viên vẫn đăng ký và đăng nhập bằng email + mật khẩu; bổ sung Google sau (mục 5).

---

## 4. Tải mã nguồn lên và cài đặt

1. Vào cPanel → **File Manager** → thư mục gốc của tên miền (thường là `public_html`; với tên miền phụ là thư mục riêng của tên miền đó).
2. **Upload** tệp `QuickTest-DNC-PHP.zip`, chuột phải chọn **Extract**. Sau khi giải nén, trong thư mục gốc phải thấy ngay `index.php`, `install.php`, `.htaccess` cùng các thư mục `app/` và `assets/`. Nếu chúng nằm trong một thư mục con thì chuyển ra ngoài.
   - `.htaccess` là tệp ẩn. Trong File Manager bật *Settings → Show Hidden Files* để thấy.
3. Mở trình duyệt, vào **https://giaolang.edu.vn/install.php**:
   - Xem bảng kiểm tra máy chủ: các mục phải có dấu ✓.
   - **Địa chỉ trang web:** `https://giaolang.edu.vn`
   - **CSDL:** điền 3 thông tin đã ghi ở mục 2.
   - **Tài khoản quản trị đầu tiên:** họ tên, email (ví dụ `tranvanthien@nctu.edu.vn`) và mật khẩu. Đây là tài khoản quản lý toàn hệ thống.
   - **Google (không bắt buộc):** dán Client ID và Client secret nếu đã có.
   - Bấm **Kiểm tra kết nối và cài đặt**.
4. Thấy dòng **"Cài đặt thành công… 200 câu hỏi, 3 đề"** là xong.
5. **Xóa tệp `install.php`** trong File Manager. Đây là bước bắt buộc để bảo mật.
6. Bật bắt buộc HTTPS: mở tệp `.htaccess`, bỏ dấu `#` ở hai dòng `RewriteCond %{HTTPS} !=on` và `RewriteRule ^ https://...`.

> **Chạy `install.php` ngay sau khi tải lên.** Trong khoảng thời gian chưa cài, ai mở được trang cài đặt cũng có thể điền thông tin vào đó.
>
> Nếu trình cài đặt báo *"Không ghi được tệp app/config.php"*, trang sẽ hiện sẵn nội dung tệp. Tạo tệp `app/config.php` bằng File Manager, dán nội dung đó vào, lưu lại, rồi tải lại `install.php`.

**Các địa chỉ sau khi cài:**

| Trang | Địa chỉ |
|---|---|
| Trang thi | https://giaolang.edu.vn/ |
| Đăng nhập / đăng ký | https://giaolang.edu.vn/auth.php |
| Trang quản lý đề (giáo viên, quản trị viên) | https://giaolang.edu.vn/admin.php (hoặc /admin) |
| Tài khoản của tôi | https://giaolang.edu.vn/account.php |
| Link riêng của từng đề | https://giaolang.edu.vn/?k=&lt;mã&gt; (dạng ngắn `/d/<mã>` cũng dùng được) |

---

## 5. Tài khoản, vai trò và đăng nhập Google

**Ba vai trò:**

| Vai trò | Làm được gì |
|---|---|
| **Người dự thi** | Làm bài, xem kết quả của mình (nếu đề cho xem đáp án) |
| **Giáo viên** | Mọi việc của người dự thi, cộng: ngân hàng câu hỏi, đề thi, link/QR, kết quả và thống kê **của riêng mình**. Không thấy đề, câu hỏi hay kết quả của giáo viên khác |
| **Quản trị viên** | Xem toàn hệ thống (lọc theo từng giáo viên ở ô **Đang xem**), sửa giúp đề và câu hỏi của giáo viên, quản lý **Thành viên**, **Cài đặt** chung, **sao lưu** |

**Đăng ký và duyệt giáo viên:**

1. Người dùng vào trang đăng nhập, chọn thẻ **Đăng ký**, điền họ tên, email, đơn vị, mật khẩu, rồi chọn vai trò *Người dự thi* hoặc *Giáo viên*.
2. Chọn *Giáo viên* thì tài khoản ở trạng thái **chờ duyệt**. Trong lúc chờ, người đó vẫn vào thi được như người dự thi.
3. Quản trị viên vào **Quản lý đề thi → Thành viên**. Số yêu cầu đang chờ hiện ngay trên thẻ. Bấm **Duyệt GV** hoặc **Từ chối**.
4. Cũng ở thẻ Thành viên, quản trị viên có thể:
   - đổi vai trò, kể cả nâng ai đó lên quản trị viên;
   - **Khóa** tài khoản (người đó bị đăng xuất ngay);
   - **Cấp lại MK**: tạo mật khẩu tạm, gửi trực tiếp cho người dùng.

Người dùng tự đổi mật khẩu, họ tên, đơn vị ở trang **Tài khoản**. Người dự thi cũng xin quyền giáo viên từ trang này.

> Hệ thống **không gửi email xác minh**, vì hosting chia sẻ thường gửi thư không ổn định. Vì vậy quyền giáo viên luôn do quản trị viên duyệt. Hãy kiểm tra họ tên, email trước khi bấm Duyệt.

**Bật đăng nhập Google.** Mở `app/config.php` bằng File Manager (*Edit*) và điền:

```php
'google_client_id' => 'xxxx.apps.googleusercontent.com',
'google_client_secret' => 'GOCSPX-xxxx',
'allowed_domains' => 'nctu.edu.vn',
'admin_emails' => 'tranvanthien@nctu.edu.vn',
```

Lưu lại là có hiệu lực ngay. Khi Google đã bật:
- Trang đăng nhập có nút **Đăng nhập bằng Google**, chỉ nhận tài khoản @nctu.edu.vn.
- Email @nctu.edu.vn **không đăng ký bằng mật khẩu được**, phải dùng nút Google. Điều này chặn việc người khác đăng ký mạo danh email của trường.
- Lần đầu bấm Google, hệ thống tự tạo tài khoản *Người dự thi*. Email nằm trong `admin_emails` tự thành quản trị viên.
- Redirect URI khai trên Google Cloud vẫn là `https://giaolang.edu.vn/auth.php?a=callback` (thay đúng tên miền hoặc thư mục đang dùng).

**Dọn dữ liệu thi thử** trước khi thi thật: vào Quản lý đề thi → **Kết quả**, mở từng lượt rồi **Xóa lượt thi này**. Muốn xóa hết: Cài đặt → **Tải bản sao lưu**, rồi trong phpMyAdmin chạy `DELETE FROM qt_attempts;` (bảng chi tiết `qt_answers` tự xóa theo).

---

## 6. Vận hành kỳ thi

Trang quản trị và giao diện thi giống bản chạy trên máy chủ nội bộ:

- **Tổng quan:** số người đang làm bài, số bài đã nộp, điểm trung bình và tỷ lệ đạt từng đề.
- **Đề thi:** thêm, sửa đề (hình thức, số câu, thời gian, giờ mở và đóng, xem đáp án, làm lại, xáo trộn). Nút **Link & QR** để sao chép link, tải ảnh QR (PNG), mở trang in QR khổ A4, hoặc đổi link mới.
- **Câu hỏi:** tìm, sửa, thêm câu; **Tải file mẫu**, **Nhập từ Excel**, **Xuất Excel**.
- **Kết quả:** xem chi tiết từng bài; **Xuất Excel** gồm trang KetQua và ChiTiet.
- **Thống kê:** theo xếp loại, mảng nội dung, đối tượng, đơn vị; tỷ lệ đúng từng câu.
- **Thành viên** (quản trị viên): duyệt giáo viên, đổi vai trò, khóa, cấp lại mật khẩu.
- **Cài đặt** (quản trị viên): tiêu đề trang thi, thời gian nộp trễ cho phép, danh sách đối tượng và đơn vị, sao lưu, nhật ký thao tác.

**Người thi chỉ vào đề bằng link hoặc mã QR.** Trang thi không hiển thị danh sách đề. Giáo viên gửi link (hoặc in mã QR) của từng đề cho thí sinh; người thi mở link hoặc quét QR, đăng nhập rồi làm đúng đề đó. Người thi cũng có thể dán link vào ô *"Hoặc dán đường link đề thi vào đây"* trên trang thi. Bấm **Đổi link mới** thì link và QR cũ hết hiệu lực ngay.

**Soạn đề bằng file mẫu (giáo viên):**

1. Vào Quản lý đề thi → **Câu hỏi** → **Tải file mẫu**. Tệp gồm 3 trang:
   - `CauHoi`: tiêu đề cột và 3 câu ví dụ.
   - `DeThi`: 2 đề ví dụ, mã đề đã gắn tiền tố riêng của Thầy/Cô.
   - `HuongDan`: giải thích từng cột.
2. Xóa các dòng ví dụ, điền câu hỏi của mình. Nếu muốn tạo đề luôn thì điền trang `DeThi`; nếu không, tạo đề trên web sau.
3. Bấm **Nhập từ Excel** và chọn tệp. Hệ thống báo số câu và số đề đã nhập. Nếu tệp có lỗi, hệ thống báo đúng dòng và không ghi gì.

Mã câu (`MaCau`) chỉ cần không trùng **trong bộ của Thầy/Cô**: hai giáo viên đặt cùng mã `C01` vẫn được. **Mã đề** (`MaDe`) phải không trùng trên toàn hệ thống. Trùng với đề của người khác thì hệ thống báo để đổi mã.

**Dùng cho nhiều môn:** mỗi môn hoặc kỳ thi là một **nhóm câu hỏi** (cột `De`, ví dụ `GK1`, `CK`) cộng một hoặc nhiều đề dùng nhóm đó. Đề chỉ lấy câu trong ngân hàng của chính giáo viên ra đề.

---

## 7. Sao lưu và khôi phục

- **Từ trang quản trị:** Cài đặt → **Tải bản sao lưu (.sql)**. Nên tải sau mỗi ca thi.
- **Từ hosting:** cPanel → *Backup* hoặc phpMyAdmin → chọn CSDL → *Export*.
- **Khôi phục:** phpMyAdmin → chọn CSDL đã cài QuickTest → *Import* tệp `.sql`. Tệp sao lưu sẽ thay toàn bộ dữ liệu hiện có bằng dữ liệu trong tệp.
- Giữ một bản sao của `app/config.php` ở nơi an toàn. Tệp này chứa mật khẩu CSDL và khóa bí mật.

---

## 8. Cập nhật phiên bản mới (kể cả nâng cấp từ bản 1.x lên 2.0)

1. Vào Quản lý đề thi → Cài đặt → **Tải bản sao lưu (.sql)** để giữ dữ liệu phòng khi cần.
2. Tải tệp zip mới lên thư mục cài đặt và **giải nén đè**. Tệp `app/config.php` không có trong zip nên được giữ nguyên.
3. Mở trang web bất kỳ: hệ thống **tự nâng cấp CSDL**. Mọi câu hỏi, đề, kết quả cũ được chuyển cho tài khoản quản trị (email đầu tiên trong `admin_emails`).
4. **Riêng khi nâng cấp từ bản 1.x:** bản cũ chỉ đăng nhập bằng Google (hoặc chế độ thử), nên tài khoản quản trị chưa có mật khẩu. Làm thêm:
   - Mở `https://giaolang.edu.vn/install.php`. Bản zip mới có tệp này.
   - Trang sẽ hỏi **mật khẩu CSDL** (trong `app/config.php`) để xác nhận Thầy là chủ hosting, rồi cho đặt mật khẩu quản trị.
   - Đăng nhập tại `auth.php` bằng email quản trị và mật khẩu vừa đặt.
   - **Xóa lại tệp `install.php`.**
5. Dòng `'auth_mode' => ...` trong `app/config.php` của bản cũ không còn tác dụng, để nguyên cũng được. Chế độ thử (nhập email tùy ý) đã được bỏ.

---

## 9. Hiệu năng và lưu ý khi thi đông người

**Kết quả kiểm thử** trên máy 2 CPU (Apache + PHP 8.3, công cụ tạo tải chạy cùng máy):

| Tình huống | MariaDB 10.11 | MySQL 8.0 |
|---|---|---|
| 450 người bấm "Bắt đầu" cùng lúc | 450/450, chậm nhất 1,2 s | 450/450, chậm nhất 1,4 s |
| 450 người nộp bài 100 câu cùng lúc | 450/450, chậm nhất 2,5 s | 450/450, chậm nhất 2,8 s |
| 1000 người nộp cùng lúc (MariaDB) | 1000/1000, chậm nhất 5,9 s | – |

Không mất bản ghi nào.

**Giới hạn của hosting chia sẻ.** Hosting giá rẻ thường giới hạn **số tiến trình PHP chạy cùng lúc** (cPanel gọi là *Entry Processes*, thường 20–50) và giới hạn CPU, RAM. Khi vượt giới hạn, máy chủ trả lỗi 508/503. Trang thi đã có sẵn cơ chế **tự gửi lại bài khi máy chủ bận** và **giữ bài làm trên máy người thi**, nên không mất bài, nhưng người thi có thể phải chờ.

**Trước ca thi đông** (ví dụ khoảng 450 người cùng nộp):

1. Hỏi nhà cung cấp hosting giới hạn Entry Processes, CPU, RAM của gói đang dùng. Cân nhắc nâng gói hoặc dùng **VPS** trong mùa thi.
2. Chạy thử tải trên chính hosting. Cách làm: cài **một bản riêng** ở thư mục hoặc tên miền phụ, dùng CSDL riêng, ở chế độ thử. Công cụ kiểm thử nằm trong tệp `QuickTest-DNC-PHP-KiemThu.zip` (chạy `node load.js 450` trên một máy có Node.js; sửa biến `BASE` thành địa chỉ bản thử). Xóa bản thử sau khi xong.
3. Nếu dùng VPS với Apache **prefork**: tắt `KeepAlive` hoặc đặt `KeepAliveTimeout 2`, hoặc dùng PHP-FPM với mpm_event/Nginx. Trong kiểm thử, `KeepAlive On` (5 giây) làm thời gian chờ tăng từ khoảng 2,5 giây lên khoảng 9 giây vì tiến trình bị giữ bởi kết nối rảnh. LiteSpeed, loại thường gặp ở hosting Việt Nam, không gặp vấn đề này.

**Đăng nhập đồng loạt.** Mật khẩu được mã hóa bằng bcrypt, cố ý chậm để chống dò mật khẩu. Kiểm thử trên máy 2 CPU: **450 người cùng bấm đăng nhập bằng mật khẩu trong cùng một giây** thì người chậm nhất chờ khoảng **15 giây** (tất cả đều vào được). Đăng nhập Google không tốn thời gian này. Cách tránh: cho người thi **đăng nhập trước giờ thi 5–10 phút**. Phiên đăng nhập giữ 12 giờ, đến giờ chỉ cần quét QR là vào đề.

**Hosting dùng Nginx (không đọc `.htaccess`):** cần thêm vào cấu hình Nginx của tên miền:

```nginx
location ^~ /app/ { deny all; }
location ~ \.(md|sql|log|xlsx)$ { deny all; }
rewrite ^/d/([A-Za-z0-9_-]+)/?$ /index.php?k=$1 redirect;
```

Thư mục `app/` chứa cấu hình và ngân hàng câu hỏi. Các tệp nhạy cảm trong đó đều là tệp `.php` nên không đọc trộm được nội dung, nhưng vẫn nên chặn hẳn thư mục này.

---

## 10. Lưu ý bảo mật

- Xóa `install.php` ngay sau khi cài.
- Không chia sẻ `app/config.php`, Client secret hay mật khẩu CSDL qua chat hoặc email.
- Giữ HTTPS bắt buộc (mục 4, bước 6). Cookie đăng nhập được đặt `Secure`, `HttpOnly`, `SameSite=Lax`.
- Đáp án không gửi xuống trình duyệt trước khi nộp; bài được chấm trên máy chủ. Mỗi lượt thi gắn với tài khoản đã đăng nhập.
- Nhật ký thao tác nằm ở Quản lý đề thi → Cài đặt: đăng nhập, đăng ký, duyệt giáo viên, sửa đề, nhập tệp, xóa kết quả, cấp lại mật khẩu…
- **Chống dò mật khẩu:**
  - Sai 8 lần trong 15 phút thì email đó bị tạm khóa đăng nhập 15 phút.
  - Giới hạn theo địa chỉ IP đặt cao (500 lần / 15 phút) vì cả trường có thể ra Internet qua chung một địa chỉ.
- Mật khẩu lưu dạng băm bcrypt, không ai đọc được mật khẩu gốc, kể cả quản trị viên. Quên mật khẩu thì quản trị viên **cấp lại mật khẩu tạm**.
- Giáo viên chỉ truy cập được dữ liệu của mình, kể cả khi cố gửi yêu cầu kèm email của người khác. Điều này đã được kiểm thử tự động.
