# Tìm hiểu về Docker Swarm

## Mục lục
- [1. Docker swarm là gì?](#1)
- [2. Tính năng nổi bật của Docker Swarm](#2)
- [3. Mô hình của Docker swarm](#3)
    - [3.1 Các node trong swarm là gì và cách thức làm việc](#31)
- [4. Service và Task trong Docker swarm](#4)
- [5. Tính năng nổi bật](#5)
    - [5.1 Service Discovery](#51)
    - [5.2 Load Balancing](#52)
    - [5.3 High Avalibity](#53)
    - [5.4 Scheduling](#54)

<a name="1"></a>

# 1. Docker swarm là gì?

Docker swarm là một nhóm các máy chạy Docker và tập hợp lại với nhau thành một cluster. Không như docker engine, sau khi các máy này tập hợp vào Swarm, mõi câu lệnh Docker sẽ được thực thi trên Swarm manager.

Các máy tham gia vào swarm được gọi là worrker node. Các node này chỉ có khả năng cung cấp khả năng hoạt động chứ không có quyền quản lý các node khác.

Docker Swarm có khả năng khởi chạy các container trên nhiều máy(máy ảo hoặc máy vật lý) hoặc trên một máy duy nhất.

Docker Swarm là một phần mềm hỗ trợ việc tạo và quản lý các container hoặc các hệ thống container orchestration. Nó là một cluster nơi mà người dùng quản lý các Docker engine hoặc các node nơi mà các service được deploy và chạy.

Ngoài ra Docker swarm còn có những tính năng để hỗ trợ việc quản lý các container khi chạy trên môi trường phân tán và để chắc chắn container trong một cluster hoạt động ổn định.

<a name="2"></a>

# 2. Tính năng nổi bật của Docker Swarm
Docker swarm cung cấp cho chúng ta rất nhiều tính năng nổi bật ví dụ như:
- Quản lý cluster với Docker engine
- Thiết kế phân cấp
- Scaling
- Multi-host networking
- Service Discovery
- Load Balancing
- Bảo mật
- Rolling updates

<a name="3"></a>

# 3. Mô hình của Docker swarm

![Imgur](https://i.imgur.com/4wrW8WL.jpg)

![Imgur](https://i.imgur.com/q424vEr.png)

<a name="31"></a>

## 3.1 Các node trong swarm là gì và cách thức làm việc

Các node trong Docker swarm là các máy chạy Docker engine được join vào swarm. Các node này có thể được gọi là Docker node.

Để có thể deploy một ứng dụng trên swarm, người dùng cần phải định nghĩa các service đến manager node. Manager node này sẽ chuyển service này thành các task và đưa xuống các node bên dưới để thực thi.

Docker swarm còn được coi là một dạng native cluster do các node trong cluster kể cả là manager node cũng có thể hoạt động như một worker node. Các worker node được giao task từ manager node sẽ thông báo về cho manager tình trạng của task đang chạy trên node đó thông qua một `agent` ở mỗi node để cho manager node có thể đảm bảo trạng thái của mỗi node bên dưới nó.

![Imgur](https://i.imgur.com/U8ej1z1.jpg)

<a name="4"></a>

# 4. Service và Task trong Docker swarm

Service trong Docker swarm đảm nhận chức năng định danh các task hoạt động cho manager node hoặc các worker node.

Ngoài docker-CLI thì các service này cũng có thể được deploy bằng Docker-compose.

Khi khởi tạo một service trong swarm mode, người dùng sẽ xác định container image nào sẽ được sử dụng và câu lệnh nào sẽ chạy bên trong container đó. Nó có khả năng sắp xếp, quản lý container của swarm.

Các container images được chia sẻ và upload lên Dockerhub.

Một task có nhiệm vụ xác định Docker container và câu lệnh sẽ được chạy bên trong container đó.

Node manager sẽ đưa các task này cho các worker node theo số lượng replicas đưcọ định nghĩa trong service scale. Khi một task được đưa cho một worker node, nó sẽ không thể di chuyển sang node khác, nó chỉ có thể trên node được manager node chỉ định hoặc `fail`.

Ví dụ service `nginx` được replicas thành 3 container:

![Imgur](https://i.imgur.com/tUzibyT.jpg)

Có 2 kiểu service là:
- **Replicated service**: Ở chế độ này thì swarm manager sẽ xác định số lượng các bản sao mà một task của một service sẽ được chạy trên các nodes dựa theo sự mong muốn của người dùng.
- **Global service**: Ở chế độ này, swarm manager sẽ chạy một task cho một service trên tất cả các node có khả năng chạy được trong cluster.

Ngoài ra, nếu người dùng tạo một service mà trong cluster hiện tại chưa có node nào có kahr năng chạy service đó thì service đó sẽ được đưa về chế độ Pending Service.

Service có thể bị Pending mãi mãi nếu không có node nào trong cluster có khẳ năng thỏa mãn yêu cầu sử dụng của service.

<a name="5"></a>

# 5. Tính năng nổi bật

<a name="51"></a>

## 5.1 Service Discovery

Service Discovery là một cơ chế mà Docker sử dụng để định tuyến lại request từ một client bên ngoài đến một node riêng lẻ bên trong swarm mà client không cần biết có bao nhiêu node đang tham gia vào service hoặc IP hoặc port.

Service Discovery có thể hoạt động theo 2 cách khác nhau, sử dụng V-IP(Virtual IP) hoặc DNS round robin(DNSRR).

![Imgur](https://i.imgur.com/V2farLx.png)

Mặc định docker swarm sử dụng VIP. Service Discovery trong Docker swarm được thực hiện như sau:
- 1. Tạo một overlay network
- 2. Tạo một service gán nó vào network vừa được tạo
- 3. Swarm gán VIP và DNS entry cho mỗi service
- 4. VIP sẽ map đến DNS dựa theo service name
- 5. Container chia sẻ DNS mapping cho service thông qua GOSSIP
- 6. Bất kì container nào trong network cũng có thể kết nối tới service thông qua service name.

<a name="52"></a>

## 5.2 Load Balancing

Swarm manager sử dụng network ingress load balancing để đưa các service ra bên ngoài swarm. Swarm manager có thể tự động đưa các service lên các Port chưa được sử dụng trong khoảng 30000-32767. Hoặc người dùng cũng co thể tự động xác định port mà service sẽ được gán vào.

Ccác ứng dụng bên ngoài có thể truy cập vào service thông qua các Publish Port ở bất kì node nào trong cluster kể cả node đó có service này hay không.

Tất cae các node trong Swarm đều sẽ kết nối đến route ingress để kết nối đến các task đang hoạt động.

Bên trong Docker swarm mode, mỗi một service đều sẽ được gán cho một DNS nội bộ để truy cập. Swarm manager sẽ sử dụng khả năng load balancing nội bộ để gửi các yêu cầu đến các service trong cluster dựa theo DNS của service đó.
