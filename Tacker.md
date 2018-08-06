### Tacker

Tacker là một dịch vụ về Network Function Virtualization (NFV), quản lý Virtual Network Functions (VNF) chạy trên nền tảng đám mây OpenStack. Ưu điểm là nhỏ gọn, dễ sử dụng, có thể cài đặt trên cùng 1 máy (All in one) đầy đủ với hệ thống OpenStack bằng DevStack. Nhược điểm là chưa đầy đủ chức năng và toàn vẹn nếu so sánh với ONAP, OpenBaton, OPNFV.





### Cài đặt

Tạo databases:

``` sh
mysql -uroot -pWelcome123
CREATE DATABASE tacker;
grant all privileges on tacker.* to 'tacker'@'%' identified by 'Welcome123';
grant all privileges on tacker.* to 'tacker'@'localhost' identified by 'Welcome123';
flush privileges;
```
Tạo user tacker, gán role admin và user tacker và tạo endpoint:


``` sh
openstack user create --domain default --password Welcome123 tacker
openstack role add --project services --user tacker admin

openstack service create --name tacker \
           --description "Tacker Project" nfv-orchestration
           
openstack endpoint create --region RegionOne nfv-orchestration \
           public http://192.168.239.190:9890/
openstack endpoint create --region RegionOne nfv-orchestration \
           internal http://192.168.239.190:9890/
openstack endpoint create --region RegionOne nfv-orchestration \
           admin http://192.168.239.190:9890/
yum install -y openstack-tacker openstack-tacker-common \
            puppet-tacker python-tacker python2-tackerclient
```
Tải project Tacker:

`git clone https://github.com/openstack/tacker`

Cài đặt các packages yêu cầu:

``` sh
cd tacker
pip install -r requirements.txt
```

```
cd tacker
pip install tosca-parser
```
Cài đặt Tacker: `python setup.py install`

Tạo đường dẫn file log: `mkdir /var/log/tacker`


Chỉnh sửa file tacker.conf : `vi /etc/tacker/tacker.conf`

``` sh			

[DEFAULT]

auth_strategy = keystone
policy_file = /etc/tacker/policy.json
debug = True
use_syslog = False
bind_host = 192.168.239.190
bind_port = 9890
service_plugins = nfvo,vnfm
state_path = /var/lib/tacker

[nfvo]
vim_drivers = openstack
[keystone_authtoken]
memcached_servers = 192.168.239.190:11211
region_name = RegionOne
auth_type = password
project_domain_name = Default
user_domain_name = Default
username = tacker
project_name = services
password = tacker
auth_url = http://192.168.239.190:5000
auth_uri = http://192.168.239.190:5000
[agent]
root_helper = sudo /usr/bin/tacker-rootwrap /etc/tacker/rootwrap.conf
[database]
connection = mysql://tacker:Welcome123@192.168.239.190:3306/tacker
[tacker]
monitor_driver = ping,http_ping
```
Đồng bộ dữ liệu:

`tacker-db-manage --config-file /etc/tacker/tacker.conf upgrade head`

Cài đặt Tacker client:

`git clone https://github.com/openstack/python-tackerclient`

``` sh
cd python-tackerclient
sudo python setup.py install
```





Cấu hình dashboard:

`git clone https://github.com/openstack/tacker-horizon`

```
cd tacker-horizon
sudo python setup.py install
```

`cp tacker_horizon/enabled/*     /usr/share/openstack-dashboard/openstack_dashboard/enabled/`

Khởi động lại dịch vụ:

```
systemctl restart httpd
systemctl restart openstack-tacker-server.service
systemctl enable  openstack-tacker-server.service
```










































































