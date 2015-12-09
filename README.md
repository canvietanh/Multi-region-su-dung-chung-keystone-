#Multi-region sử dụng chung Keystone trong OpenStack

##A. Mô hình cài đặt

<img src="http://i.imgur.com/mTUSYVD.png">


##B. Cài đặt

  Chúng ta sẽ thực hiện cài đặt OpenStak bản kilo bằng script


###I. Cài đặt trên region 1

####1. Các bước thực hiện trên cả 3 node

#####Bước 1:

Thực hiện chạy các lệnh sau để cập nhật và cài đặt OpenStack


    apt-get update
    apt-get -y install git

    git clone https://github.com/congto/openstack-kilo-multinode-U14.04-v1.git
    mv openstack-kilo-multinode-U14.04-v1/KILO-U14.04/ /root/
    rm -rf openstack-kilo-multinode-U14.04-v1
    cd KILO-U14.04
    chmod +x *.sh

#####Bước 2:
 Sửa filde config.cfg để khi cài đặt script ip sẽ trong OpenStack sẽ được cấu hình theo file  này. Đảm bảo những ip dùng chưa được cấp cho các máy nào trong dải mạng

    # Khai bao IP cho CONTROLLER NODE           
    CON_MGNT_IP=10.10.10.140
    CON_EXT_IP=10.145.25.140

    # Khai bao IP cho NETWORK NODE
    NET_MGNT_IP=10.10.10.141
    NET_EXT_IP=10.145.25.141
    NET_DATA_VM_IP=10.10.20.141

    # Khai bao IP cho COMPUTE1 NODE
    COM1_MGNT_IP=10.10.10.142
    COM1_EXT_IP=10.145.25.142
    COM1_DATA_VM_IP=10.10.20.142

    # Khai bao IP cho COMPUTE2 NODE
    COM2_MGNT_IP=10.10.10.74
    COM2_EXT_IP=192.168.1.74
    COM2_DATA_VM_IP=10.10.20.74

    GATEWAY_IP=10.145.25.1
    NETMASK_ADD=255.255.255.0

    # Set password
    DEFAULT_PASS='Welcome123'


####2. Thực hiện cài đặt
Thực hiện cài đặt trên region 1 theo hướng dẫn ở link sau:

    https://github.com/congto/openstack-kilo-multinode-U14.04-v1

 Sau khi cài đặt xong đăng nhập thử vào DashBoard của OpenStack kiểm tra thử tất cả các tính năng về tạo dải mạng, tạo máy ảo, nếu thành công là các bạn đã hoàn thành bước này.

####3. Sau khi cài đặt
Trên controller node chúng ta sẽ tạo Keystone dùng cho region 2 như sau

#####Bước 1: Tạo end point cho keystone

    openstack endpoint create \
    --publicurl http://10.145.25.140:5000/v2.0 \
    --internalurl http://10.145.25.140:5000/v2.0 \
    --adminurl http://10.145.25.140:35357/v2.0 \
    --region RegionTwo \
    identity
Đây là endpoint sử dụng cho region 2 với địa chỉ là public ip của controller trên region 1

#####Bước 2: Tạo endpoint cho glance

    openstack endpoint create \
    --publicurl http://10.145.25.160:9292 \
    --internalurl http://10.145.25.160:9292 \
    --adminurl http://10.145.25.160:9292 \
    --region RegionTwo \
    image

#####Bước 3: Tạo endpoint cho nova

    openstack endpoint create \
	--publicurl http://10.145.25.160:8774/v2/%\(tenant_id\)s \
	--internalurl http://10.145.25.160:8774/v2/%\(tenant_id\)s \
	--adminurl http://10.145.25.160:8774/v2/%\(tenant_id\)s \
	--region RegionTwo \
	compute

#####Bước 4: Tạo endpoint cho neutron

    openstack endpoint create \
    --publicurl http://10.145.25.160:9696 \
    --adminurl http://10.145.25.160:9696 \
    --internalurl http://10.145.25.160:9696 \
    --region RegionTwo \
    network

Sau khi tạo endpoint cho các dịch vụ ở Region 2, chúng ta chuyển sang cài đặt các node trên Region 2


###II. Cài đặt trên region 2
####1. Các bước thực hiện trên cả 3 node

#####Bước 1:

Thực hiện chạy các lệnh sau để cập nhật và cài đặt OpenStack


    apt-get update
    apt-get -y install git

    git clone https://github.com/congto/openstack-kilo-multinode-U14.04-v1.git
    mv openstack-kilo-multinode-U14.04-v1/KILO-U14.04/ /root/
    rm -rf openstack-kilo-multinode-U14.04-v1
    cd KILO-U14.04
    chmod +x *.sh

#####Bước 2:
 Sửa filde config.cfg để khi cài đặt script ip sẽ trong OpenStack sẽ được cấu hình theo file  này. Đảm bảo những ip dùng chưa được cấp cho các máy nào trong dải mạng

    # Khai bao IP cho CONTROLLER NODE           
    CON_MGNT_IP=20.20.20.140
    CON_EXT_IP=10.145.25.160

    # Khai bao IP cho NETWORK NODE
    NET_MGNT_IP=20.20.20.141
    NET_EXT_IP=10.145.25.161
    NET_DATA_VM_IP=20.20.30.141

    # Khai bao IP cho COMPUTE1 NODE
    COM1_MGNT_IP=20.20.20.142
    COM1_EXT_IP=10.145.25.162
    COM1_DATA_VM_IP=20.20.30.142

    # Khai bao IP cho COMPUTE2 NODE
    COM2_MGNT_IP=10.10.10.74
    COM2_EXT_IP=192.168.1.74
    COM2_DATA_VM_IP=10.10.20.74

    GATEWAY_IP=10.145.25.1
    NETMASK_ADD=255.255.255.0

    # Set password
    DEFAULT_PASS='Welcome123'


