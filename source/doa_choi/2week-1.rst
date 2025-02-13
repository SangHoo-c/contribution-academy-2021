============================
2주차 과제 - CLI와 친해지기
============================

00. 서버 정보
------------------------------

**Server**: Ubuntu 18.04v (LTS)

**Openstack Version**: 5.5.0v

|

01. Cirros images CLI 인스턴스 생성
--------------------------------------------------
현재 오픈스택 서비스 서버에 할당한 프로바이더 네트워크는 192.168.100.0/24 대역입니다.
이 네트워크 대역을 설명하기 이전에 ISP에 대해서 이야기를 하자면, ISP는 Internet service provider 의 약자로 인터넷 서비스 제공자를 의미합니다.
ISP, 인터넷 서비스 제공업체들은 네트워크에 접속할 수 있는 환경을 제공해줍니다.

이와 비슷하게 오픈스택의 프로바이더 네트워크는 인터넷에 연결이 되어 있는 네트워크로, 오픈스택에서 생성한 가상 인스턴스가 외부에서 액세스할 수 있도록 호스트 아이피를 할당해줄 수 있습니다.

이 네트워크는 현재 시스템에서 public 에 해당됩니다. 다음으로 오픈스택 구축시 할당한 사설 네트워크 대역은 10.0.0.0/26 입니다. 이 네트워크 대역은 private 에 해당되며, 오픈스택에서는 이를 Self-Service Network 라고 합니다. 인스턴스 생성시 해당 네트워크 대역의 호스트 아이피로 동적 할당됩니다.



 .. code-block:: none

     stack@doa-wallaby:~/devstack$ openstack network list
     +--------------------------------------+---------+--------------------------------------------
     --------------------------------+
     | ID                                   | Name    | Subnets                                                                    |
     +--------------------------------------+---------+----------------------------------------------------------------------------+
     | 75ef8908-d335-471a-a561-5de378cb0371 | public  | 77aa158c-e0fc-4adb-8337-0e7ca30bc275, f8b95048-723b-44f8-9cfb-4ec283669e18 |
     | b1d129cc-ca87-445b-8880-0e2923c47cd6 | shared  | 7ea5ea69-162b-4a1f-8973-708663a7fc16                                       |
     | dacb03f0-4ed9-43cc-82ae-10df1246556f | private | 30754a5c-dd3a-4363-a786-86bfd7e2d7a0, e1d9f47b-c0b0-4f9c-abf3-d6619640622e |
     +--------------------------------------+---------+--------------------------------------------
    --------------------------------+

     stack@doa-wallaby:~/devstack$ openstack subnet list
     +--------------------------------------+---------------------+-------------------------------- ------+---------------------+
     | ID                                   | Name                | Network
     | Subnet              |
     +--------------------------------------+---------------------+-------------------------------- ------+---------------------+
     | 30754a5c-dd3a-4363-a786-86bfd7e2d7a0 | ipv6-private-subnet | dacb03f0-4ed9-43cc-82ae-
     10df1246556f | fd33:b9ad:a660::/64 |
     | 77aa158c-e0fc-4adb-8337-0e7ca30bc275 | ipv6-public-subnet  | 75ef8908-d335-471a-a561- 5de378cb0371 | 2001:db8::/64       |
     | 7ea5ea69-162b-4a1f-8973-708663a7fc16 | shared-subnet       | b1d129cc-ca87-445b-8880- 0e2923c47cd6 | 192.168.233.0/24    |
     | e1d9f47b-c0b0-4f9c-abf3-d6619640622e | private-subnet      | dacb03f0-4ed9-43cc-82ae- 10df1246556f | 10.0.0.0/26         |
     | f8b95048-723b-44f8-9cfb-4ec283669e18 | public-subnet       | 75ef8908-d335-471a-a561- 5de378cb0371 | 192.168.100.0/24    |
     +--------------------------------------+---------------------+-------------------------------- ------+---------------------+

|

 2. 각 네트워크 대역의 정보는 다음과 같습니다.

 .. code-block:: none

     stack@doa-wallaby:~/devstack$ openstack subnet show f8b95048-723b-44f8-9cfb-4ec283669e18
     +----------------------+--------------------------------------+
     | Field                | Value                                |
     +----------------------+--------------------------------------+
     | allocation_pools     | 192.168.100.50-192.168.100.250       |
     | cidr                 | 192.168.100.0/24                     |
     | created_at           | 2021-08-19T12:08:12Z                 |
     | description          |                                      |
     | dns_nameservers      |                                      |
     | dns_publish_fixed_ip | None                                 |
     | enable_dhcp          | False                                |
     | gateway_ip           | 192.168.100.1                        |
     | host_routes          |                                      |
     | id                   | f8b95048-723b-44f8-9cfb-4ec283669e18 |
     | ip_version           | 4                                    |
     | ipv6_address_mode    | None                                 |
     | ipv6_ra_mode         | None                                 |
     | name                 | public-subnet                        |
     | network_id           | 75ef8908-d335-471a-a561-5de378cb0371 |
     | prefix_length        | None                                 |
     | project_id           | 4d6190dd59524a5e8247f70566a71c1b     |
     | revision_number      | 0                                    |
     | segment_id           | None                                 |
     | service_types        |                                      |
     | subnetpool_id        | None                                 |
     | tags                 |                                      |
     | updated_at           | 2021-08-19T12:08:12Z                 |
     +----------------------+--------------------------------------+


     stack@doa-wallaby:~/devstack$ openstack subnet show e1d9f47b-c0b0-4f9c-abf3-d6619640622e
     +----------------------+--------------------------------------+
     | Field                | Value                                |
     +----------------------+--------------------------------------+
     | allocation_pools     | 10.0.0.2-10.0.0.62                   |
     | cidr                 | 10.0.0.0/26                          |
     | created_at           | 2021-08-19T12:07:52Z                 |
     | description          |                                      |
     | dns_nameservers      |                                      |
     | dns_publish_fixed_ip | None                                 |
     | enable_dhcp          | True                                 |
     | gateway_ip           | 10.0.0.1                             |
     | host_routes          |                                      |
     | id                   | e1d9f47b-c0b0-4f9c-abf3-d6619640622e |
     | ip_version           | 4                                    |
     | ipv6_address_mode    | None                                 |
     | ipv6_ra_mode         | None                                 |
     | name                 | private-subnet                       |
     | network_id           | dacb03f0-4ed9-43cc-82ae-10df1246556f |
     | prefix_length        | None                                 |
     | project_id           | d8c257ebe8b04e869a00434ae6665f3c     |
     | revision_number      | 0                                    |
     | segment_id           | None                                 |
     | service_types        |                                      |
     | subnetpool_id        | 5a1e8482-c944-423d-8f68-50cbe0c9a7d1 |
     | tags                 |                                      |
     | updated_at           | 2021-08-19T12:07:52Z                 |
     +----------------------+--------------------------------------+

