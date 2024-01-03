# pginfer: Postgres extension support AI inference.

>Integrate AI-model in postgresql database via extension
>
>It offers a example about how to integrate AI in database to support data analysis or other application.
>
>the execution process is closely coupled with the design of AI model, which is not mentioned in this repo. Thus, the extension `.c` file is not generialize well for different AI application.
>
>Note: repo is under development.



### Deploy in ubuntu



**prepare**

we assume you `postgres` has been downloaded and configuraed in your server.

check you can successfully connect to postgres server through `psql`, the `test_db` can be any database you created.

```shell
$ psql --host localhost --username postgres --password --dbname test_db
Password:
psql (16.1 (Ubuntu 16.1-1.pgdg20.04+1))
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, compression: off)
Type "help" for help.

test_db=# 
```



**extension dependency**

For ubuntu, you need these packages:

```shell
sudo apt-get install postgresql-server-dev-all postgresql-common
```

Then add a postgreSQL binary path to your environment, to ensure that `pg_config` is there in path.

In my Ubuntu laptop, I choose the version 16, this is how

```shell
export PATH=/usr/lib/postgresql/16/bin:$PATH
```

make sure that pg_config is executing without specifying th path:

```shell
$ pg_config
```

It will display some configuration settings.

Postgresql installation provides a build infrastructure for extensions, called PGXS, so that simple extension modules can be build simply against an already-installed server. It automates common build rules for simple server extension modules.

```shell
$ pg_config --pgxs
/usr/lib/postgresql/16/lib/pgxs/src/makefiles/pgxs.mk
```



**set an environment variable for postgres process**

all environmet variable in postgres process is stored under  `/etc/postgresql` directory.

More specifically, in my ubuntu, it stored in file `/etc/postgresql/16/main/environment`

```shell
sudo echo "PG_EXTENSION_SHARE_DIR=/usr/share/postgresql/16" >> environment
```

```shell
$ cat environment 
# environment variables for postgres processes
# This file has the same syntax as postgresql.conf:
#  VARIABLE = simple_value
#  VARIABLE2 = 'any value!'
# I. e. you need to enclose any value which does not only consist of letters,
# numbers, and '-', '_', '.' in single quotes. Shell commands are not
# evaluated.
PG_EXTENSION_SHARE_DIR='/usr/share/postgresql/16'
```



**onnxruntime dependency**

download the onnx library in [link](https://github.com/microsoft/onnxruntime/releases), in my ubuntu, I use the ONNXRuntime `v1.16.3`.

file name `onnxruntime-linux-x64-1.16.3.tgz`. unzip and you will get following files

```shell
$ ls onnxruntime-linux-x64-1.16.3 
GIT_COMMIT_ID  include  lib  LICENSE  Privacy.md  README.md  ThirdPartyNotices.txt  VERSION_NUMBER
```

we need to move the header file and library to linux systems' library path

```shell
cp ./include/* /usr/include
cp ./lib/* /usr/lib
```



**build **

```shell
git clone https://github.com/Zrealshadow/pginfer.git
cd pginfer
```



go into `pginfer` directoy.

```shell
$ sudo make clean
rm -f pginfer.so   libpginfer.a  libpginfer.pc
rm -f pginfer_setting.o pginfer_onnx_utils.o pginfer.o pginfer_setting.bc pginfer_onnx_utils.bc pginfer.bc
rm -rf /usr/include/postgresql/16/server/extension_pginfer

$ sudo make install
```

it will build `.so` share library which will be automatically installed into  `$PG_EXTENSION_SHARE_DIR`



**declare extension**

connect to postgresql server.

```shell
$ psql --host localhost --username postgres --password --dbname test_db

test_db=# CREATE EXTENSION pginfer

text_db=# \dx

                      List of installed extensions
  Name   | Version |   Schema   |              Description              
---------+---------+------------+---------------------------------------
 pginfer | 0.0.1   | public     | Apply AI model to data by ONNXRuntime
 plpgsql | 1.0     | pg_catalog | PL/pgSQL procedural language
(2 rows)

```



### Usage

the default `model.onnx` is a classifier for [iris dataset](https://scikit-learn.org/stable/auto_examples/datasets/plot_iris_dataset.html).

through [sql scripts](https://gist.github.com/faustofjunqueira/ba97008616148653a9c633c066edaba9) to insert iris dataset.  execute sql query:

```sql
SELECT
    infer(
        iris.sepal_l,
        iris.sepal_w,
        iris.petal_l,
        iris.petal_w
    ) as target
from
    iris
```



