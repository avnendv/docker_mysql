- Tìm kiếm mysql trên docker
  ```
    docker search mysql
  ```
- Tạo mysql

  ```
    docker run --name testmysql -p 3307:3306 -e MYSQL_ROOT_PASSWORD=passroot -d mysql:latest --max-connections=100
  ```

- Kiểm tra mysql đã được tạo chưa
  ```
    docker ps
  ```
- Kết nối bash mysql trong docker
  ```
    docker exec -it testmysql bash
  ```
- Login mysql
  ```
    mysql -uroot -ppassroot
  ```
- Kiểm tra danh sách database
  ```
    show databases;
  ```
- Tạo user và cấp quyền cho user quản lý database

  ```
    CREATE USER 'testuser'@'%' IDENTIFIED BY 'testpass';
    create database avnendv;
    GRANT ALL PRIVILEGES ON avnendv.* TO 'testuser'@'%';
    FLUSH PRIVILEGES;
  ```

---

# MySQL Setup master slave

- Tao network
  ```
    docker network create my_master_slave_mysql
  ```
- Tao mysql master and slave

  ```
    docker run -d --name mysql8-master --network my_master_slave_mysql -p 8811:3306 -e MYSQL_ROOT_PASSWORD=avnendv mysql:8.0

    docker run -d --name mysql8-slave --network my_master_slave_mysql -p 8822:3306 -e MYSQL_ROOT_PASSWORD=avnendv mysql:8.0
  ```

- Kiem tra xem da tao duoc chua

  ```
    docker ps -a
  ```

- Lay file config cua mysql8-master

  ```
    docker exec -it mysql8-master bash
    cat /etc/my.cnf
    exit
    docker cp bf562f16049b:/etc/my.cnf ./mysql/master/
  ```

- Lay file config cua mysql8-slave
  ```
    docker cp 6b8af8dbb09a:/etc/my.cnf ./mysql/slave/
  ```
- Cap nhat thong tin file config
  ```
    login_bin=mysql-bin
    server-id=1 // 2 for slave
  ```
- Cap nhap file config tren docker
  ```
    docker cp ./mysql/master/my.cnf bf562f16049b:/etc
    docker cp ./mysql/slave/my.cnf 6b8af8dbb09a:/etc
  ```
- Restart
  ```
    docker restart mysql8-master
    docker restart mysql8-slave
  ```
- Ket noi vao slave

  ```
    // lay file log cua master
    show master status; // sql query
    // lay ip cua master
    docker inspect bf562f16049b
    // kiem tra server-id
    show variables like 'server_id';
  ```

  ```
    docker exec -it mysql8-slave bash
    mysql -uroot -pavnendv
    //sql
    CHANGE MASTER TO
    MASTER_HOST='master_host', //172.19.0.2
    MASTER_PORT=3306,
    MASTER_USER='root',
    MASTER_PASSWORD='avnendv',
    master_log_file='binlogfile', // mysql-bin.000001
    master_log_pos=157,
    master_connect_retry=60,
    GET_MASTER_PUBLIC_KEY=1;

    START SLAVE;
    SHOW SLAVE STATUS\G;

    START REPLICA;
    SHOW REPLICA STATUS\G;

    //exp
      CHANGE MASTER TO
      MASTER_HOST='172.19.0.2',
      MASTER_PORT=3306,
      MASTER_USER='root',
      MASTER_PASSWORD='avnendv',
      master_log_file='mysql-bin.000001',
      master_log_pos=157,
      master_connect_retry=60,
      GET_MASTER_PUBLIC_KEY=1;
  ```