|

 3. 인스턴스를 생성하기 위해서 각 인스턴스에 할당 가능한 리소스 정보를 확인합니다.
 첫번째 인스턴스는 cirrOS 운영체제로, 해당 OS는 Nova에서 테스트 이미지로 사용되는 최소한의 리눅스 배포판입니다.


 .. code-block:: none

    stack@doa-wallaby:~/devstack$ openstack flavor list
    +----+-----------+-------+------+-----------+-------+-----------+
    | ID | Name      |   RAM | Disk | Ephemeral | VCPUs | Is Public |
    +----+-----------+-------+------+-----------+-------+-----------+
    | 1  | m1.tiny   |   512 |    1 |         0 |     1 | True      |
    | 2  | m1.small  |  2048 |   20 |         0 |     1 | True      |
    | 3  | m1.medium |  4096 |   40 |         0 |     2 | True      |
    | 4  | m1.large  |  8192 |   80 |         0 |     4 | True      |
    | 42 | m1.nano   |   128 |    1 |         0 |     1 | True      |
    | 5  | m1.xlarge | 16384 |  160 |         0 |     8 | True      |
    | 84 | m1.micro  |   192 |    1 |         0 |     1 | True      |
    | c1 | cirros256 |   256 |    1 |         0 |     1 | True      |
    | d1 | ds512M    |   512 |    5 |         0 |     1 | True      |
    | d2 | ds1G      |  1024 |   10 |         0 |     1 | True      |
    | d3 | ds2G      |  2048 |   10 |         0 |     2 | True      |
    | d4 | ds4G      |  4096 |   20 |         0 |     4 | True      |
    +----+-----------+-------+------+-----------+-------+-----------+

|

 4. 사용 가능한 이미지 목록을 확인합니다. 오픈스택에서는 기본적으로 cirros 이미지를 제공합니다.
 하기의 이미지 리스트로 조회된 cirros VM 이미지로 인스턴스를 생성합니다.


 .. code-block:: none

     stack@doa-wallaby:~/devstack$ openstack image list
    +--------------------------------------+--------------------------+--------+
    | ID                                   | Name                     | Status |
    +--------------------------------------+--------------------------+--------+
    | 64436365-443b-412b-bb20-07492191e3c4 | cirros-0.5.2-x86_64-disk | active |
    +--------------------------------------+--------------------------+--------+

|

 5. 인스턴스 생성시 사용할 보안 정책을 확인합니다.
 방화벽 및 디폴트 포트 정보는 다음 URL 에서 확인이 가능합니다.
 *URL : https://docs.openstack.org/ko_KR/install-guide/firewalls-default-ports.html*

 .. code-block:: none

    stack@doa-wallaby:~/devstack$ openstack security group list
    +--------------------------------------+---------+------------------------+----------------------------------+------+
    | ID                                   | Name    | Description            | Project                          | Tags |
    +--------------------------------------+---------+------------------------+----------------------------------+------+
    | 6bdec141-e1f3-4e2f-8264-e4783b75ca01 | default | Default security group | 4d6190dd59524a5e8247f70566a71c1b | []   |
    | bc0298a1-d50e-4b34-8897-79170e2a7522 | default | Default security group | d8c257ebe8b04e869a00434ae6665f3c | []   |
    +--------------------------------------+---------+------------------------+----------------------------------+------+

|

 6. 인스턴스에 접근하기 위한 키페어를 생성합니다. 생성할 인스턴스에 접근하기 위한 공개키 파일명의 이름은 mykey 입니다.


 .. code-block:: none

    stack@doa-wallaby:~/devstack$ openstack keypair create --public-key ~/.ssh/id_rsa.pub mykey
    +-------------+-------------------------------------------------+
    | Field       | Value                                           |
    +-------------+-------------------------------------------------+
    | created_at  | None                                            |
    | fingerprint | 32:7e:f1:a3:9a:c7:3c:77:b0:67:69:bb:82:24:52:4a |
    | id          | mykey                                           |
    | is_deleted  | None                                            |
    | name        | mykey                                           |
    | type        | ssh                                             |
    | user_id     | 29b9101dc52844f5bcd6874fc7a1b4ae                |
    +-------------+-------------------------------------------------+

