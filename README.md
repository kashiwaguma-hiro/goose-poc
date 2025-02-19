
# goose poc

[goose](https://github.com/pressly/goose/tree/main) を利用したDBマイグレーションのPoC

## 初期状態
```
export GOOSE_DRIVER=mysql GOOSE_DBSTRING="root:rootroot@tcp(localhost:23306)/test" 
export GOOSE_MIGRATION_DIR=./migrations
% goose version
2025/02/19 23:36:17 goose: version 0

% goose status 
2025/02/19 23:36:22 goose run: failed to collect migrations: no migration files found
```

## ファイル作成
```
% goose create hoge sql
2025/02/17 23:21:09 Created new file: 20250217142109_hoge.sql

 % goose status
2025/02/19 23:23:22     Applied At                  Migration
2025/02/19 23:23:22     =======================================
2025/02/19 23:23:22     Pending                  -- 20250219141258_test.sql
```

## 状況確認
```
% docker exec -i goose-poc-db-1 mysql -uroot -prootroot -e "show databases"
Database
information_schema
mysql
performance_schema
sys
test

% docker exec -i goose-poc-db-1 mysql -uroot -prootroot test -e "show tables"
Tables_in_test
goose_db_version

% docker exec -i goose-poc-db-1 mysql -uroot -prootroot test -e "select * from goose_db_version;"
id      version_id      is_applied      tstamp
1       0       1       2025-02-17 23:13:25
```

## 反映
```
% goose up
2025/02/19 23:24:35 OK   20250219141258_test.sql (26.01ms)
2025/02/19 23:24:35 goose: successfully migrated database to version: 20250219141258

% goose status       
2025/02/19 23:24:52     Applied At                  Migration
2025/02/19 23:24:52     =======================================
2025/02/19 23:24:52     Wed Feb 19 23:24:35 2025 -- 20250219141258_test.sql

% docker exec -i goose-poc-db-1 mysql -uroot -prootroot test -e "select * from goose_db_version;"
id      version_id      is_applied      tstamp
1       0       1       2025-02-17 23:13:25
4       20250219141258  1       2025-02-19 23:24:35
```

## ロールバック
```
% goose down
2025/02/19 23:25:20 OK   20250219141258_test.sql (12.04ms)

% goose status
2025/02/19 23:25:30     Applied At                  Migration
2025/02/19 23:25:30     =======================================
2025/02/19 23:25:30     Pending                  -- 20250219141258_test.sql

% docker exec -i goose-poc-db-1 mysql -uroot -prootroot test -e "select * from goose_db_version;"
id      version_id      is_applied      tstamp
1       0       1       2025-02-17 23:13:25
```

## さらに新しいバージョン
```
% goose create test2 sql
2025/02/19 23:42:55 Created new file: migrations/20250219144255_test2.sql

% goose status
2025/02/19 23:43:00     Applied At                  Migration
2025/02/19 23:43:00     =======================================
2025/02/19 23:43:00     Pending                  -- 20250219141258_test.sql
2025/02/19 23:43:00     Pending                  -- 20250219144255_test2.sql
```

## 未反映をすべて反映
```
% goose up
2025/02/19 23:43:06 OK   20250219141258_test.sql (8.68ms)
2025/02/19 23:43:06 OK   20250219144255_test2.sql (4.67ms)
2025/02/19 23:43:06 goose: successfully migrated database to version: 20250219144255
```

## 1つ戻す
```
% goose status
2025/02/19 23:43:19     Applied At                  Migration
2025/02/19 23:43:19     =======================================
2025/02/19 23:43:19     Wed Feb 19 23:43:06 2025 -- 20250219141258_test.sql
2025/02/19 23:43:19     Wed Feb 19 23:43:06 2025 -- 20250219144255_test2.sql
% goose down
2025/02/19 23:43:33 OK   20250219144255_test2.sql (11.31ms)
% goose status
2025/02/19 23:43:37     Applied At                  Migration
2025/02/19 23:43:37     =======================================
2025/02/19 23:43:37     Wed Feb 19 23:43:06 2025 -- 20250219141258_test.sql
2025/02/19 23:43:37     Pending                  -- 20250219144255_test2.sql
```

## さらに新しいバージョンを追加
```
% goose create test3 sql 
2025/02/19 23:49:57 Created new file: migrations/20250219144957_test3.sql

% goose status
2025/02/19 23:50:00     Applied At                  Migration
2025/02/19 23:50:00     =======================================
2025/02/19 23:50:00     Wed Feb 19 23:43:06 2025 -- 20250219141258_test.sql
2025/02/19 23:50:00     Pending                  -- 20250219144255_test2.sql
2025/02/19 23:50:00     Pending                  -- 20250219144957_test3.sql
```

## 1つだけ反映してみる
```
% goose up-by-one
2025/02/19 23:50:22 OK   20250219144255_test2.sql (6.42ms)

% goose status   
2025/02/19 23:50:28     Applied At                  Migration
2025/02/19 23:50:28     =======================================
2025/02/19 23:50:28     Wed Feb 19 23:43:06 2025 -- 20250219141258_test.sql
2025/02/19 23:50:28     Wed Feb 19 23:50:22 2025 -- 20250219144255_test2.sql
2025/02/19 23:50:28     Pending                  -- 20250219144957_test3.sql
```