# Hướng dẫn Reset password cho các máy ảo Boot từ volume trên hệ thống OpenStack phiên bản Mitaka
### Mục tiêu của hướng dẫn:
```
 - Hỗ trợ khách hàng có nhu cầu phục hồi password cho các máy ảo: Windows và Linux.
 - Do phiên bản Mitaka không hỗ trợ tính năng rescue cho VM boot từ volume, do đó sẽ phải dùng giải pháp khác để reset password VM
```

#### <i>Chú ý: </i>
 - Hướng dẫn thực hiện trên phiên bản OpenStack Mitaka, OS Ubuntu 14.04

#### Cách thức thực hiện: 
 - Chỉnh sửa DB của OpenStack để gỡ được volume boot khỏi máy ảo.
 - Gắn 1 volume khác có sẵn SystemRescueCD vào máy ảo .
 - Thực hiện reset password.
 - Gỡ volume SystemRescueCD khỏi máy ảo, chỉnh sửa lại DB OpenStack để đưa máy ảo về trạng thái ban đầu.
 - Boot máy ảo và đăng nhập bằng password mới.


## 1. Gỡ volume boot khỏi máy ảo
### 1.1. Kiểm tra danh sách máy ảo trên horizon
![resetpassword](/images/resetpassword/rp_9.png)

### 1.2. Kiểm tra danh sách volume trên horizon
![resetpassword](/images/resetpassword/rp_10.png)

### 1.3. Thực hiện tắt máy ảo cần reset password
![resetpassword](/images/resetpassword/rp_11.png)

### 1.4. Trên host Controller, đăng nhập vào MySQL và thực hiện update bản ghi của Volume boot của máy ảo cần chỉnh sửa
```sh
MariaDB [nova]> use nova;
MariaDB [nova]> update block_device_mapping set device_name = '/dev/vdl', boot_index=1 where volume_id = 'fe14c432-493e-42ca-970e-4b542fcf3cbf' and deleted = 0;       

MariaDB [nova]> use cinder;
MariaDB [cinder]> update volume_attachment set mountpoint='/dev/vdl' where volume_id ='fe14c432-493e-42ca-970e-4b542fcf3cbf' and deleted=0;
```
Trong đó:
 - `fe14c432-493e-42ca-970e-4b542fcf3cbf`: ID của Volume boot

Kết quả:
![resetpassword](/images/resetpassword/rp_12.png)

## 2. Tạo Volume SystemRescueCD
### 2.1. Tạo image SystemRescueCD
Thực hiện như hướng dẫn tại [đây](https://github.com/VNPT-SmartCloud-System/DongGoiImage_OpenStack/blob/master/docs/Huongdan_ResetPasswordVM_SystemRescueCD.md#1-systemrescuecd)

### 2.2. Tạo volume với image SystemRescueCD vừa tạo
![resetpassword](/images/resetpassword/rp_13.png)

### 2.3. Mount volume SystemRescueCD vào máy ảo
![resetpassword](/images/resetpassword/rp_15.png)

### 2.4. Đăng nhập vào MySQL và thực hiện update bản ghi của Volume SystemRescueCD
```sh
MariaDB [nova]> use nova;
MariaDB [nova]> update block_device_mapping set device_name = '/dev/vda', boot_index=1 where volume_id = '1c710e37-9eb1-49be-854b-e9c904a4fa59' and deleted = 0;       

MariaDB [nova]> use cinder;
MariaDB [cinder]> update volume_attachment set mountpoint='/dev/vda' where volume_id ='1c710e37-9eb1-49be-854b-e9c904a4fa59' and deleted=0;
```
Trong đó:
 - `1c710e37-9eb1-49be-854b-e9c904a4fa59`: ID của Volume SystemRescueCD

Kết quả:
![resetpassword](/images/resetpassword/rp_14.png)

### 2.5. Khởi động máy ảo


## 3. Thực hiện Reset password VM
Thực hiện như hướng dẫn tại [đây](https://github.com/longsube/DongGoiImage_OpenStack/blob/master/docs/Huongdan_ResetPasswordVM_SystemRescueCD.md#3-th%E1%BB%B1c-hi%E1%BB%87n-reset-password-vm)

### 3.1. Tắt máy ảo SystemRescueCD
```sh
shutdown -h now
```

### 3.2. Đăng nhập vào MySQL và thực hiện update lại bản ghi của Volume SystemRescueCD
```sh
MariaDB [nova]> use nova;
MariaDB [nova]> update block_device_mapping set device_name = '/dev/vdb', boot_index=1 where volume_id = '1c710e37-9eb1-49be-854b-e9c904a4fa59' and deleted = 0;       

MariaDB [nova]> use cinder;
MariaDB [cinder]> update volume_attachment set mountpoint='/dev/vdb' where volume_id ='1c710e37-9eb1-49be-854b-e9c904a4fa59' and deleted=0;
```
Trong đó:
 - `1c710e37-9eb1-49be-854b-e9c904a4fa59`: ID của Volume SystemRescueCD

### 3.3. Gỡ bỏ volume SystemRescueCD khỏi máy ảo

### 3.4. Trên host Controller, đăng nhập vào MySQL và thực hiện update lại bản ghi của Volume boot
```sh
MariaDB [nova]> use nova;
MariaDB [nova]> update block_device_mapping set device_name = '/dev/vda', boot_index=1 where volume_id = 'fe14c432-493e-42ca-970e-4b542fcf3cbf' and deleted = 0;       

MariaDB [nova]> use cinder;
MariaDB [cinder]> update volume_attachment set mountpoint='/dev/vda' where volume_id ='fe14c432-493e-42ca-970e-4b542fcf3cbf' and deleted=0;
```
Trong đó:
 - `fe14c432-493e-42ca-970e-4b542fcf3cbf`: ID của Volume boot

Kết quả:
![resetpassword](/images/resetpassword/rp_17.png)

### 3.4. Khởi động máy ảo và đăng nhập

## Done

# Tham khảo:
- https://bugs.launchpad.net/nova/+bug/1396965