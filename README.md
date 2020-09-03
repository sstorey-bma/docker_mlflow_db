# docker_mlflow_db
A docker-compose setup to quickly provide MlFlow service with database backend
and potentially reverse proxy frontend for authentication.

### Summary:
Originally based on [Guillaume Androz's 10-Jan-2020 Toward-Data-Science post,
"Deploy MLflow with docker compose"]
(https://towardsdatascience.com/deploy-mlflow-with-docker-compose-8059f16b6039),
with some changes to:
* replace AWS usage with local mapping for artifact store (done)
* replace mysql with postgresql and other options.
3. use nginx to apply htpasswd access control to mlflow website

and overall allowing me to quickly clone to to wherever I'm working.

There are several docker-compose.yaml files in the compose_variations
subdirectory, any of which can be copied onto the docker-compose.yaml in the
root directory to use the desired variation.  I'm sure there's a better way
to do that.  :-)

Future To-dos:
* Add check for whether the env vars are set in shell (or .env file)
before kicking off container - this ia a mistake I comment make myself.
* Let's put this postgres image in a new container defined in a subdir
after all, and add some lines in its Dockerfile that run PG cmdline
routines to add/config an mlflow user so we're not using postgres
admin/owner account for mlflow.  Then we'd have DBADMIN_USER and
DBADMIN_PW as well as MLFLOW_USER and MLFLOW_PW.

### To run:
Set the following env vars in shell first (these are listed in comments at
top of the docker-compose.yaml files).  Set them as desired for your own
system.  These (and the example output below) correspond to the
docker-compose.mlflow_postgres.yaml file in compose_variations; that's
what the root dir's docker-compose.yaml file is by default, running mlflow
with its backend store in postgresql and its artifact store in a local
docker volume.  The database is accessible on the host network rather
than hidden on a backend network (as in other variations in the subdir)
to allow me to directly access the database to view results through 
alternate means than the mlflow website.
```bash
export MLFLOW_PORT=5001
export DB_NAME=mlflowdb
export DB_PORT=5432
export DB_USER=postgres
export DB_PW=<somepassword>        # (choose an actual pw)
```
(or you can put these into an .env file without the 'export's...)

Then start the containers with:
```bash
docker-compose up -d --build 
```
(-d for detached mode, --build to build the underlying containers if needed)
The first time will download/build the containers, but after that it will
generate output similar to the below, which can be seen via
```bash
docker-compose logs -f
```
where the -f acts like in `tail -f`, allowing open-ended streaming of the
new additions to the logs.