|

 7. 인스턴스 생성에 필요한 모든 정보를 수집하였으므로, 인스턴스를 생성합니다.
 인스턴스를 생성하기 위해 필요한 정보는 다음과 같습니다.


 - 인스턴스에 할당할 서버 사양 정보

 - 인스턴스의 운영체제

 - 인스턴스가 호스트 아이피를 할당받을 네트워크 정보

 - 보안 그룹 정보

 - 인스턴스로 액세스하기 위한 키페어

 - 인스턴스명


 .. code-block:: none

     stack@doa-wallaby:~/devstack$ openstack server create --flavor m1.tiny --image cirros-0.5.2-x86_64-disk \
    --nic net-id=dacb03f0-4ed9-43cc-82ae-10df1246556f --security-group 6bdec141-e1f3-4e2f-8264-e4783b75ca01 \
    --key-name mykey private-instance-prac01

|


 8. 위 명령으로 생성된 인스턴스의 활성 상태와 정보는 다음과 같습니다.

 .. code-block:: none

    stack@doa-wallaby:~/devstack$ openstack server list
    +--------------------------------------+-------------------------+--------+---------------------------------------------------------+--------------------------+---------+
    | ID                                   | Name                    | Status | Networks                                                | Image                    | Flavor  |
    +--------------------------------------+-------------------------+--------+---------------------------------------------------------+--------------------------+---------+
    | acbf1669-0ed9-46fb-b38b-8769569d27a3 | private-instance-prac01 | ACTIVE | private=10.0.0.14, fd33:b9ad:a660:0:f816:3eff:fe36:8a90 | cirros-0.5.2-x86_64-disk | m1.tiny |
    +--------------------------------------+-------------------------+--------+---------------------------------------------------------+--------------------------+---------+

    stack@doa-wallaby:~/.ssh$ openstack server show acbf1669-0ed9-46fb-b38b-8769569d27a3
    +-------------------------------------+-----------------------------------------------------------------+
    | Field                               | Value                                                           |
    +-------------------------------------+-----------------------------------------------------------------+
    | OS-DCF:diskConfig                   | MANUAL                                                          |
    | OS-EXT-AZ:availability_zone         | nova                                                            |
    | OS-EXT-SRV-ATTR:host                | doa-wallaby                                                     |
    | OS-EXT-SRV-ATTR:hypervisor_hostname | doa-wallaby                                                     |
    | OS-EXT-SRV-ATTR:instance_name       | instance-00000001                                               |
    | OS-EXT-STS:power_state              | Running                                                         |
    | OS-EXT-STS:task_state               | None                                                            |
    | OS-EXT-STS:vm_state                 | active                                                          |
    | OS-SRV-USG:launched_at              | 2021-08-19T14:21:21.000000                                      |
    | OS-SRV-USG:terminated_at            | None                                                            |
    | accessIPv4                          |                                                                 |
    | accessIPv6                          |                                                                 |
    | addresses                           | private=10.0.0.14, fd33:b9ad:a660:0:f816:3eff:fe36:8a90         |
    | config_drive                        |                                                                 |
    | created                             | 2021-08-19T14:21:12Z                                            |
    | flavor                              | m1.tiny (1)                                                     |
    | hostId                              | a7cfaf95007ab1386481f7ca38819729a0a266a4e72690140c62f66b        |
    | id                                  | acbf1669-0ed9-46fb-b38b-8769569d27a3                            |
    | image                               | cirros-0.5.2-x86_64-disk (64436365-443b-412b-bb20-07492191e3c4) |
    | key_name                            | mykey                                                           |
    | name                                | private-instance-prac01                                         |
    | progress                            | 0                                                               |
    | project_id                          | 4d6190dd59524a5e8247f70566a71c1b                                |
    | properties                          |                                                                 |
    | security_groups                     | name='default'                                                  |
    | status                              | ACTIVE                                                          |
    | updated                             | 2021-08-19T14:21:21Z                                            |
    | user_id                             | 29b9101dc52844f5bcd6874fc7a1b4ae                                |
    | volumes_attached                    |                                                                 |
    +-------------------------------------+-----------------------------------------------------------------+

|

 9. horizon 대시보드에 접속하여 커맨드라인 명령어로 생성한 인스턴스를 확인하고, openstack-kr.org 도메인으로 통신 테스트를 합니다.
 통신 테스트 전, 도메인에 대한 아이피 주소 정보를 알아야 하므로 resolv.conf 파일에 1.1.1.1 클라우드 플레어 네임서버를 등록합니다.

 .. image:: images/2week_2-1.png


|


02. ubuntu 이미지를 받고, root password를 설정한 다음 CLI로 이미지를 등록한 후 인스턴스를 생성하고 접속까지 하기
-----------------------------------------------------------------------------------------------------------------------

 1. ubuntu 20.04v 이미지를 다운로드 받을 수 있는 저장소는 URL은 다음과 같습니다. 해당 저장소에서 focal-server-cloudimg-amd64.img 이미지를 서버로 가져옵니다.

 *URL : https://cloud-images.ubuntu.com/focal/current*

 .. code-block:: none

    stack@doa-wallaby:/var/lib/glance/images$ curl -O https://cloud-images.ubuntu.com/focal/current/focal-server-cloudimg-amd64.img
      % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                     Dload  Upload   Total   Spent    Left  Speed
    100  537M  100  537M    0     0  12.2M      0  0:00:43  0:00:43 --:--:-- 12.5M


|

 2. 이미지를 다운로드 받은 후 초기 패스워드를 설정합니다. 내려받은 이미지는 libguestfs-tools 패키지를 설치하여 관리자 계정의 초기 패스워드 설정이 가능합니다.

 .. code-block:: none

    stack@doa-wallaby:/var/lib/glance/images$ sudo virt-customize -a focal-server-cloudimg-amd64.img --root-password password:secret
    [   0.0] Examining the guest ...
    [  46.4] Setting a random seed
    virt-customize: warning: random seed could not be set for this type of
    guest
    [  46.5] Setting the machine ID in /etc/machine-id
    [  46.5] Setting passwords
    [  60.2] Finishing off

