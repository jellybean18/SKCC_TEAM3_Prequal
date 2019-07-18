# Prequal 실습 (첨부파일 참고 부탁드립니다.)
## 1. System Configuration Check
```
1. Check vm.swappiness on all your nodes
 1) swap 확인 
  -> sysctl vm.swappiness
 2) 영구 적용 위해 /etc/sysctl.conf 추가 필요 
  -> vi /etc/sysctl.conf --> vm.swappiness = 1 추가
```
![photo.PNG](https://github.com/jellybean18/SKCC_TEAM3_Prequal/blob/master/image/1.png?raw=true)
![photo.PNG](https://github.com/jellybean18/SKCC_TEAM3_Prequal/blob/master/image/2.png?raw=true)
  
```
2. Show the mount attributes of your volume(s)
 1) 명령어로 확인 -> df -h
```
![photo.PNG](https://github.com/jellybean18/SKCC_TEAM3_Prequal/blob/master/image/3.png?raw=true)
```
3. If you have ext-based volumes, list the reserve space setting
 1) 명령어로 확인 -> fsck -a /dev/nvme0n1p1
```
![photo.PNG](https://github.com/jellybean18/SKCC_TEAM3_Prequal/blob/master/image/4.png?raw=true)
```
 
4. Disable transparent hugepage support
 1) cat /sys/kernel/mm/transparent_hugepage/enabled 상태확인
    [always] madvise never --> 이렇게 나오면 enabled 상태
    always madvise [never] --> 이렇게 나와야 disabled 상태
 2) sudo vi /etc/rc.d/rc.local 편집하여 아래내용 추가
    echo never > /sys/kernel/mm/transparent_hugepage/enabled
    echo never > /sys/kernel/mm/transparent_hugepage/defrag
 3) 권한 변경 -> chmod +x /etc/rc.d/rc.local 
 4) sudo vi /etc/default/grub 편집하여 아래내용 추가
    transparent_hugepage=never
 5) 명령어 실행 -> grub2-mkconfig -o /boot/grub2/grub.cfg

```
![photo.PNG](https://github.com/jellybean18/SKCC_TEAM3_Prequal/blob/master/image/5.png?raw=true)
![photo.PNG](https://github.com/jellybean18/SKCC_TEAM3_Prequal/blob/master/image/6.png?raw=true)
![photo.PNG](https://github.com/jellybean18/SKCC_TEAM3_Prequal/blob/master/image/7.png?raw=true)
```
5. List your network interface configuration
 1) ip addr 실행하여 상태확인
```
![photo.PNG](https://github.com/jellybean18/SKCC_TEAM3_Prequal/blob/master/image/8.png?raw=true)
```

6. Show that forward and reverse host lookups are correctly resolved
 1) centos 계정 비밀번호 설정
   sudo passwd centos --> admin으로 통일
 2) ssh 설정 변경
   sudo vi /etc/ssh/sshd_config 편집하여 아래내용 변경
   PasswordAuthentication yes 주석 풀기
   PasswordAuthentication no 주석 처리
 3) hostname 설정
   sudo hostnamectl set-hostname 노드번호.team3.com (ex) nd1.team3.com)
```
![photo.PNG](https://github.com/jellybean18/SKCC_TEAM3_Prequal/blob/master/image/9.png?raw=true)
![photo.PNG](https://github.com/jellybean18/SKCC_TEAM3_Prequal/blob/master/image/10.png?raw=true)
```
   
 4) /etc/hosts 설정
   172.31.0.147 nd1.team3.com nd1
   172.31.2.54 nd2.team3.com nd2
   172.31.7.7 nd3.team3.com nd3
   172.31.2.235 nd4.team4.com nd4
   172.31.4.97 nd5.team5.com nd5
 5) sudo reboot 로 재시작 후 ssh로 각 노드 접속 테스트
 
7. Show the nscd service is running
 sudo yum install nscd -y
 sudo systemctl start nscd --> 서비스 시작
 systemctl status nscd -> 서비스 상태 확인
 
8. Show the ntpd service is running
 sudo yum install ntp -y
 sudo systemctl start ntpd --> 서비스 시작
 systemctl status ntpd --> 서비스 상태 확인

```
![photo.PNG](https://github.com/jellybean18/SKCC_TEAM3_Prequal/blob/master/image/11.png?raw=true)
```
```

## 2. Cloudera Manger 구성

```
1. CM 5.15.2 Repo 설정 (모든 host)
 sudo yum install -y wget
 sudo wget https://archive.cloudera.com/cm5/redhat/7/x86_64/cm/cloudera-manager.repo -P /etc/yum.repos.d/
 sudo vi /etc/yum.repos.d/cloudera-manager.repo 편집하여 아래내용 변경
 baseurl=https://archive.cloudera.com/cm5/redhat/7/x86_64/cm/5.15.2/

2. JDK, JDBC 설치(모든 host)
 1) jdk -> sudo yum install java-1.8.0-openjdk-devel.x86_64
 2) jdbc -> 
    wget https://dev.mysql.com/get/Downloads/Connector-J/mysql-connector-java-5.1.47.tar.gz
    tar zxvf mysql-connector-java-5.1.47.tar.gz
    sudo mkdir -p /usr/share/java/
    cd mysql-connector-java-5.1.47
    sudo cp mysql-connector-java-5.1.47-bin.jar /usr/share/java/mysql-connector-java.jar
    
 3. yum Install CM (CM host)
   sudo yum install -y cloudera-manager-daemons cloudera-manager-server
  
 4. Install MySQL server (CM host)
   sudo yum install -y mariadb-server
   sudo systemctl enable mariadb (안되면 재설치)
   sudo systemctl start mariadb
   sudo /usr/bin/mysql_secure_installation
   
   [...]
Enter current password for root (enter for none):
OK, successfully used password, moving on...
[...]
Set root password? [Y/n] Y
New password:
Re-enter new password:
[...]
Remove anonymous users? [Y/n] Y
[...]
Disallow root login remotely? [Y/n] N
[...]
Remove test database and access to it [Y/n] Y
[...]
Reload privilege tables now? [Y/n] Y
[...]
All done!  If you've completed all of the above steps, your MariaDB
installation should now be secure.

Thanks for using MariaDB!

   
 5. MySQL DB생성 및 권한부여 (CM host)
 mysql -u root -p --> db 접속
 
  CREATE DATABASE scm DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
  CREATE DATABASE amon DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
  CREATE DATABASE rman DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
  CREATE DATABASE hue DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
  CREATE DATABASE metastore DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
  CREATE DATABASE sentry DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
  CREATE DATABASE nav DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
  CREATE DATABASE navms DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
  CREATE DATABASE oozie DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;

  GRANT ALL ON scm.* TO 'scm'@'%' IDENTIFIED BY 'password';
  GRANT ALL ON amon.* TO 'amon'@'%' IDENTIFIED BY 'password';
  GRANT ALL ON rman.* TO 'rman'@'%' IDENTIFIED BY 'password';
  GRANT ALL ON hue.* TO 'hue'@'%' IDENTIFIED BY 'password';
  GRANT ALL ON metastore.* TO 'hive'@'%' IDENTIFIED BY 'password';
  GRANT ALL ON sentry.* TO 'sentry'@'%' IDENTIFIED BY 'password';
  GRANT ALL ON nav.* TO 'nav'@'%' IDENTIFIED BY 'password';
  GRANT ALL ON navms.* TO 'navms'@'%' IDENTIFIED BY 'password';
  GRANT ALL ON oozie.* TO 'oozie'@'%' IDENTIFIED BY 'password';

  SHOW DATABASES;
 
  EXIT;
 ``` 
  ![photo.PNG](https://github.com/jellybean18/SKCC_TEAM3_Prequal/blob/master/image/databases.PNG?raw=true)
 ``` 
6. cloudera manager 설치 (CM Host)
sudo yum install cloudera-manager-daemons cloudera-manager-server
7. cloudera manager db 설정 (CM Host)
sudo /usr/share/cmf/schema/scm_prepare_database.sh mysql scm scm password
db.mgmt.properries 존재한다면
sudo rm /etc/cloudera-scm-server/db.mgmt.properties
8. cloudera manager 실행 (CM Host)
sudo systemctl start cloudera-scm-server
sudo tail -f /var/log/cloudera-scm-server/cloudera-scm-server.log --> 로그 확인
```
##3. Install a cluster and deploy CDH
```
1. host에 node 명 입력 하여 connection test
```
![photo.PNG](https://github.com/jellybean18/SKCC_TEAM3_Prequal/blob/master/image/host.PNG?raw=true)
```
2. parcel에서 5.15.2 확인 후 선택하여 설치 진행
```
![photo.PNG](https://github.com/jellybean18/SKCC_TEAM3_Prequal/blob/master/image/parcel.PNG?raw=true)
```
3. install 완료 후 설치 서비스 선택하여 DB 정보 입력
```
![photo.PNG](https://github.com/jellybean18/SKCC_TEAM3_Prequal/blob/master/image/db_test.PNG?raw=true)

![photo.PNG](https://github.com/jellybean18/SKCC_TEAM3_Prequal/blob/master/image/0718/ClusterCfg-1.png?raw=true)
![photo.PNG](https://github.com/jellybean18/SKCC_TEAM3_Prequal/blob/master/image/0718/ClusterCfg-2.png?raw=true)
![photo.PNG](https://github.com/jellybean18/SKCC_TEAM3_Prequal/blob/master/image/0718/ClusterCfg-3.png?raw=true)
![photo.PNG](https://github.com/jellybean18/SKCC_TEAM3_Prequal/blob/master/image/0718/ClusterCfg-4.png?raw=true)
![photo.PNG](https://github.com/jellybean18/SKCC_TEAM3_Prequal/blob/master/image/0718/ClusterCfg-5-1.png?raw=true)
![photo.PNG](https://github.com/jellybean18/SKCC_TEAM3_Prequal/blob/master/image/0718/ClusterCfg-5-2.png?raw=true)
![photo.PNG](https://github.com/jellybean18/SKCC_TEAM3_Prequal/blob/master/image/0718/ClusterCfg-6.png?raw=true)
```
4. 모든 설치 완료 된 모습
```
![photo.PNG](https://github.com/jellybean18/SKCC_TEAM3_Prequal/blob/master/image/finish.PNG?raw=true)

```
----------------------------------------------------------------------------------------------------------------------------------
```

```
카프카 설치
- parcel(package) 에서 Kafka 패키지 "다운로드" -> "배포" -> "활성" 수행  
  cluster 메뉴에서 Kafka 서비스 추가
  -> 브로커는 데이터노드에, 게이트웨이는 전체 노드에

![photo.PNG](https://github.com/jellybean18/SKCC_TEAM3_Prequal/blob/master/image/0718/Kafka1.png?raw=true)
![photo.PNG](https://github.com/jellybean18/SKCC_TEAM3_Prequal/blob/master/image/0718/Kafka2.PNG?raw=true)
![photo.PNG](https://github.com/jellybean18/SKCC_TEAM3_Prequal/blob/master/image/0718/Kafka3.PNG?raw=true)
![photo.PNG](https://github.com/jellybean18/SKCC_TEAM3_Prequal/blob/master/image/0718/Kafka4.PNG?raw=true)




  
  
  
  
 