```bash
docker_mlflow_db 00:29:37> docker-compose up --build
Creating network "docker_mlflow_db_mydefault" with driver "bridge"
Creating volume "docker_mlflow_db_db_datapg" with default driver
Creating volume "docker_mlflow_db_mlrun_data" with default driver
Building app
Step 1/2 : FROM python:3.7-slim-buster
 ---> d3fbf7fff365
Step 2/2 : RUN pip install mlflow psycopg2-binary pymysql  # boto3
 ---> Using cache
 ---> 82425881b36b

Successfully built 82425881b36b
Successfully tagged mlflow_server:latest
Creating mlflow_db ... done
Creating mlflow_server ... done
Attaching to mlflow_db, mlflow_server
mlflow_db | The files belonging to this database system will be owned by user "postgres".
mlflow_db | This user must also own the server process.
mlflow_db | 
mlflow_db | The database cluster will be initialized with locale "en_US.utf8".
mlflow_db | The default database encoding has accordingly been set to "UTF8".
mlflow_db | The default text search configuration will be set to "english".
mlflow_db | 
mlflow_db | Data page checksums are disabled.
mlflow_db | 
mlflow_db | fixing permissions on existing directory /var/lib/postgresql/data ... ok
mlflow_db | creating subdirectories ... ok
mlflow_db | selecting dynamic shared memory implementation ... posix
mlflow_db | selecting default max_connections ... 100
mlflow_db | selecting default shared_buffers ... 128MB
mlflow_db | selecting default time zone ... Etc/UTC
mlflow_db | creating configuration files ... ok
mlflow_db | running bootstrap script ... ok
mlflow_db | performing post-bootstrap initialization ... ok
mlflow_db | syncing data to disk ... ok
mlflow_db | 
mlflow_db | 
mlflow_db | Success. You can now start the database server using:
mlflow_db | 
mlflow_db |     pg_ctl -D /var/lib/postgresql/data -l logfile start
mlflow_db | 
mlflow_db | initdb: warning: enabling "trust" authentication for local connections
mlflow_db | You can change this by editing pg_hba.conf or using the option -A, or
mlflow_db | --auth-local and --auth-host, the next time you run initdb.
mlflow_db | waiting for server to start....2020-09-03 07:29:43.392 UTC [46] LOG:  starting PostgreSQL 12.4 (Debian 12.4-1.pgdg100+1) on x86_64-pc-linux-gnu, compiled by gcc (Debian 8.3.0-6) 8.3.0, 64-bit
mlflow_db | 2020-09-03 07:29:43.394 UTC [46] LOG:  listening on Unix socket "/var/run/postgresql/.s.PGSQL.5432"
mlflow_db | 2020-09-03 07:29:43.411 UTC [47] LOG:  database system was shut down at 2020-09-03 07:29:43 UTC
mlflow_db | 2020-09-03 07:29:43.421 UTC [46] LOG:  database system is ready to accept connections
mlflow_db |  done
mlflow_db | server started
mlflow_db | CREATE DATABASE
mlflow_db | 
mlflow_db | 
mlflow_db | /usr/local/bin/docker-entrypoint.sh: ignoring /docker-entrypoint-initdb.d/*
mlflow_db | 
mlflow_db | waiting for server to shut down....2020-09-03 07:29:43.665 UTC [46] LOG:  received fast shutdown request
mlflow_db | 2020-09-03 07:29:43.667 UTC [46] LOG:  aborting any active transactions
mlflow_db | 2020-09-03 07:29:43.671 UTC [46] LOG:  background worker "logical replication launcher" (PID 53) exited with exit code 1
mlflow_db | 2020-09-03 07:29:43.675 UTC [48] LOG:  shutting down
mlflow_db | 2020-09-03 07:29:43.690 UTC [46] LOG:  database system is shut down
mlflow_db |  done
mlflow_db | server stopped
mlflow_db | 
mlflow_db | PostgreSQL init process complete; ready for start up.
mlflow_db | 
mlflow_db | 2020-09-03 07:29:43.779 UTC [1] LOG:  starting PostgreSQL 12.4 (Debian 12.4-1.pgdg100+1) on x86_64-pc-linux-gnu, compiled by gcc (Debian 8.3.0-6) 8.3.0, 64-bit
mlflow_db | 2020-09-03 07:29:43.780 UTC [1] LOG:  listening on IPv4 address "0.0.0.0", port 5432
mlflow_db | 2020-09-03 07:29:43.780 UTC [1] LOG:  listening on IPv6 address "::", port 5432
mlflow_db | 2020-09-03 07:29:43.783 UTC [1] LOG:  listening on Unix socket "/var/run/postgresql/.s.PGSQL.5432"
mlflow_db | 2020-09-03 07:29:43.798 UTC [64] LOG:  database system was shut down at 2020-09-03 07:29:43 UTC
mlflow_db | 2020-09-03 07:29:43.807 UTC [1] LOG:  database system is ready to accept connections
mlflow_server | 2020/09/03 07:29:44 INFO mlflow.store.db.utils: Creating initial MLflow database tables...
mlflow_server | 2020/09/03 07:29:44 INFO mlflow.store.db.utils: Updating database tables
mlflow_server | INFO  [alembic.runtime.migration] Context impl PostgresqlImpl.
mlflow_server | INFO  [alembic.runtime.migration] Will assume transactional DDL.
mlflow_server | INFO  [alembic.runtime.migration] Running upgrade  -> 451aebb31d03, add metric step
mlflow_server | INFO  [alembic.runtime.migration] Running upgrade 451aebb31d03 -> 90e64c465722, migrate user column to tags
mlflow_server | INFO  [alembic.runtime.migration] Running upgrade 90e64c465722 -> 181f10493468, allow nulls for metric values
mlflow_server | INFO  [alembic.runtime.migration] Running upgrade 181f10493468 -> df50e92ffc5e, Add Experiment Tags Table
mlflow_server | INFO  [alembic.runtime.migration] Running upgrade df50e92ffc5e -> 7ac759974ad8, Update run tags with larger limit
mlflow_server | INFO  [alembic.runtime.migration] Running upgrade 7ac759974ad8 -> 89d4b8295536, create latest metrics table
mlflow_server | INFO  [89d4b8295536_create_latest_metrics_table_py] Migration complete!
mlflow_server | INFO  [alembic.runtime.migration] Running upgrade 89d4b8295536 -> 2b4d017a5e9b, add model registry tables to db
mlflow_server | INFO  [2b4d017a5e9b_add_model_registry_tables_to_db_py] Adding registered_models and model_versions tables to database.
mlflow_server | INFO  [2b4d017a5e9b_add_model_registry_tables_to_db_py] Migration complete!
mlflow_server | INFO  [alembic.runtime.migration] Running upgrade 2b4d017a5e9b -> cfd24bdc0731, Update run status constraint with killed
mlflow_server | INFO  [alembic.runtime.migration] Running upgrade cfd24bdc0731 -> 0a8213491aaa, drop_duplicate_killed_constraint
mlflow_server | INFO  [alembic.runtime.migration] Running upgrade 0a8213491aaa -> 728d730b5ebd, add registered model tags table
mlflow_server | INFO  [alembic.runtime.migration] Running upgrade 728d730b5ebd -> 27a6a02d2cf1, add model version tags table
mlflow_server | INFO  [alembic.runtime.migration] Running upgrade 27a6a02d2cf1 -> 84291f40a231, add run_link to model_version
mlflow_server | INFO  [alembic.runtime.migration] Context impl PostgresqlImpl.
mlflow_server | INFO  [alembic.runtime.migration] Will assume transactional DDL.
mlflow_server | [2020-09-03 07:29:44 +0000] [8] [INFO] Starting gunicorn 20.0.4
mlflow_server | [2020-09-03 07:29:44 +0000] [8] [INFO] Listening at: http://0.0.0.0:5001 (8)
mlflow_server | [2020-09-03 07:29:44 +0000] [8] [INFO] Using worker: sync
mlflow_server | [2020-09-03 07:29:44 +0000] [11] [INFO] Booting worker with pid: 11
mlflow_server | [2020-09-03 07:29:44 +0000] [13] [INFO] Booting worker with pid: 13
mlflow_server | [2020-09-03 07:29:44 +0000] [14] [INFO] Booting worker with pid: 14
mlflow_server | [2020-09-03 07:29:44 +0000] [15] [INFO] Booting worker with pid: 15
```

We can verify it's all up and ready via:
```bash
docker_mlflow_db 00:10:10> docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
22900b897469        mlflow_server       "sh -c 'mlflow serve…"   2 minutes ago       Up 2 minutes        0.0.0.0:5001->5001/tcp   mlflow_server
8d452ae5dcb5        postgres:latest     "docker-entrypoint.s…"   2 minutes ago       Up 2 minutes        0.0.0.0:5432->5432/tcp   mlflow_db
```

While it's up we can access the MLFlow website via http://localhost:5001 and its database directly via:
```bash
make psqld   # which calls psql within the container
```

Shut it all down via:
```bash
docker_mlflow_db 00:33:26> docker-compose down --volumes
Stopping mlflow_server ... done
Stopping mlflow_db     ... done
Removing mlflow_server ... done
Removing mlflow_db     ... done
Removing network docker_mlflow_db_mydefault
Removing volume docker_mlflow_db_db_datapg
Removing volume docker_mlflow_db_mlrun_data
```

### Other relevant links:

https://github.com/ymym3412/mlflow-docker-compose  
https://medium.com/vantageai/keeping-your-ml-model-in-shape-with-kafka-airflow-and-mlflow-143d20024ba6  