|

 3. 이미지 파일을 등록합니다. 이미지의 디스크 타입은 기존에 등록되어 있던 cirrOS 디스크 포맷과 동일하게 qcow2 타입으로 생성하였고, 오픈스택 메뉴얼에서 권장하고 있는 bare 컨테이너 포멧, 모든 프로젝트에서 액세스가 가능하도록 이미지를 등록하였습니다.

 .. code-block:: none

    stack@doa-wallaby:/var/lib/glance/images$ openstack image create "ubuntu_20.04v" --file focal-server-cloudimg-amd64.img --disk-format qcow2 --container-format bare --public                                                                                             amd64.img --disk-format qcow2 --container-format bare --public
    +------------------+---------------------------------------------------------------------------------------------------------------------------------------------------+
    | Field            | Value                                                                                                                                             |
    +------------------+---------------------------------------------------------------------------------------------------------------------------------------------------+
    | container_format | bare                                                                                                                                              |
    | created_at       | 2021-08-21T11:02:53Z                                                                                                                              |
    | disk_format      | qcow2                                                                                                                                             |
    | file             | /v2/images/afd44c7b-1828-476a-9395-b95b4e16f12d/file                                                                                              |
    | id               | afd44c7b-1828-476a-9395-b95b4e16f12d                                                                                                              |
    | min_disk         | 0                                                                                                                                                 |
    | min_ram          | 0                                                                                                                                                 |
    | name             | ubuntu_20.04v                                                                                                                                     |
    | owner            | 4d6190dd59524a5e8247f70566a71c1b                                                                                                                  |
    | properties       | os_hidden='False', owner_specified.openstack.md5='', owner_specified.openstack.object='images/ubuntu_20.04v', owner_specified.openstack.sha256='' |
    | protected        | False                                                                                                                                             |
    | schema           | /v2/schemas/image                                                                                                                                 |
    | status           | queued                                                                                                                                            |
    | tags             |                                                                                                                                                   |
    | updated_at       | 2021-08-21T11:02:53Z                                                                                                                              |
    | visibility       | public                                                                                                                                            |
    +------------------+---------------------------------------------------------------------------------------------------------------------------------------------------+

|

 4. 등록한 이미지 리스트를 조회합니다. ubuntu_20.04v 이미지가 생성이 된 것을 확인해볼 수 있습니다.

 .. code-block:: none

    stack@doa-wallaby:/var/lib/glance/images$ openstack image list
    +--------------------------------------+---------------------------------+--------+
    | ID                                   | Name                            | Status |
    +--------------------------------------+---------------------------------+--------+
    | 64436365-443b-412b-bb20-07492191e3c4 | cirros-0.5.2-x86_64-disk        | active |
    | 0f09bd47-2fad-4da9-aca3-b402c1f21fb1 | private-instance-prac01-snap    | active |
    | afd44c7b-1828-476a-9395-b95b4e16f12d | ubuntu_20.04v                   | active |
    +--------------------------------------+---------------------------------+--------+

