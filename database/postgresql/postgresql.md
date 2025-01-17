## 数据类型

**Table 8.2. Numeric Types**

| Name          | Storage Size | Description                    | Range                                        |
| ------------- | ------------ | ------------------------------ | -------------------------------------------- |
| `bigint`      | 8 bytes      | large-range integer            | -9223372036854775808 to +9223372036854775807 |
| `smallserial` | 2 bytes      | small autoincrementing integer | 1 to 32767                                   |
| `serial`      | 4 bytes      | autoincrementing integer       | 1 to 2147483647                              |
| `bigserial`   | 8 bytes      | large autoincrementing integer | 1 to 9223372036854775807                     |



```sql
CREATE TABLE test_bigserial(
   id BIGSERIAL PRIMARY KEY,
   name VARCHAR NOT NULL
);

-- Sequence and defined type
CREATE SEQUENCE IF NOT EXISTS test_bigserial_id_seq;

-- Table Definition
CREATE TABLE "public"."test_bigserial" (
    "id" int8 NOT NULL DEFAULT nextval('test_bigserial_id_seq'::regclass),
    "name" varchar NOT NULL,
    PRIMARY KEY ("id")
);
```

By assigning the `SERIAL` pseudo-type to the `id` column, PostgreSQL performs the following:

- First, create a sequence object and set the next value generated by the sequence as the default value for the column.
- Second, add a `NOT NULL` constraint to the `id` column because a sequence always generates an integer, which is a non-null value.
- Third, assign the owner of the sequence to the `id` column; as a result, the sequence object is deleted when the `id` column or table is dropped

## 权限

### 角色

本地数据库 postgresql 为超级管理员

postgresql 中的 user 和 role 是同一个东西。user 可以登录，都可以使用 `CREATE ROLE <rolename>;` 

将一个 role a grant to role b, b 会继承 a 的权限。b 是 a 的一个 member

```sql
-- 查看 role
\du 
select * from pg_roles;

-- 创建 role 可以登录
-- 常用 option SUPERUSER，CREATEDB，CREATEROLE，INHERIT，[ ENCRYPTED ] PASSWORD 'password'
CREATE ROLE name [ [ WITH ] option [ ... ] ]
CREATE ROLE service_order WITH LOGIN PASSWORD '123456';


-- 修改 role
ALTER ROLE service_order WITH INHERIT SUPERUSER;


-- 删除 role
DROP ROLE service_order;

-- 查询某个 role 下有哪些子 role
SELECT pr.rolname parent_role,pr2.rolname son_role
    FROM pg_roles pr
    INNER JOIN pg_auth_members pam ON pr.oid = pam.roleid
    INNER JOIN pg_roles pr2 ON pr2.oid = pam."member"
    WHERE pr.rolname = 'postgres';
-- role1 获得 group_role 的权限   
GRANT group_role TO role1, ... ;
REVOKE group_role FROM role1, ... ;    
```

### 权限

postgresql 中创建的表，默认只有这个表的 owner(默认是创建者) 可以访问

```sql
CRETAE  TABLE t1 (id int);
ALTER TABLE t1 OWNER TO postgres;

-- 将某个 role 的 owner 权限全部转移，然后
REASSIGN OWNED BY doomed_role TO successor_role;
DROP OWNED BY doomed_role;
-- repeat the above commands in each database of the cluster
DROP ROLE doomed_role;


-- 把新建表的所有权限赋予 database_role
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT ALL ON TABLES TO database_role;

-- 把 public 下现有的表所有权限赋予 database_role
GRANT ALL ON ALL TABLES IN SCHEMA public TO database_role;
```





### aws postgres

[创建一个最大权限的 role](https://aws.amazon.com/cn/premiumsupport/knowledge-center/rds-aurora-postgresql-clone-master-user/)

aws postgres 默认创建了一个超级管理员用于它自身数据库维护。

**rds_superuser** 角色拥有 RDS 数据库实例的最大权限。



## 基础操作

### docker 启动 postgresql

```shell
# 启动 colima ，本地可以执行 docker 命令
colima start
docker pull postgres:14.3-alpine

# 启动数据库
docker volume create fly-postgres-data && docker run --name fly-postgres \
-e POSTGRES_USER=postgres \
-e POSTGRES_PASSWORD=123456 \
-e POSTGRES_DB=sample \
-p 5432:5432 \
-v fly-postgres-data:/var/lib/postgresql/data \
-d postgres:14.3-alpine

# 关闭数据库
docker container stop fly-postgres && docker container rm fly-postgres && docker volume rm fly-postgres-data


psql -h localhost -U postgres -d postgres
```

### Psql 安装及链接数据库

```shell
brew install libpq 
brew link --force libpq

# psql postgres://username:password@host:port/dbname  
# psql -U username -h hostname -p port -d dbname 
psql postgres://postgres:123456@localhost:5432/sample
psql -h localhost -U postgres -d sample

```

### psql操作

```sql
-- 显示数据库下表的信息
\d

-- 显示某个表的表结构
\d tablename

-- 显示索引
\di
-- 自动补全，两次 tab
-- 查看用户信息
\du
-- 查看表的访问权限
\z
```
