# Một vài ghi chép về Docker-compose v3

## Mục lục

- [1. deploy key](#1)

<a name="1">></a>

# 1. deploy key
- Cho phép cấu hình thông số cho services
- Ví dụ:
```
deploy:
  mode: replicated
  replicas: 5
  update_config:
    parallelism: 2
    delay: 10s
  restart_policy:
    condition: on-failure
```

- Trong ví dụ trên:
    - mode:
        - global:
        - replicated: Chỉ trên 1 số container, mặc định mode ở chế độ replicated
    - placement:
        - Chỉ định 1 số ràng buộc về vị trí
        - Ví dụ:
```
        placement:
  constraints:
    - node.role == manager
    - engine.labels.operatingsystem == ubuntu 16.04
```

- update_config:
    - Cập nhật thông số muốn config
        - parallelism: Chỉ định số container được update
        - delay: Thời gian delay update giữa các group
        - failure_action: Chỉ định sẽ làm gì nếu việc update fails. Có thể là `continue` hoặc `pause`. Mặc định là `pause`
        - monitor: Duration after each task update to monitor for failure (ns|us|ms|s|m|h). Mặc định 0s
        - max_failure_ratio: Failure rate to tolerate diuring an update
        - oder: support v3.4 or higher
    - Ví dụ:
```
update_config: 
  parallelism: 2 
  delay: 10s
```


- resources:
    - Để config resources constraints 
    - Ví dụ:
```
resources:
  limits:
    cpus: '1.5'
    memory: 500M
  reservations:
    cpus: '1.0'
    memory: 200M
```

- restart_policy:
    - Để config khi nào container restart nếu nó thoát
    - Có 4 dạng:
        - `condition`: Có thể là `none`, `on-failure` hoặc `any`. Mặc định `any`
        - `delay`: Chỉ định thời gian delay restart. Mặc định là 0
        - `max_attempts`: Bao nhiêu lần restart lại container trước khi bỏ cuộc. Mặc định: never give up
        - `window`: How long to wait before deciding if a restart has succeeded. Mặc định là: decide immediately
    - Ví dụ:
```
restart_policy: 
   condition: on-failure 
   delay: 5s 
   max_attempts: 3 
   window: 120s
```

- labels: 
    - Thiết lập label cho service
    - Labels chỉ thiết lập cho service, container không bị ảnh hưởng
    - Ví dụ:
```
version: "3"
services:
  web:
    image: web
    deploy:
      labels:
        com.example.description: "This label will appear on the web service"
```

Muốn set label cho container, pahir đặt `labels` ra ngoài `deploy`:

```
version: "3"
services:
  web:
    image: web
    labels:
      com.example.description: "This label will appear on all containers for the web service"
```

- depends_on:
    - Chỉ ra sự phụ thuộc giữa các services
    - Ví dụ:
```
version: '3'
services:
  web:
    build: .
    depends_on:
      - db
      - redis
  redis:
    image: redis
  db:
    image: postgres
```

- dns:
    - Custom DNS server. Có thể là signle DNS hoặc list DNS
    - Ví dụ:
```
dns: 8.8.8.8
dns:
  - 8.8.8.8
  - 9.9.9.9
```

- image: Chỉ định image được sử dụng để start container

Ví dụ:
```
image: redis
image: ubuntu:14.04
image: tutum/influxdb
image: example-registry.com:4000/postgresql
image: a4bc65fd
```



