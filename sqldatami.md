
# 📦 Migration MySQL Data in Docker

Migrating MySQL Data to Another Container in a Docker Environment
In a Docker environment, migrating MySQL data from one container to another involves creating a backup of the database, transferring that backup to the new container, and restoring the data. The process typically includes three major steps: backup, container creation, and restoration.

## ⚙️ 1. Backup of MySQL Data
In the first step, the MySQL data from the existing container is backed up using a tool like `mysqldump`. This generates a SQL file that contains the schema and data of the databases stored within the MySQL server. The purpose of this step is to preserve the data in a transferable format so that it can be moved to a new container.

## 🛠️ 2. Creation of a New MySQL Container
Once the backup is made, a new MySQL container is created using the Docker `run` command. This container serves as the new MySQL instance where the backed-up data will be restored. During this process, the new container is configured with necessary parameters such as the MySQL root password and version.

## 🔄 3. Restoration of Data in the New Container
After creating the new MySQL container, the backup file is copied into the new container using the `docker cp` command. Then, the data is restored using the MySQL `mysql` command, which reads the backup file and reconstructs the databases inside the new container. This effectively transfers the data from the old container to the new one.

---

## 📋 Reasons for Migrating MySQL Data to Another Container

### 🆙 Version Upgrades:
When upgrading the MySQL version in a container, it's often necessary to create a new container with the upgraded version. Migrating data from the old container to the new one ensures that all the data is preserved while taking advantage of the new version's features or security patches.

### 🚨 Disaster Recovery:
In the event of container failure or corruption, the ability to quickly migrate data to a new container is essential for minimizing downtime. By keeping regular backups, data can be restored to a new MySQL container without significant disruption.

### 📈 Scaling and Load Balancing:
When scaling an application, you may need to create additional containers or move the database to a more powerful instance. Migrating MySQL data allows you to distribute the database load or move it to a better-suited environment.

### 🛡️ Isolating Environment Changes:
In development and testing environments, migrating MySQL data from one container to another can be useful when isolating changes or testing new features without affecting the production database.

### 🌐 Platform Migration:
If the Docker host or environment is changed, for example, moving from an on-premise system to a cloud-based infrastructure, the MySQL data needs to be migrated to a new environment, which can be achieved by moving the data between containers.

### 🔐 Security Enhancements:
In some cases, the original container might have been created with less secure configurations. Migrating the data to a new container with updated security policies allows you to improve the security without losing the data.

---

## 🚀 Practical Steps for Migration

### 1. Create Container
![image](https://github.com/user-attachments/assets/4d0587ac-7db7-45a6-b9cc-82b430f100c5)
![image](https://github.com/user-attachments/assets/57d4b532-59ce-4e54-9b65-0ce535123e5f)
```bash
docker run --name mysqldb -e MYSQL_ROOT_PASSWORD=[password] -d -p 3306:3306 mysql:latest
```

### 2. Insert Data into MySQL
![image](https://github.com/user-attachments/assets/0a2f2900-4ba7-4201-9bcc-12b661ab623d)
```sql
CREATE DATABASE fisa;
USE fisa;

GRANT ALL PRIVILEGES ON fisa.* TO 'root'@'localhost';
FLUSH PRIVILEGES;

CREATE TABLE dept (
    deptno               int  NOT NULL ,
    dname                varchar(20),
    loc                  varchar(20),
    CONSTRAINT pk_dept PRIMARY KEY ( deptno )
);

INSERT INTO dept VALUES(10, 'ACCOUNTING', 'NEW YORK');
INSERT INTO dept VALUES(20, 'RESEARCH', 'DALLAS');
INSERT INTO dept VALUES(30, 'SALES', 'CHICAGO');
INSERT INTO dept VALUES(40, 'OPERATIONS', 'BOSTON');
COMMIT;

SELECT * FROM dept;
```

### 3. Create Backup Data
![image](https://github.com/user-attachments/assets/672b7579-69b0-4208-a7b8-26db39424fb6)
![image](https://github.com/user-attachments/assets/f522e07b-73ac-4b45-81bb-6f826d120cbd)
```bash
docker exec -it mysqldb mysqldump -u [username] -p[password] --databases fisa > backup.sql
```

### 4. Create New Container for Migration
![image](https://github.com/user-attachments/assets/2bb5e987-e450-4549-8c6d-bbaffcc7e6b5)
```bash
docker run --name newmysqldb -e MYSQL_ROOT_PASSWORD=[password] -d mysql:latest
```

### 5. Import Backup Data into New Container
![image](https://github.com/user-attachments/assets/a4762645-c6b9-4583-b647-4c8e3730cc44)
```bash
docker cp backup.sql newmysqldb:/backup.sql
```

### 6. Restore Data in New Container
![image](https://github.com/user-attachments/assets/f110706d-ea6b-4c2e-856a-0807162b33f8)
```bash
docker exec -it newmysqldb bash
mysql -u root -p[newpassword] < /backup.sql
```
### Troubleshooting
![image](https://github.com/user-attachments/assets/e149d598-a39f-4bc5-ba02-fee5327b1315)
Because of command line 1, It cannot operate. so, I just delete that line.

### 7. Result
![image](https://github.com/user-attachments/assets/86ac8777-9315-4528-b694-06a277bd2b64)
I succesfully import sql data from mysqldb container

---

## 🕒 Using Crontab for Backup and Restore

### 1. Writing Backup and Restore Script
```bash
#!/bin/bash

# Backup MySQL existing Data
docker exec -it mysqldb mysqldump -u root -proot --databases fisa > /backup.sql

# Create new container
docker run --name newmysqldb2 -e MYSQL_ROOT_PASSWORD=root -d mysql:latest

# Copy backup file to new container
docker cp /backup.sql newmysqldb2:/backup.sql

# Restore backup data in new container
docker exec -it newmysqldb2 mysql -u root -proot < /backup.sql

# Check if restoration of data is completed
docker exec -it newmysqldb2 mysql -u root -proot -e "SHOW DATABASES;"
```

### 2. Change Mode for Script Execution
![image](https://github.com/user-attachments/assets/e12a7451-a7a3-4b70-ab60-ad2beba1318f)
```bash
chmod +x /path/to/backup_and_restore.sh
```

### 3. Set Crontab to Run Every 3 Minutes
![image](https://github.com/user-attachments/assets/a0217890-315f-423e-b34f-19f078974964)
```bash
*/3 * * * * /path/to/backup_and_restore.sh > /path/to/log_file.log 2>&1
```

--- 

## 🎉 Conclusion
Docker containerized environments offer flexibility and scalability, but managing persistent data such as MySQL databases requires careful handling. Migrating MySQL data from one container to another is a practical solution for upgrading systems, recovering from failures, and maintaining seamless operations while scaling or updating the infrastructure.