####2. Thực hiện cài đặt.
#####Bước 1: Trên controller node:

Thực hiện chạy những câu lệnh sau:

- Sửa file ctl-1-ipadd.sh

Sửa hostname cho máy

    Sửa dòng 34: controller thành contrller9

Sau đó thực hiện chạy script

    bash ctl-1-ipadd.sh

- Sửa file ctl2-prepare.sh

Tiếp tục sửa hostname
  
    Sửa dòng 13,14 thành controller9

    Sửa dòng 15 thành compute9

    Sửa dòng 17 thành network9

Sau khi sửa xong file tiếp tục chạy script này

    bash ctl-2-prepare.sh

- Sửa file ctl-3-create-db.sh

Vì cta ko cài keystone trên controller node này nên bỏ phần tạo database đi
     
    Bỏ toàn bộ phần tạo database cho keystone đi

Chạy script
     
    bash ctl-3-create-db.sh

Sau khi cài đặt xong bước này ta sẽ tạo 1 file biến môi trường như sau:

        vi admin-openrc.sh
        export OS_PROJECT_DOMAIN_ID=default
        export OS_USER_DOMAIN_ID=default
        export OS_PROJECT_NAME=admin
        export OS_TENANT_NAME=admin
        export OS_USERNAME=admin
        export OS_PASSWORD=Welcome123
        export OS_AUTH_URL=http://10.145.25.140:35357/v3
        export OS_VOLUME_API_VERSION=2
        export OS_REGION_NAME=RegionTwo




- Sửa file ctl-6-glance.sh

Sửa dòng 68, 69 lần lượt thành

      auth_uri = http://10.145.25.140:5000/v2.0
      auth_url = http://10.145.25.140:35357 

Sửa dòng 165,166 thành

      auth_uri = http://10.145.25.140:5000/v2.0
      auth_url = http://10.145.25.140:35357 

Cài đặt project glance

      bash ctl-6-glance.sh

- Tiếp tục thực hiện sửa file ctl-7-nova.sh

Sửa dòng 61,62 thành

     auth_uri = http://10.145.25.140:5000/v2.0
     auth_url = http://10.145.25.140:35357

Sửa dòng 77 thành

    url = http://10.145.25.160:9696

Sửa dòng 79 thành
 
    admin_auth_url = 10.145.25.140:35357/v2.0
Sau đó tiến hành cài đặt project nova
  
     bash ctl-7-nova.sh

- Cuối cùng sửa file tạo ctl-8-neutron.sh
 
Sửa dòng 41 thành:
    
     nova_url = http://10.145.25.160:8774/v2     

Sửa dòng 51,52 thành:

     auth_uri = http://10.145.25.140:5000/v2.0
     auth_url = http://10.145.25.140:35357
       
Sửa dòng 67 thành:

    auth_url = http://10.145.25.140:35357
Sửa dòng 71 thành:
    
    region_name= RegionTwo

Thực hiện tạo neutron
    
    bash ctl-8-neutron.sh

 Kết thúc cài đặt trên controller

#####Bước 2: Trên network node
- Tiến hành sửa file net-ipadd.sh

Đặt lại hostname cho máy

    Sửa dòng 88 network thành network9

Chạy file:
    
    bash net-ipadd.sh

- Sau khi chạy xong, tiến hành sửa file net-prepare.sh

Thay đổi về đặt tên các node cũng như các file cấu hình

    Dòng 13 thành network9
    Dòng 14 thành controller9
    Dòng 15 thành compute9
    Dòng 17 thành network9
    

Dòng 64,65 thành:
    
    auth_uri = http://10.145.25.140:5000/v2.0
    auth_url = http://10.145.25.140:35357 

Dòng 142,143 thành:

    auth_uri = http://10.145.25.140:5000/v2.0
    auth_url = http://10.145.25.140:35357 

- Tiến hành thực thi file

Chạy file prepare để tiến hành cài đặt

    bash net-prepare.sh

 Kết thúc cài đặt trên neutron

#####Bước 3: Chạy trên compute node

- Sửa file com1-ipadd.sh

Sửa file để cài đặt lại hostname

    Sửa dòng 14 thành compute9


- Thực thi file

Chạy file để cấu hình lại hostname

     bash com1-ipadd.sh

- Sửa tiếp file com1-prepare

Sửa file để cbi cài đặt compute

    Sửa dòng 13 thành compute9
    Sửa dòng 14 thành controller9
    Sửa dòng 15 thành compute9
    Sửa dòng 17 thành network9

Sửa dòng 117 thành
     
    url = http://10.145.25.160:9696

Sửa dòng 119 thành:

    admin_auth_url = 10.145.25.140:35357/v2.0


Sửa dòng 125,126 lần lượt thành: 
    
    auth_uri = http://10.145.25.140:5000/v2.0
    auth_url = http://10.145.25.140:35357 

Sửa dòng 186,187 thành:

    auth_uri = http://10.145.25.140:5000/v2.0
    auth_url = http://10.145.25.140:35357 

 Kết thúc cài đặt trên compute


##C. Tiến hành kiểm tra

Toàn bộ việc cài đặt trên region 2 chủ yêu là để tránh việc hostname ko trùng với region1 tránh việc rabbitMQ ko hiểu và dẫn đến truyền thông nhầm, cùng với việc thay đổi địa chỉ endpoint đc cấu hình trc trong các file script để trỏ về dúng region 2

<img src="http://i.imgur.com/xF0Q3yx.png">
