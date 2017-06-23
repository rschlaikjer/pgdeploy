# pgdeploy
### Postgres schema management tool

## Usage

First install `pgdeploy` with `pip install pgdeploy`, then create a directory
with your migrations, named in the form `[number]_[descriptive_name]`. E.g., I
would create my first migration `001_create_users.sql` and inside put the
following:

```
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username TEXT,
    pw_salt CHAR(128),
    pw_hash CHAR(128)
);

--//@UNDO

DROP TABLE user;
```

Once you have your migrations written out, you can then invoke the migrator as
part of your development / deployment pipeline to bring your database up to date
with the migrations:

```
from pgdeploy.migrator import Migrator

Migrator(
    migration_dir='data/migration/',
    db_host='localhost',
    db_name='my_db',
    db_user='my_db_user',
    db_password='my_db_password'
).run_migrations()
```

Now, if we look in psql, we can see that the table is created:

```
my_db=# \d+ users
                                                  Table "public.users"
  Column  |      Type      |                     Modifiers                      | Storage  | Stats target | Description
----------+----------------+----------------------------------------------------+----------+--------------+-------------
 id       | integer        | not null default nextval('users_id_seq'::regclass) | plain    |              |
 username | text           |                                                    | extended |              |
 pw_salt  | character(128) |                                                    | extended |              |
 pw_hash  | character(128) |                                                    | extended |              |
Indexes:
    "users_pkey" PRIMARY KEY, btree (id)
```

PGDeploy stores database version information in the table `pgdeploy_version`:

```
my_db=# select * from pgdeploy_version;
 migration_version |         applied_at
-------------------+----------------------------
                 1 | 2017-06-23 21:28:10.059298
(3 rows)
```
