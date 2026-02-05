---
title: "MariaDB 쿼리 로그 관리"
date: 2026-02-05 14:07:00 +0900
categories: [dev, mariadb]
tags: [mariadb]
---

# mariadb-query-log-mgmt
MariaDB Query 작업 이력 정보를 관리

## Root(관리자) 계정으로 수행해야 하는 작업

> 아래 항목은 **DB root(또는 root급 권한)** / **OS root(sudo)** 가 필요합니다.  
> 권한을 분리(A 방식)하는 경우, `audit_admin`은 적재/가공만 수행하고 “글로벌 설정 변경”은 root가 수행합니다.

---

### 1) DB root(또는 root급 권한)으로 해야 하는 작업

#### 1.1 server_audit 플러그인 설치
```sql
INSTALL SONAME 'server_audit';

-- 설치 확인
SELECT PLUGIN_NAME, PLUGIN_STATUS
FROM information_schema.PLUGINS
WHERE PLUGIN_NAME='server_audit';
```

#### 1.2 server_audit 영구 설정 적용 (my.cnf 반영)

Ubuntu 기준 예: /etc/mysql/mariadb.conf.d/50-server.cnf 에 [mysqld]로 추가/수정

```sql
[mysqld]
plugin_load_add = server_audit

server_audit_logging = ON
server_audit_events = CONNECT,QUERY,TABLE

server_audit_output_type = FILE
server_audit_file_path = /var/log/mysql/server_audit.log

server_audit_file_rotate_size = 100000000
server_audit_file_rotations   = 10
server_audit_query_log_limit  = 8192

server_audit_excl_users = root,audit_admin
server_audit_incl_users =
```

```sql
-- 적용(재시작 필요):
sudo systemctl restart mariadb
```

```sql
-- 정상 동작 확인:
SHOW GLOBAL STATUS LIKE 'Server_audit_active';
SHOW GLOBAL VARIABLES LIKE 'server_audit_events';
SHOW GLOBAL VARIABLES LIKE 'server_audit_file_path';
```

#### 1.3 배치에서 사용하는 GLOBAL 변수 변경 (권한 필요)
아래 구문은 GLOBAL 시스템 변수 변경이므로 root(또는 SUPER / SYSTEM_VARIABLES_ADMIN급) 권한이 필요합니다.

```sql
SET GLOBAL server_audit_file_rotate_now = ON;
SET GLOBAL server_audit_logging = OFF;
SET GLOBAL server_audit_logging = ON;
```

A 방식에서는 위 구문을 audit-root.cnf(root 계정)로 실행합니다.

#### 1.4 초기 SQL 초기 정보를 구축한다.

```sql
mysql -u root -p < auditdb-init.sql
```

#### 1.5 계정 생성 및 권한 부여(관리자 작업)
audit_admin 계정 생성 및 권한 부여는 관리자 권한으로 수행합니다.

```sql
CREATE USER 'audit_admin'@'localhost' IDENTIFIED BY 'CHANGE_ME_AUDIT_ADMIN_PASSWORD';

GRANT SELECT, INSERT, DELETE ON auditdb.* TO 'audit_admin'@'localhost';

-- audit_first_kw() 함수 사용 시 필요
GRANT EXECUTE ON FUNCTION auditdb.audit_first_kw TO 'audit_admin'@'localhost';

FLUSH PRIVILEGES;
```

### 2) OS root(sudo)로 해야 하는 작업
#### 2.1 audit 로그 디렉토리 권한 세팅

mysqld(mysql)이 로그 파일을 생성/쓰기 할 수 있어야 합니다.

```sql
sudo mkdir -p /var/log/mysql
sudo chown -R mysql:mysql /var/log/mysql
sudo chmod 750 /var/log/mysql
```

#### 2.2 rotate 파일 권한 조정/삭제(배치 스크립트 동작 조건)

배치 스크립트가 .01 파일을 읽고(LOAD) 삭제하는 흐름이라면, 해당 파일 권한이 필요합니다.

```sql
sudo chmod 644 /var/log/mysql/server_audit.log.01
sudo chown mysql:mysql /var/log/mysql/server_audit.log.01
sudo rm -f /var/log/mysql/server_audit.log.01
```
#### 2.3 cnf 파일 이동

audit-admin.cnf, audit-root.cnf 를 /etc/mysql/mariadb.conf.d 로 이동시킨다.
(디렉토리 경로는 Linux 버전에 따라 경로는 달라질 수 있음)

#### 2.4 cnf 파일 권한 잠금(보안 필수)

cnf에는 평문 비밀번호가 들어가므로 반드시 권한을 제한합니다.

```sql
sudo chown root:root audit-root.cnf audit-admin.cnf
sudo chmod 600 audit-root.cnf audit-admin.cnf
```


