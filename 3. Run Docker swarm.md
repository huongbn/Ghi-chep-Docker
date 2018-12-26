# Khởi tạo Docker swarm

# Mục lục
- [1. Khởi tạo swarm](#1)
    - [1.1 Config default address pools](#11)
    - [1.2 Config advertise address](#12)
- [2. Drain node](#2)

<a name="1"></a>

# 1. Khởi tạo Docker swarm

- Thực hiện chạy câu lệnh:
```
docker swarm init
```

Sau khi thực hiện câu lệnh trên, output sẽ như sau:

<a name="11"></a>

## 1.1 Config default address pools

Mặc định, Docker swarm sử dụng default adress pool là `10.0.0.0/8` cho mode `overlay`. Mọi subnet khác không được chỉ định rõ trong swarm sẽ sử dụng subnet default này làm mặc định.

Trong trường hợp default subnet này `10.0.0.0/8` xảy ra xung đột với một network khác trong hệ thống, ta phải sử dụng tùy chọn khai báo `--subnet` để có thể tránh xảy ra tình trạng này.

Để cấu hình một default address pool, ta định nghĩa pool này sử dụng `--default-addr-pool`. Ví dụ dưới đây khai báo 1 default address pool có subnetmask là `/16`. Trong trường hợp `--default-addr-pool-mask-len` không được nêu rõ, thì mặc định sẽ là `/24`
- Ví dụ:
```
docker swarm init --default-addr-pool 10.0.0.0/16
```
Khởi tạo 1 default address pool với subnetmask là `/16` cho 2 subnet là `10.20.0.0` và `10.30.0.0`, subnetmask cho mỗi mạng là `/26`
```
docker swarm init --default-addr-pool 10.20.0.0/16 --default-addr-pool 10.30.0.0/16 --default-addr-pool-mask-length 26
```

<a name="12"></a>

## 1.2 Config advertise address
Manager nodes sử dụng advertise address để cho phép các nodes khác join vào swarm. 

```
docker swarm init --advertise-addr <MANAGER-IP>
```

Kiểm tra token trên worker và manager node. Token trên các nodes này phải khác nhau.
- Trên worker nodes, sử dụng command:
```
docker swarm join-token worker
```

Kết quả:
```
To add a worker to this swarm, run the following command:

    docker swarm join \
    --token SWMTKN-1-49nj1cmql0jkz5s954yi3oex3nedyz0fb0xx14ie39trti4wxv-8vxv8rssmk743ojnwacrr2e7c \
    192.168.99.100:2377

This node joined a swarm as a worker.
```

- Trên manager nodes, sử dụng command:
```
docker join-token manager
```
 Kết quả:
 ```
 To add a worker to this swarm, run the following command:

    docker swarm join \
    --token SWMTKN-1-59egwe8qangbzbqb3ryawxzk3jn97ifahlsrw01yar60pmkr90-bdjfnkcflhooyafetgjod97sz \
    192.168.99.100:2377
```

*Lưu ý: Sử dụng tùy chọn `--quiet` để chỉ in ra token
*Lưu ý: Các token là rất quan trọng, nó là key để có thể cho phép một node khác join vào swarm với vai trò là worker hay là manager. Để tăng tính an toàn, chúng ta nên sử dụng `rotate` để có thể xoay vòng token, khuyến nghị nên xoay vòng 6 tháng 1 lần trên `worker` hoặc `manager` nodes.
- Ví dụ:
```
docker swarm join-token --rotate worker
```

Kết quả:
```
To add a worker to this swarm, run the following command:

    docker swarm join \
    --token SWMTKN-1-2kscvs0zuymrsc9t0ocyy1rdns9dhaodvpl639j2bqx55uptag-ebmn5u927reawo27s3azntd44 \
    192.168.99.100:2377
```

<a name="2"></a>

# 2. Drain node
Sử dụng `Drain` khi muốn node nào đó không nhận task hoặc phục vụ công việc bảo trị
- Cú pháp:
```
docker node update --availability drain <NODE>
```



