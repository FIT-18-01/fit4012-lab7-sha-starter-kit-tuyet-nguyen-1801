# FIT4012 Lab 7 - Báo cáo 1 trang: SHA-256

**Sinh viên:** Nguyen Tuyet  
**Ngày thực hiện:** 2026-05-22

## 1. Mục tiêu / Objective

Bài thực hành này nhằm mục tiêu:

- Hiểu và quan sát từng bước của thuật toán SHA-256: khởi tạo H0, sinh lịch thông điệp W[0..63], chạy 64 vòng nén với các hàm `Ch`, `Maj`, `Sigma`, cập nhật trạng thái băm trung gian và xuất kết quả 256-bit.
- Chạy chương trình băm chuỗi và kiểm tra với known answer test vectors (NIST).
- Xây dựng chương trình kiểm tra toàn vẹn file bằng cách so sánh SHA-256 hash.
- Minh họa lưu trữ mật khẩu an toàn hơn bằng cách băm SHA-256 và bổ sung salt.

## 2. Cách làm / Approach

Cá nhân thực hiện theo các bước sau:

1. **Đọc code** `structure.h` và `sha256_lib.h` để nắm rõ các hàm nền tảng: `ROTR`, `SHR`, `Ch`, `Maj`, `Sigma0_256`, `Sigma1_256`, `sigma0_256`, `sigma1_256`, hàm biến đổi `sha256_transform()` và hàm tính băm `calculate_sha256_bytes()`.
2. **Biên dịch** 4 chương trình bằng g++ (C++17):  
   `g++ -std=c++17 -Wall -Wextra -pedantic sha_procedure.cpp -o sha256`  
   (tương tự cho `file_integrity`, `password_hash`, `salted_password_hash`).
3. **Chạy known answer test** với `--self-test` và so sánh kết quả với vector NIST.
4. **Kiểm tra toàn vẹn file** bằng cách tính hash trước, sau đó thêm nội dung vào file và kiểm tra lại.
5. **Băm mật khẩu** dùng `password_hash`: đăng ký (lưu hash vào file), đăng nhập đúng và đăng nhập sai.
6. **Thêm salt** dùng `salted_password_hash`: đăng ký cùng một mật khẩu hai lần và so sánh hai bản ghi `salt:hash`.
7. **Chạy 5 test shell** trong thư mục `tests/` để kiểm tra tự động.

## 3. Kết quả / Result

### 3.1 Known Answer Test Vectors (NIST)

| Input | Expected SHA-256 | Kết quả |
|-------|-----------------|---------|
| `""` (chuỗi rỗng) | `e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855` | **[PASS]** |
| `"abc"` | `ba7816bf8f01cfea414140de5dae2223b00361a396177a9cb410ff61f20015ad` | **[PASS]** |
| `"hello FIT4012 SHA"` | `e942ae0ecb44fe48e1490144162fc64e9c3c6c399bb4e2d686532195cdc3eae6` | **[PASS]** |

### 3.2 Kiểm tra toàn vẹn file

- **Hash của file mẫu** (`"FIT4012 SHA file integrity sample\n"`) trước khi sửa:  
  `5ee62dc925a9958dbd6732c570a23c7f65a8c11066e889b15068cfb4bf1a0bd9`
- **Kết quả kiểm tra trước khi sửa:** `[PASS] File integrity OK`
- **Sau khi thêm dòng "tampered content"** vào file:
  - Hash mới: `a69e0b7a0768cf3c7258b5ee277b9238604a97be5cec4a6e79eafd5d94a065ba`
  - **Kết quả:** `[FAIL] File was changed or expected hash is incorrect`

### 3.3 Băm mật khẩu (không salt)

- Đăng ký mật khẩu `"fit4012-demo-password"` → hash được lưu vào file.
- Đăng nhập với mật khẩu **đúng:** `[PASS] Login success`
- Đăng nhập với mật khẩu **sai** (`"wrong-password"`): `[FAIL] Login failed: wrong password`

### 3.4 Băm mật khẩu có salt

- Đăng ký cùng mật khẩu `"fit4012-demo-password"` hai lần với salt ngẫu nhiên:
  - Bản ghi 1: `a3dfa68c742c82de7d0689c297363a7a:0c67baeb19a2c285682a8b409bca8b0130841e0bbc7bc652f0492764fbd3e5ea`
  - Bản ghi 2: `f207950cfbbec7264ddff1eef6a60908:0815f16e04b62786c03a21837e511acccf292b642e0d218fef6c5279299a717e`
- **Hai bản ghi có giống nhau không?** **KHÔNG** — salt khác nhau nên hash cũng khác nhau hoàn toàn dù cùng mật khẩu.
- Đăng nhập với mật khẩu đúng: `[PASS] Login success`
- Đăng nhập với mật khẩu sai: `[FAIL] Login failed: wrong password`

### 3.5 Kết quả chạy 5 test shell

Tất cả 5 test đều pass:
- `test_sha_compile.sh` → `[PASS] SHA programs compile successfully.`
- `test_known_vectors.sh` → `[PASS] SHA-256 known answer tests passed.`
- `test_file_integrity_tamper.sh` → `[PASS] Tamper / flip 1 byte negative test passed.`
- `test_password_hash.sh` → `[PASS] Password hash and wrong password negative test passed.`
- `test_salted_password.sh` → `[PASS] Salted password test passed.`

## 4. Kết luận / Conclusion

**SHA-256 giúp phát hiện thay đổi dữ liệu như thế nào?**  
SHA-256 là hàm băm một chiều với tính chất avalanche effect: chỉ cần thay đổi 1 byte trong file, hash đầu ra thay đổi hoàn toàn (256-bit khác nhau). Chương trình `file_integrity` tính SHA-256 của file hiện tại rồi so sánh với hash đã lưu trước đó — nếu file bị sửa, hai hash sẽ không khớp và chương trình báo `[FAIL]`.

**Vì sao cần salt khi lưu hash mật khẩu?**  
Nếu không có salt, hai người dùng cùng mật khẩu sẽ có cùng hash → kẻ tấn công có thể dùng rainbow table (bảng hash tính sẵn) để tra ngược mật khẩu từ hash. Khi thêm salt ngẫu nhiên cho mỗi người dùng, cùng một mật khẩu sẽ cho ra hash khác nhau, vô hiệu hóa rainbow table và dictionary attack trên toàn bộ database.

**Vì sao SHA-256 demo trong lab chưa nên dùng trực tiếp cho hệ thống xác thực thật?**  
SHA-256 được thiết kế để tính **nhanh** — hiện nay GPU có thể tính hàng tỷ SHA-256/giây. Điều này tạo điều kiện cho brute-force attack. Các thuật toán chuyên dụng cho mật khẩu như **Argon2id**, **bcrypt**, hoặc **scrypt** được thiết kế để tốn nhiều CPU/memory, làm chậm đáng kể tốc độ brute-force. Ngoài ra, demo này không dùng constant-time comparison, có thể bị timing attack trong môi trường thực.