|

 5. 등록한 이미지는 Ubuntu 20.04v 이며, 해당 운영체제의 권장 시스템 요구사항은 1GHz CPU, 512MB ~ 1GB 메모리, 5GB 디스크입니다.
 현재 시스템의 적절한 리소스 사용률을 파악하여 인스턴스를 생성하여야 합니다.

 .. code-block:: none

    stack@doa-wallaby:/var/lib/glance/images$ openstack flavor list
    +----+-----------+-------+------+-----------+-------+-----------+
    | ID | Name      |   RAM | Disk | Ephemeral | VCPUs | Is Public |
    +----+-----------+-------+------+-----------+-------+-----------+
    | 1  | m1.tiny   |   512 |    1 |         0 |     1 | True      |
    | 2  | m1.small  |  2048 |   20 |         0 |     1 | True      |
    | 3  | m1.medium |  4096 |   40 |         0 |     2 | True      |
    | 4  | m1.large  |  8192 |   80 |         0 |     4 | True      |
    | 42 | m1.nano   |   128 |    1 |         0 |     1 | True      |
    | 5  | m1.xlarge | 16384 |  160 |         0 |     8 | True      |
    | 84 | m1.micro  |   192 |    1 |         0 |     1 | True      |
    | c1 | cirros256 |   256 |    1 |         0 |     1 | True      |
    | d1 | ds512M    |   512 |    5 |         0 |     1 | True      |
    | d2 | ds1G      |  1024 |   10 |         0 |     1 | True      |
    | d3 | ds2G      |  2048 |   10 |         0 |     2 | True      |
    | d4 | ds4G      |  4096 |   20 |         0 |     4 | True      |
    +----+-----------+-------+------+-----------+-------+-----------+`

|

 6. 등록한 이미지로 내부 네트워크 대역의 호스트를 생성합니다.

 .. code-block:: none

    stack@doa-wallaby:/var/lib/glance/images$ openstack server create --flavor m1.small --image b67c1d90-eacf-4123-8350-1f6ffea37b2f --nic net-id=dacb03f0-4ed9-43cc-82ae-10df1246556f --security-group 6bdec141-e1f3-4e2f-8264-e4783b75ca01 --key-name my-ubuntu-keypair private-instance-ubuntu-20.04
    +-------------------------------------+---------------------------------------------------------+
    | Field                               | Value                                                   |
    +-------------------------------------+---------------------------------------------------------+
    | OS-DCF:diskConfig                   | MANUAL                                                  |
    | OS-EXT-AZ:availability_zone         |                                                         |
    | OS-EXT-SRV-ATTR:host                | None                                                    |
    | OS-EXT-SRV-ATTR:hypervisor_hostname | None                                                    |
    | OS-EXT-SRV-ATTR:instance_name       |                                                         |
    | OS-EXT-STS:power_state              | NOSTATE                                                 |
    | OS-EXT-STS:task_state               | scheduling                                              |
    | OS-EXT-STS:vm_state                 | building                                                |
    | OS-SRV-USG:launched_at              | None                                                    |
    | OS-SRV-USG:terminated_at            | None                                                    |
    | accessIPv4                          |                                                         |
    | accessIPv6                          |                                                         |
    | addresses                           |                                                         |
    | adminPass                           | Mq7TRw9vnJUG                                            |
    | config_drive                        |                                                         |
    | created                             | 2021-08-21T12:16:26Z                                    |
    | flavor                              | m1.small (2)                                            |
    | hostId                              |                                                         |
    | id                                  | c7a395ff-838f-4bf9-a178-7be1827687e7                    |
    | image                               | pw_ubuntu_20.04v (b67c1d90-eacf-4123-8350-1f6ffea37b2f) |
    | key_name                            | my-ubuntu-keypair                                       |
    | name                                | private-instance-ubuntu-20.04                           |
    | progress                            | 0                                                       |
    | project_id                          | 4d6190dd59524a5e8247f70566a71c1b                        |
    | properties                          |                                                         |
    | security_groups                     | name='6bdec141-e1f3-4e2f-8264-e4783b75ca01'             |
    | status                              | BUILD                                                   |
    | updated                             | 2021-08-21T12:16:26Z                                    |
    | user_id                             | 29b9101dc52844f5bcd6874fc7a1b4ae                        |
    | volumes_attached                    |                                                         |
    +-------------------------------------+---------------------------------------------------------+

|

 7. 생성된 인스턴스의 활성 상태와 기타 정보는 다음과 같습니다. 해당 인스턴스가 동적으로 할당된 아이피는 10.0.0.58 입니다.

 .. code-block:: none

    stack@doa-wallaby:/var/lib/glance/images$ openstack server list
    +--------------------------------------+-------------------------------+--------+---------------------------------------------------------+--------------------------+----------+
    | ID                                   | Name                          | Status | Networks                                                | Image                    | Flavor   |
    +--------------------------------------+-------------------------------+--------+---------------------------------------------------------+--------------------------+----------+
    | c7a395ff-838f-4bf9-a178-7be1827687e7 | private-instance-ubuntu-20.04 | ACTIVE | private=10.0.0.58, fd33:b9ad:a660:0:f816:3eff:fe57:6a09 | pw_ubuntu_20.04v         | m1.small |
    | acbf1669-0ed9-46fb-b38b-8769569d27a3 | private-instance-prac01       | ACTIVE | private=10.0.0.14, fd33:b9ad:a660:0:f816:3eff:fe36:8a90 | cirros-0.5.2-x86_64-disk | m1.tiny  |
    +--------------------------------------+-------------------------------+--------+---------------------------------------------------------+--------------------------+----------+

    stack@doa-wallaby:/var/lib/glance/images$ openstack server show c7a395ff-838f-4bf9-a178-7be1827687e7
    +-------------------------------------+----------------------------------------------------------+
    | Field                               | Value                                                    |
    +-------------------------------------+----------------------------------------------------------+
    | OS-DCF:diskConfig                   | MANUAL                                                   |
    | OS-EXT-AZ:availability_zone         | nova                                                     |
    | OS-EXT-SRV-ATTR:host                | doa-wallaby                                              |
    | OS-EXT-SRV-ATTR:hypervisor_hostname | doa-wallaby                                              |
    | OS-EXT-SRV-ATTR:instance_name       | instance-00000004                                        |
    | OS-EXT-STS:power_state              | Running                                                  |
    | OS-EXT-STS:task_state               | None                                                     |
    | OS-EXT-STS:vm_state                 | active                                                   |
    | OS-SRV-USG:launched_at              | 2021-08-21T12:16:34.000000                               |
    | OS-SRV-USG:terminated_at            | None                                                     |
    | accessIPv4                          |                                                          |
    | accessIPv6                          |                                                          |
    | addresses                           | private=10.0.0.58, fd33:b9ad:a660:0:f816:3eff:fe57:6a09  |
    | config_drive                        |                                                          |
    | created                             | 2021-08-21T12:16:26Z                                     |
    | flavor                              | m1.small (2)                                             |
    | hostId                              | a7cfaf95007ab1386481f7ca38819729a0a266a4e72690140c62f66b |
    | id                                  | c7a395ff-838f-4bf9-a178-7be1827687e7                     |
    | image                               | pw_ubuntu_20.04v (b67c1d90-eacf-4123-8350-1f6ffea37b2f)  |
    | key_name                            | my-ubuntu-keypair                                        |
    | name                                | private-instance-ubuntu-20.04                            |
    | progress                            | 0                                                        |
    | project_id                          | 4d6190dd59524a5e8247f70566a71c1b                         |
    | properties                          |                                                          |
    | security_groups                     | name='default'                                           |
    | status                              | ACTIVE                                                   |
    | updated                             | 2021-08-21T12:16:34Z                                     |
    | user_id                             | 29b9101dc52844f5bcd6874fc7a1b4ae                         |
    | volumes_attached                    |                                                          |
    +-------------------------------------+----------------------------------------------------------+

|

 8. 마지막으로 horizon 대시보드에 접근하여 관리자 계정으로 로그인을 하고, 패스워드를 수정합니다.

 .. image:: images/2week_2-2.png

|


03. CLI로 Floating ip 생성 후 인스턴스에 할당하고 해제 해보기
-----------------------------------------------------------------------------

 1. 외부에서 인스턴스로 접근하기 위해서 Floating ip 를 설정해주어야 합니다. 유동 아이피는 기본적으로 퍼블릭 네트워크 대역의 호스트 아이피로 할당이 되며, 현재 생성한 유동 아이피는 하기와 같이 192.168.100.80/24 입니다.

 .. code-block:: none

    stack@doa-wallaby:~/devstack$ openstack floating ip create public
    +---------------------+--------------------------------------+
    | Field               | Value                                |
    +---------------------+--------------------------------------+
    | created_at          | 2021-08-21T13:48:08Z                 |
    | description         |                                      |
    | dns_domain          | None                                 |
    | dns_name            | None                                 |
    | fixed_ip_address    | None                                 |
    | floating_ip_address | 192.168.100.80                       |
    | floating_network_id | 75ef8908-d335-471a-a561-5de378cb0371 |
    | id                  | 85f6609e-cece-47d8-b3b9-9ff21ab711db |
    | name                | 192.168.100.80                       |
    | port_details        | None                                 |
    | port_id             | None                                 |
    | project_id          | 4d6190dd59524a5e8247f70566a71c1b     |
    | qos_policy_id       | None                                 |
    | revision_number     | 0                                    |
    | router_id           | None                                 |
    | status              | DOWN                                 |
    | subnet_id           | None                                 |
    | tags                | []                                   |
    | updated_at          | 2021-08-21T13:48:08Z                 |
    +---------------------+--------------------------------------+

    stack@doa-wallaby:~/devstack$ openstack floating ip list
    +--------------------------------------+---------------------+------------------+------+--------------------------------------+----------------------------------+
    | ID                                   | Floating IP Address | Fixed IP Address | Port | Floating Network                     | Project                          |
    +--------------------------------------+---------------------+------------------+------+--------------------------------------+----------------------------------+
    | 85f6609e-cece-47d8-b3b9-9ff21ab711db | 192.168.100.80      | None             | None | 75ef8908-d335-471a-a561-5de378cb0371 | 4d6190dd59524a5e8247f70566a71c1b |
    +--------------------------------------+---------------------+------------------+------+--------------------------------------+----------------------------------+

|

 3. ubuntu 20.04v 인스턴스에 생성한 유동 아이피를 할당합니다. 유동 아이피를 할당한 후 서버 리스트를 확인해보면, 네트워크 정보에 유동 아이피 정보가 포함이 된 것을 확인해볼 수 있습니다.

 .. code-block:: none

    stack@doa-wallaby:~/devstack$ openstack server add floating ip private-instance-ubuntu-20.04 85f6609e-cece-47d8-b3b9-9ff21ab711db

    stack@doa-wallaby:~/devstack$ openstack floating ip list
    +--------------------------------------+---------------------+------------------+--------------------------------------+--------------------------------------+----------------------------------+
    | ID                                   | Floating IP Address | Fixed IP Address | Port                                 | Floating Network                     | Project                          |
    +--------------------------------------+---------------------+------------------+--------------------------------------+--------------------------------------+----------------------------------+
    | 85f6609e-cece-47d8-b3b9-9ff21ab711db | 192.168.100.80      | 10.0.0.58        | 7b378312-6402-4f70-a06f-cf0a4333dc82 | 75ef8908-d335-471a-a561-5de378cb0371 | 4d6190dd59524a5e8247f70566a71c1b |
    +--------------------------------------+---------------------+------------------+--------------------------------------+--------------------------------------+----------------------------------+

    stack@doa-wallaby:~/devstack$ openstack server list
    +--------------------------------------+-------------------------------+--------+-------------------------------------------------------------------------+--------------------------+----------+
    | ID                                   | Name                          | Status | Networks                                                                | Image                    | Flavor   |
    +--------------------------------------+-------------------------------+--------+-------------------------------------------------------------------------+--------------------------+----------+
    | c7a395ff-838f-4bf9-a178-7be1827687e7 | private-instance-ubuntu-20.04 | ACTIVE | private=10.0.0.58, 192.168.100.80, fd33:b9ad:a660:0:f816:3eff:fe57:6a09 | pw_ubuntu_20.04v         | m1.small |
    | acbf1669-0ed9-46fb-b38b-8769569d27a3 | private-instance-prac01       | ACTIVE | private=10.0.0.14, fd33:b9ad:a660:0:f816:3eff:fe36:8a90                 | cirros-0.5.2-x86_64-disk | m1.tiny  |
    +--------------------------------------+-------------------------------+--------+-------------------------------------------------------------------------+--------------------------+----------+

|

 4. 이후 할당받은 유동 아이피로 시스템에 접근하여 테스트를 해봅니다.

 .. code-block:: none

    stack@doa-wallaby:~/.ssh$ sudo ssh -i my-ubuntu-keypair.pem ubuntu@192.168.100.80
    Welcome to Ubuntu 20.04.3 LTS (GNU/Linux 5.4.0-81-generic x86_64)

     * Documentation:  https://help.ubuntu.com
     * Management:     https://landscape.canonical.com
     * Support:        https://ubuntu.com/advantage

      System information as of Sat Aug 21 14:03:08 UTC 2021

      System load:           0.77
      Usage of /:            6.7% of 19.21GB
      Memory usage:          9%
      Swap usage:            0%
      Processes:             100
      Users logged in:       1
      IPv4 address for ens3: 10.0.0.58
      IPv6 address for ens3: fd33:b9ad:a660:0:f816:3eff:fe57:6a09

    ubuntu@private-instance-ubuntu-20-04:~$ ifconfig
    ens3: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1450
            inet 10.0.0.58  netmask 255.255.255.192  broadcast 10.0.0.63
            inet6 fe80::f816:3eff:fe57:6a09  prefixlen 64  scopeid 0x20<link>
            inet6 fd33:b9ad:a660:0:f816:3eff:fe57:6a09  prefixlen 64  scopeid 0x0<global>
            ether fa:16:3e:57:6a:09  txqueuelen 1000  (Ethernet)
            RX packets 4206  bytes 664489 (664.4 KB)
            RX errors 0  dropped 0  overruns 0  frame 0
            TX packets 4951  bytes 507409 (507.4 KB)
            TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

    lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
            inet 127.0.0.1  netmask 255.0.0.0
            inet6 ::1  prefixlen 128  scopeid 0x10<host>
            loop  txqueuelen 1000  (Local Loopback)
            RX packets 3990  bytes 314495 (314.4 KB)
            RX errors 0  dropped 0  overruns 0  frame 0
            TX packets 3990  bytes 314495 (314.4 KB)
            TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

|

 5. 할당한 유동 아이피를 회수합니다.

 .. code-block:: none

    stack@doa-wallaby:~/devstack$ openstack server remove floating ip private-instance-ubuntu-20.04 85f6609e-cece-47d                                                                                                                        8-b3b9-9ff21ab711db
    stack@doa-wallaby:~/devstack$ openstack floating ip list
    +--------------------------------------+---------------------+------------------+------+--------------------------------------+----------------------------------+
    | ID                                   | Floating IP Address | Fixed IP Address | Port | Floating Network                     | Project                          |
    +--------------------------------------+---------------------+------------------+------+--------------------------------------+----------------------------------+
    | 85f6609e-cece-47d8-b3b9-9ff21ab711db | 192.168.100.80      | None             | None | 75ef8908-d335-471a-a561-5de378cb0371 | 4d6190dd59524a5e8247f70566a71c1b |
    +--------------------------------------+---------------------+------------------+------+--------------------------------------+----------------------------------+

    stack@doa-wallaby:~/devstack$ openstack server list
    +--------------------------------------+-------------------------------+--------+---------------------------------------------------------+--------------------------+----------+
    | ID                                   | Name                          | Status | Networks                                                | Image                    | Flavor   |
    +--------------------------------------+-------------------------------+--------+---------------------------------------------------------+--------------------------+----------+
    | c7a395ff-838f-4bf9-a178-7be1827687e7 | private-instance-ubuntu-20.04 | ACTIVE | private=10.0.0.58, fd33:b9ad:a660:0:f816:3eff:fe57:6a09 | pw_ubuntu_20.04v         | m1.small |
    | acbf1669-0ed9-46fb-b38b-8769569d27a3 | private-instance-prac01       | ACTIVE | private=10.0.0.14, fd33:b9ad:a660:0:f816:3eff:fe36:8a90 | cirros-0.5.2-x86_64-disk | m1.tiny  |
    +--------------------------------------+-------------------------------+--------+---------------------------------------------------------+--------------------------+----------+

|

04. 10.8.0.0/24 네트워크 대역을 생성한 후 public network와 연결하는 과정을 CLI로 해보기 (optional)
----------------------------------------------------------------------------------------------------------------

 1. 네트워크 대역을 생성하기 전, 현재 오픈스택에 할당된 네트워크 대역 정보들은 다음과 같습니다.

 .. code-block:: none

    stack@doa-wallaby:~/devstack$ openstack network list
    +--------------------------------------+---------+----------------------------------------------------------------------------+
    | ID                                   | Name    | Subnets                                                                    |
    +--------------------------------------+---------+----------------------------------------------------------------------------+
    | 75ef8908-d335-471a-a561-5de378cb0371 | public  | 77aa158c-e0fc-4adb-8337-0e7ca30bc275, f8b95048-723b-44f8-9cfb-4ec283669e18 |
    | b1d129cc-ca87-445b-8880-0e2923c47cd6 | shared  | 7ea5ea69-162b-4a1f-8973-708663a7fc16                                       |
    | dacb03f0-4ed9-43cc-82ae-10df1246556f | private | 30754a5c-dd3a-4363-a786-86bfd7e2d7a0, e1d9f47b-c0b0-4f9c-abf3-d6619640622e |
    +--------------------------------------+---------+----------------------------------------------------------------------------+

    stack@doa-wallaby:~/devstack$ openstack subnet list
    +--------------------------------------+---------------------+--------------------------------------+---------------------+
    | ID                                   | Name                | Network                              | Subnet              |
    +--------------------------------------+---------------------+--------------------------------------+---------------------+
    | 30754a5c-dd3a-4363-a786-86bfd7e2d7a0 | ipv6-private-subnet | dacb03f0-4ed9-43cc-82ae-10df1246556f | fd33:b9ad:a660::/64 |
    | 77aa158c-e0fc-4adb-8337-0e7ca30bc275 | ipv6-public-subnet  | 75ef8908-d335-471a-a561-5de378cb0371 | 2001:db8::/64       |
    | 7ea5ea69-162b-4a1f-8973-708663a7fc16 | shared-subnet       | b1d129cc-ca87-445b-8880-0e2923c47cd6 | 192.168.233.0/24    |
    | e1d9f47b-c0b0-4f9c-abf3-d6619640622e | private-subnet      | dacb03f0-4ed9-43cc-82ae-10df1246556f | 10.0.0.0/26         |
    | f8b95048-723b-44f8-9cfb-4ec283669e18 | public-subnet       | 75ef8908-d335-471a-a561-5de378cb0371 | 192.168.100.0/24    |
    +--------------------------------------+---------------------+--------------------------------------+---------------------+

|

 2. 10.8.0.0/24 네트워크 대역을 생성하기 위해서 test-subnet 를 추가합니다.

 .. code-block:: none


    stack@doa-wallaby:~/devstack$ openstack network create test-subnet
    +---------------------------+--------------------------------------+
    | Field                     | Value                                |
    +---------------------------+--------------------------------------+
    | admin_state_up            | UP                                   |
    | availability_zone_hints   |                                      |
    | availability_zones        |                                      |
    | created_at                | 2021-08-21T14:16:01Z                 |
    | description               |                                      |
    | dns_domain                | None                                 |
    | id                        | d49ef591-d14d-4009-be24-ea8a23bb78cf |
    | ipv4_address_scope        | None                                 |
    | ipv6_address_scope        | None                                 |
    | is_default                | False                                |
    | is_vlan_transparent       | None                                 |
    | mtu                       | 1450                                 |
    | name                      | test-subnet                          |
    | port_security_enabled     | True                                 |
    | project_id                | 4d6190dd59524a5e8247f70566a71c1b     |
    | provider:network_type     | vxlan                                |
    | provider:physical_network | None                                 |
    | provider:segmentation_id  | 691                                  |
    | qos_policy_id             | None                                 |
    | revision_number           | 1                                    |
    | router:external           | Internal                             |
    | segments                  | None                                 |
    | shared                    | False                                |
    | status                    | ACTIVE                               |
    | subnets                   |                                      |
    | tags                      |                                      |
    | updated_at                | 2021-08-21T14:16:01Z                 |
    +---------------------------+--------------------------------------+

|

 2. 네트워크의 서브넷을 추가합니다. 네임서버 정보는 클라우드 플레어 네임서버로 설정하였으며, 게이트웨이는 10.8.0.1, 네트워크는 10.8.0.0/24 대역으로 생성하였습니다.

 .. code-block:: none

    stack@doa-wallaby:~/devstack$ openstack subnet create --network test-subnet \
    > --dns-nameserver 1.1.1.1 --gateway 10.8.0.1 \
    > --subnet-range 10.8.0.0/24 test
    +----------------------+--------------------------------------+
    | Field                | Value                                |
    +----------------------+--------------------------------------+
    | allocation_pools     | 10.8.0.2-10.8.0.254                  |
    | cidr                 | 10.8.0.0/24                          |
    | created_at           | 2021-08-21T14:22:55Z                 |
    | description          |                                      |
    | dns_nameservers      | 1.1.1.1                              |
    | dns_publish_fixed_ip | None                                 |
    | enable_dhcp          | True                                 |
    | gateway_ip           | 10.8.0.1                             |
    | host_routes          |                                      |
    | id                   | 8b87a803-88d8-4dec-84a5-dbc5c9d26748 |
    | ip_version           | 4                                    |
    | ipv6_address_mode    | None                                 |
    | ipv6_ra_mode         | None                                 |
    | name                 | test                                 |
    | network_id           | d49ef591-d14d-4009-be24-ea8a23bb78cf |
    | prefix_length        | None                                 |
    | project_id           | 4d6190dd59524a5e8247f70566a71c1b     |
    | revision_number      | 0                                    |
    | segment_id           | None                                 |
    | service_types        |                                      |
    | subnetpool_id        | None                                 |
    | tags                 |                                      |
    | updated_at           | 2021-08-21T14:22:55Z                 |
    +----------------------+--------------------------------------+

|

 3. 네트워크를 추가하였으므로, 각 구간끼리 통신을 하기 위한 라우터의 인터페이스를 설정해주어야 합니다. 라우터는 기존에 생성된 router1 라우터에 설정하였습니다.

 .. code-block:: none

    stack@doa-wallaby:~/devstack$ openstack router list
    +--------------------------------------+---------+--------+-------+----------------------------------+-------------+-------+
    | ID                                   | Name    | Status | State | Project                          | Distributed | HA    |
    +--------------------------------------+---------+--------+-------+----------------------------------+-------------+-------+
    | 92e2a949-8a26-4604-817b-16641e16b27e | router1 | ACTIVE | UP    | d8c257ebe8b04e869a00434ae6665f3c | False       | False |
    +--------------------------------------+---------+--------+-------+----------------------------------+-------------+-------+

    stack@doa-wallaby:~/devstack$ openstack router add subnet 92e2a949-8a26-4604-817b-16641e16b27e 8b87a803-88d8-4dec-84a5-dbc5c9d26748

|

 4. 라우팅 테스트를 위해 테스트 서버를 하나 생성하였습니다. 생성한 테스트 서버의 이름은 new-network-test 입니다.

 .. code-block:: none

    stack@doa-wallaby:~/devstack$ openstack server list
    +--------------------------------------+-------------------------------+--------+---------------------------------------------------------+--------------------------+----------+
    | ID                                   | Name                          | Status | Networks                                                | Image                    | Flavor   |
    +--------------------------------------+-------------------------------+--------+---------------------------------------------------------+--------------------------+----------+
    | 3a03e95d-2956-49ac-acbf-c08084a7166d | new-network-test              | ACTIVE | test-subnet=10.8.0.211                                  | cirros-0.5.2-x86_64-disk | m1.tiny  |
    | c7a395ff-838f-4bf9-a178-7be1827687e7 | private-instance-ubuntu-20.04 | ACTIVE | private=10.0.0.58, fd33:b9ad:a660:0:f816:3eff:fe57:6a09 | pw_ubuntu_20.04v         | m1.small |
    | acbf1669-0ed9-46fb-b38b-8769569d27a3 | private-instance-prac01       | ACTIVE | private=10.0.0.14, fd33:b9ad:a660:0:f816:3eff:fe36:8a90 | cirros-0.5.2-x86_64-disk | m1.tiny  |
    +--------------------------------------+-------------------------------+--------+---------------------------------------------------------+--------------------------+----------+

|

 5. 신규 서버에서 두번째 항목에서 설치한 ubuntu 20.04v 서버로 ICMP 통신 테스트를 진행한 결과, 정상적으로 통신이 되는 것을 확인하였습니다.

 .. image:: images/2week_2-3.png

 .. image:: images/2week_2-4.png