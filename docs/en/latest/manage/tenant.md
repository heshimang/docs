---
title: Tenant and Permission
order: 8
---

# Tenant and Permission

All the following operations are performed in the CnosDB Cli.

## Tenant and User

CnosDB provides a tenant system and a user system.

- Only users with system permissions can add users and tenants.

- Only a user who holds the role of owner under a tenant or a user with system permissions can add a role under a tenant and give a user a role.

- CnosDB instance starts with a tenant 'cnosdb' and a user 'root' by default.

## Tenant 

### Show Tenant

#### Example

```sql
SELECT *
FROM cluster_schema.tenants;
```

    +-------------+---------------------------------------------------+
    | tenant_name | tenant_options                                    |
    +-------------+---------------------------------------------------+
    | cnosdb      | {"comment":"system tenant","limiter_config":null} |
    +-------------+---------------------------------------------------+

### Create Tenant

**Syntax**

```sql
CREATE
TENANT [IF NOT EXISTS] tenant_name WITH [comment = ''];
```

**Example**

```sql
CREATE
TENANT test;
SELECT *
FROM cluster_schema.tenants;
```

    +-------------+---------------------------------------------------+
    | tenant_name | tenant_options                                    |
    +-------------+---------------------------------------------------+
    | test        | {"comment":null,"limiter_config":null}            |
    | cnosdb      | {"comment":"system tenant","limiter_config":null} |
    +-------------+---------------------------------------------------+

### Alter Tenant

**Example**

```sql
ALTER
TENANT test SET COMMENT = 'abc';
```

### Drop Tenant

**Syntax**

```sql 
DROP
TENANT tenant_name;
```

**Example**

```sql
DROP
TENANT test;
```

## User

### Show User

#### Example

```sql
SELECT *
FROM cluster_schema.users;
```

    +-----------+----------+-------------------------------------------------------------------------------------------------+
    | user_name | is_admin | user_options                                                                                    |
    +-----------+----------+-------------------------------------------------------------------------------------------------+
    | root      | true     | {"password":"*****","must_change_password":true,"rsa_public_key":null,"comment":"system admin"} |
    +-----------+----------+-------------------------------------------------------------------------------------------------+

### Create User

**Syntax**

```sql
CREATE
USER [IF NOT EXISTS] user_name [WITH [PASSWORD='',] [MUST_CHANGE_PASSWORD=true,] [COMMENT = '']];
```

**Example**

```sql
CREATE
USER IF NOT EXISTS tester WITH PASSWORD='xxx', MUST_CHANGE_PASSWORD=true, COMMENT = 'test';
```

### Alter User

#### Alter User Parameters

**Syntax**

```sql
ALTER
USER user_name SET {PASSWORD = ''| MUST_CHANGE_PASSWORD = {true | false}| COMMENT = ''} = 
```

**Example**

```sql
ALTER
USER tester SET PASSWORD = 'aaa';
ALTER
USER tester SET MUST_CHANGE_PASSWORD = false;
ALTER
USER tester SET COMMENT = 'bbb';
```

### Drop User

**Syntax**

```sql
DROP
USER [IF EXISTS] user_name;
```

**Example**

```sql
DROP
USER IF EXISTS tester;
```


## Role of Tenant

The roles under the tenant are divided into system roles and user-defined roles.

- System roles:
  - Owner: This has top-level permissions on the tenant and supports all actions under the tenant.
  - Member: A tenant member that can browse various objects under a tenant.

- User-defined roles:
  - Custom roles need to inherit from system roles.
  - Multiple permissions can be given to custom roles.

### Show Role

View the roles under the current tenant.

#### Example

```sql
\c
information_schema
SELECT *
FROM roles;
```

    +------------+-----------+--------------+
    | role_name  | role_type | inherit_role |
    +------------+-----------+--------------+
    | owner      | system    |              |
    | member     | system    |              |
    +------------+-----------+--------------+

### Create Role

Only DBA and the owner of tenant can creat a new role, the new role is belonged to tenant.

**Syntax**

```sql
CREATE ROLE [IF NOT EXISTS] role_name INHERIT {owner | member};
```

**Example**

Create the owner_role under the current tenant that inherits from the owner role.

```sql
CREATE ROLE owner_role INHERIT owner;
```

Create the member_role role that inherits from the member role under the current tenant.

```sql
CREATE ROLE member_role INHERIT member;
```

### Drop Role

**Syntax**

```sql
DROP ROLE role_name;
```

**Example**

```sql
DROP ROLE owner_role;
```

## Permission

You can use `GRANT... ` To give permissions to roles under the tenant, use `REVOKE... ` Revoke permissions.

### Database Permission

The current smallest granularity of permissions is the database.

| Name  | Content                                 |
|-------|-----------------------------------------|
| read  | Permission of reading from the database |
| write | Permission of writing to the database   |
| all   | All permission of the database          |

### Grant Permission

**Syntax**

```sql
GRANT
{READ | WRITE | ALL} ON DATABASE database_name TO ROLE role_name;
```

**Example**

```sql
-- create a member rrr
CREATE ROLE rrr INHERIT member;

-- grant read permission of database air to rrr
GRANT
READ
ON DATABASE air TO ROLE rrr;
    
-- grant write permission of database wind to rrr
GRANT WRITE
ON DATABASE wind TO ROLE rrr;
    
-- grant all permission of database sea to rrr
GRANT ALL
ON DATABASE sea TO ROLE rrr;
```

### Show Permission

**Example**

```sql
\c
information_schema
SELECT *
FROM DATABASE_PRIVILEGES;
```

    +-------------+---------------+----------------+-----------+
    | tenant_name | database_name | privilege_type | role_name |
    +-------------+---------------+----------------+-----------+
    | cnosdb      | air           | Read           | rrr       |
    | cnosdb      | sea           | All            | rrr       |
    | cnosdb      | wind          | Write          | rrr       |
    +-------------+---------------+----------------+-----------+

**Notice**

Granting permissions to a role on the same database overwrites previous permissions.

### Revoke Permission

**Syntax**

```sql
REVOKE {WRITE | READ | FULL} ON DATABASE database_name FROM role_name;
```

**Example**

```sql
-- revoke read permission of database air from rrr
REVOKE READ ON DATABASE air FROM rrr;
```

### Alter Role

#### Alter a User With a Role Under a Tenant

**Syntax**

```sql
ALTER
TENANT tenant_name ADD USER user_name AS role_name;
```

**Example**

```sql
CREATE
USER user_a;
ALTER
TENANT cnosdb ADD USER user_a AS ROLE rrr;
```

#### Alter the User Out of the Role Under the Tenant

The role will not be removed only if the user no longer holds the role of tenant.

**Syntax**

```sql
ALTER
TENANT tenant_name REMOVE USER user_name;
```

```sql
ALTER
TENANT cnosdb REMOVE USER user_a;
```








