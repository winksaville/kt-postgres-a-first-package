# Kurtosis - Write Your First Package

From: https://docs.kurtosis.com/quickstart-write-a-package/

Questions:
 * Is there a built in way to export a services name and ports to an environment variable so that if I created an enclave, test1, I could do a GET with something like `curl -X GET "http://localhost:$test1-api-port/actor?first_name=eq.Kevin"` and the `-port` name could be supplied in the "plan"

## Initialize a trivial package

### Step 1: Create a new directory for your package

```bash
mkdir kurtosis-postgres && cd kurtosis-postgres
```

### Step 2: Run the Kurtosis CLI to create a new package

Running the init creates kurtosis.yml and main.star
```bash
wink@3900x 24-04-18T15:01:45.188Z:~/prgs/kt/wyfp/kurtosis-postgres (main)
$ kurtosis package init 
wink@3900x 24-04-18T15:01:53.776Z:~/prgs/kt/wyfp/kurtosis-postgres (main)
$ ls
kurtosis.yml  main.star
wink@3900x 24-04-18T15:01:55.333Z:~/prgs/kt/wyfp/kurtosis-postgres (main)
```

### Step 3: Run the enclave

Which simply prints a message "hello world!" and exits:
```bash
$ kurtosis run --enclave kurtosis-postgres main.star 
INFO[2024-04-18T07:51:14-07:00] Creating a new enclave for Starlark to run inside... 
INFO[2024-04-18T07:51:26-07:00] Enclave 'kurtosis-postgres' created successfully 

Printing a message
hello world!

Starlark code successfully run. No output was returned.

⭐ us on GitHub - https://github.com/kurtosis-tech/kurtosis
INFO[2024-04-18T07:51:28-07:00] ========================================================== 
INFO[2024-04-18T07:51:28-07:00] ||          Created enclave: kurtosis-postgres          || 
INFO[2024-04-18T07:51:28-07:00] ========================================================== 
Name:            kurtosis-postgres
UUID:            a6116c2fb2ce
Status:          RUNNING
Creation Time:   Thu, 18 Apr 2024 07:51:14 PDT
Flags:           

========================================= Files Artifacts =========================================
UUID   Name

========================================== User Services ==========================================
UUID   Name   Ports   Status

wink@3900x 24-04-18T14:51:28.671Z:~/prgs/kt/wyfp/kurtosis-postgres
```

## Change the enclave to run a Postgres container

### Step 1: Remove the "Hello World" enclave

```bash
wink@3900x 24-04-18T15:14:48.755Z:~/prgs/kt/wyfp/kurtosis-postgres (main)
$ kt enclave ls
UUID           Name                Status     Creation Time
892c40a2837a   kurtosis-postgres   RUNNING    Thu, 18 Apr 2024 08:04:23 PDT
wink@3900x 24-04-18T15:14:54.532Z:~/prgs/kt/wyfp/kurtosis-postgres (main)
$ kt clean -a
INFO[2024-04-18T08:15:04-07:00] Cleaning old Kurtosis engine containers...   
INFO[2024-04-18T08:15:04-07:00] Successfully cleaned old Kurtosis engine containers 
INFO[2024-04-18T08:15:04-07:00] Cleaning enclaves...                         
INFO[2024-04-18T08:15:05-07:00] Successfully removed the following enclaves: 
892c40a2837a4948aac510b66dd19e9e	kurtosis-postgres
INFO[2024-04-18T08:15:05-07:00] Successfully cleaned enclaves                
INFO[2024-04-18T08:15:05-07:00] Cleaning unused images...                    
INFO[2024-04-18T08:15:05-07:00] Successfully cleaned unused images           
wink@3900x 24-04-18T15:15:05.165Z:~/prgs/kt/wyfp/kurtosis-postgres (main)
$ kt enclave ls
UUID   Name   Status   Creation Time
wink@3900x 24-04-18T15:15:08.039Z:~/prgs/kt/wyfp/kurtosis-postgres (main)
```

### Step 2: Change main.star to create a plan for a postgres container

```bash
POSTGRES_PORT_ID = "postgres"
POSTGRES_DB = "app_db"
POSTGRES_USER = "app_user"
POSTGRES_PASSWORD = "password"

def run(plan, args):
    # Add a Postgres server
    postgres = plan.add_service(
        name = "postgres",
        config = ServiceConfig(
            image = "postgres:15.2-alpine",
            ports = {
                POSTGRES_PORT_ID: PortSpec(5432, application_protocol = "postgresql"),
            },
            env_vars = {
                "POSTGRES_DB": POSTGRES_DB,
                "POSTGRES_USER": POSTGRES_USER,
                "POSTGRES_PASSWORD": POSTGRES_PASSWORD,
            },
        ),
    )
```

### Step 3: Run the enclave

The enclave it build and is running:
```bash
wink@3900x 24-04-18T15:20:10.132Z:~/prgs/kt/wyfp/kurtosis-postgres (main)
$ kurtosis run --enclave kurtosis-postgres main.star
INFO[2024-04-18T08:21:36-07:00] Creating a new enclave for Starlark to run inside... 
INFO[2024-04-18T08:21:37-07:00] Enclave 'kurtosis-postgres' created successfully 

Container images used in this run:
> postgres:15.2-alpine - locally cached

Adding service with name 'postgres' and image 'postgres:15.2-alpine'
Service 'postgres' added with service UUID '0f387558a69345728f960705ae604def'

Starlark code successfully run. No output was returned.

⭐ us on GitHub - https://github.com/kurtosis-tech/kurtosis
INFO[2024-04-18T08:21:41-07:00] ========================================================== 
INFO[2024-04-18T08:21:41-07:00] ||          Created enclave: kurtosis-postgres          || 
INFO[2024-04-18T08:21:41-07:00] ========================================================== 
Name:            kurtosis-postgres
UUID:            f4994a5bc34f
Status:          RUNNING
Creation Time:   Thu, 18 Apr 2024 08:21:36 PDT
Flags:           

========================================= Files Artifacts =========================================
UUID   Name

========================================== User Services ==========================================
UUID           Name       Ports                                                Status
0f387558a693   postgres   postgres: 5432/tcp -> postgresql://127.0.0.1:32776   RUNNING

wink@3900x 24-04-18T15:21:41.626Z:~/prgs/kt/wyfp/kurtosis-postgres (main)
```

And we also see it using `enclave ls`:
```
$ kt enclave ls
UUID           Name                Status     Creation Time
f4994a5bc34f   kurtosis-postgres   RUNNING    Thu, 18 Apr 2024 08:21:36 PDT
wink@3900x 24-04-18T15:22:24.055Z:~/prgs/kt/wyfp/kurtosis-postgres (main)
```

## Change enclave plan to build and populate the database

The "database is a TAR of DVD rental information, courtesy of postgresqltutorial.com."

### Step 1: Update the plan

Adds code to populate the database:

```bash
data_package_module = import_module("github.com/kurtosis-tech/awesome-kurtosis/data-package/main.star")

POSTGRES_PORT_ID = "postgres"
POSTGRES_DB = "app_db"
POSTGRES_USER = "app_user"
POSTGRES_PASSWORD = "password"

SEED_DATA_DIRPATH = "/seed-data"

def run(plan, args):
    # Make data available for use in Kurtosis
    data_package_module_result = data_package_module.run(plan, {})

    # Add a Postgres server
    postgres = plan.add_service(
        name = "postgres",
        config = ServiceConfig(
            image = "postgres:15.2-alpine",
            ports = {
                POSTGRES_PORT_ID: PortSpec(5432, application_protocol = "postgresql"),
            },
            env_vars = {
                "POSTGRES_DB": POSTGRES_DB,
                "POSTGRES_USER": POSTGRES_USER,
                "POSTGRES_PASSWORD": POSTGRES_PASSWORD,
            },
            files = {
                SEED_DATA_DIRPATH: data_package_module_result.files_artifact,
            }
        ),
    )

    # Load the data into Postgres
    postgres_flags = ["-U", POSTGRES_USER,"-d", POSTGRES_DB]
    plan.exec(
        service_name = "postgres",
        recipe = ExecRecipe(command = ["pg_restore"] + postgres_flags + [
            "--no-owner",
            "--role=" + POSTGRES_USER,
            SEED_DATA_DIRPATH + "/" + data_package_module_result.tar_filename,
        ]),
    )
```

The diff is:
```diff
wink@3900x 24-04-18T15:32:09.611Z:~/prgs/kt/wyfp/kurtosis-postgres (main)
$ git --no-pager diff main.star
diff --git a/main.star b/main.star
index 0561e05..c2af180 100644
--- a/main.star
+++ b/main.star
@@ -1,9 +1,16 @@
+data_package_module = import_module("github.com/kurtosis-tech/awesome-kurtosis/data-package/main.star")
+
 POSTGRES_PORT_ID = "postgres"
 POSTGRES_DB = "app_db"
 POSTGRES_USER = "app_user"
 POSTGRES_PASSWORD = "password"
 
+SEED_DATA_DIRPATH = "/seed-data"
+
 def run(plan, args):
+    # Make data available for use in Kurtosis
+    data_package_module_result = data_package_module.run(plan, {})
+
     # Add a Postgres server
     postgres = plan.add_service(
         name = "postgres",
@@ -17,5 +24,19 @@ def run(plan, args):
                 "POSTGRES_USER": POSTGRES_USER,
                 "POSTGRES_PASSWORD": POSTGRES_PASSWORD,
             },
+            files = {
+                SEED_DATA_DIRPATH: data_package_module_result.files_artifact,
+            }
         ),
     )
+
+    # Load the data into Postgres
+    postgres_flags = ["-U", POSTGRES_USER,"-d", POSTGRES_DB]
+    plan.exec(
+        service_name = "postgres",
+        recipe = ExecRecipe(command = ["pg_restore"] + postgres_flags + [
+            "--no-owner",
+            "--role=" + POSTGRES_USER,
+            SEED_DATA_DIRPATH + "/" + data_package_module_result.tar_filename,
+        ]),
+    )
```

### Step 2: Clean and re-run the enclave

The result will be a new enclave with the database populated.

>Note: By running with "." is allowed because the current directory
has a `kurtosis.yml` file and therefore is a kurtosis package.


```bash
wink@3900x 24-04-18T15:34:02.861Z:~/prgs/kt/wyfp/kurtosis-postgres (main)
$ kurtosis clean -a && kurtosis run --enclave kurtosis-postgres .
INFO[2024-04-18T08:34:06-07:00] Cleaning unused images...                    
INFO[2024-04-18T08:34:06-07:00] Successfully cleaned unused images           
INFO[2024-04-18T08:34:06-07:00] Cleaning old Kurtosis engine containers...   
INFO[2024-04-18T08:34:06-07:00] Successfully cleaned old Kurtosis engine containers 
INFO[2024-04-18T08:34:06-07:00] Cleaning enclaves...                         
INFO[2024-04-18T08:34:07-07:00] Successfully removed the following enclaves: 
f4994a5bc34f42278be2b6569a612b95        kurtosis-postgres
INFO[2024-04-18T08:34:07-07:00] Successfully cleaned enclaves                
INFO[2024-04-18T08:34:07-07:00] Creating a new enclave for Starlark to run inside... 
INFO[2024-04-18T08:34:09-07:00] Enclave 'kurtosis-postgres' created successfully 
INFO[2024-04-18T08:34:09-07:00] Executing Starlark package at '/home/wink/prgs/kt/wyfp/kurtosis-postgres' as the passed argument '.' looks like a directory 
INFO[2024-04-18T08:34:09-07:00] Compressing package 'github.com/example-org/example-package' at '.' for upload 
INFO[2024-04-18T08:34:09-07:00] Uploading and executing package 'github.com/example-org/example-package' 

Container images used in this run:
> postgres:15.2-alpine - locally cached

Uploading file './dvd-rental-data.tar' to files artifact 'tranquil-leaf'
Files with artifact name 'tranquil-leaf' uploaded with artifact UUID '305184a9f7054790a472482328432b1a'

Adding service with name 'postgres' and image 'postgres:15.2-alpine'
Service 'postgres' added with service UUID 'e991ddab3c434167a5ffdb383ba02998'

Executing command on service 'postgres'
Command returned with exit code '0' with no output

Starlark code successfully run. No output was returned.

⭐ us on GitHub - https://github.com/kurtosis-tech/kurtosis
INFO[2024-04-18T08:34:18-07:00] ========================================================== 
INFO[2024-04-18T08:34:18-07:00] ||          Created enclave: kurtosis-postgres          || 
INFO[2024-04-18T08:34:18-07:00] ========================================================== 
Name:            kurtosis-postgres
UUID:            761d6d6ce191
Status:          RUNNING
Creation Time:   Thu, 18 Apr 2024 08:34:07 PDT
Flags:           

========================================= Files Artifacts =========================================
UUID           Name
305184a9f705   tranquil-leaf

========================================== User Services ==========================================
UUID           Name       Ports                                                Status
e991ddab3c43   postgres   postgres: 5432/tcp -> postgresql://127.0.0.1:32779   RUNNING
```

Verify there are tables and data in the database:

```bash
wink@3900x 24-04-18T15:36:13.972Z:~/prgs/kt/wyfp/kurtosis-postgres (main)
$ kurtosis service shell kurtosis-postgres postgres
Found bash on container; creating bash shell...
2f2fc46b126f:/# psql -U app_user -d app_db -c '\dt'
             List of relations
 Schema |     Name      | Type  |  Owner   
--------+---------------+-------+----------
 public | actor         | table | app_user
 public | address       | table | app_user
 public | category      | table | app_user
 public | city          | table | app_user
 public | country       | table | app_user
 public | customer      | table | app_user
 public | film          | table | app_user
 public | film_actor    | table | app_user
 public | film_category | table | app_user
 public | inventory     | table | app_user
 public | language      | table | app_user
 public | payment       | table | app_user
 public | rental        | table | app_user
 public | staff         | table | app_user
 public | store         | table | app_user
(15 rows)

2f2fc46b126f:/# 
```

## Add API

### Modify main.star

```bash
data_package_module = import_module("github.com/kurtosis-tech/awesome-kurtosis/data-package/main.star")

POSTGRES_PORT_ID = "postgres"
POSTGRES_DB = "app_db"
POSTGRES_USER = "app_user"
POSTGRES_PASSWORD = "password"

SEED_DATA_DIRPATH = "/seed-data"

POSTGREST_PORT_ID = "http"

def run(plan, args):
    # Make data available for use in Kurtosis
    data_package_module_result = data_package_module.run(plan, {})

    # Add a Postgres server
    postgres = plan.add_service(
        name = "postgres",
        config = ServiceConfig(
            image = "postgres:15.2-alpine",
            ports = {
                POSTGRES_PORT_ID: PortSpec(5432, application_protocol = "postgresql"),
            },
            env_vars = {
                "POSTGRES_DB": POSTGRES_DB,
                "POSTGRES_USER": POSTGRES_USER,
                "POSTGRES_PASSWORD": POSTGRES_PASSWORD,
            },
            files = {
                SEED_DATA_DIRPATH: data_package_module_result.files_artifact,
            }
        ),
    )

    # Load the data into Postgres
    postgres_flags = ["-U", POSTGRES_USER,"-d", POSTGRES_DB]
    plan.exec(
        service_name = "postgres",
        recipe = ExecRecipe(command = ["pg_restore"] + postgres_flags + [
            "--no-owner",
            "--role=" + POSTGRES_USER,
            SEED_DATA_DIRPATH + "/" + data_package_module_result.tar_filename,
        ]),
    )

    # Add PostgREST
    postgres_url = "postgresql://{}:{}@{}:{}/{}".format(
        POSTGRES_USER,
        POSTGRES_PASSWORD,
        postgres.ip_address,
        postgres.ports[POSTGRES_PORT_ID].number,
        POSTGRES_DB,
    )
    api = plan.add_service(
        name = "api", # Naming our PostgREST service "api"
        config = ServiceConfig(
            image = "postgrest/postgrest:v10.2.0",
            env_vars = {
                "PGRST_DB_URI": postgres_url,
                "PGRST_DB_ANON_ROLE": POSTGRES_USER,
            },
            ports = {POSTGREST_PORT_ID: PortSpec(3000, application_protocol = "http")},
        )
    )
```

Here is the diff:
```diff
wink@3900x 24-04-18T17:29:38.126Z:~/prgs/kt/wyfp/kurtosis-postgres (main)
$ git --no-pager diff main.star
diff --git a/main.star b/main.star
index c2af180..06d7283 100644
--- a/main.star
+++ b/main.star
@@ -7,6 +7,8 @@ POSTGRES_PASSWORD = "password"
 
 SEED_DATA_DIRPATH = "/seed-data"
 
+POSTGREST_PORT_ID = "http"
+
 def run(plan, args):
     # Make data available for use in Kurtosis
     data_package_module_result = data_package_module.run(plan, {})
@@ -40,3 +42,23 @@ def run(plan, args):
             SEED_DATA_DIRPATH + "/" + data_package_module_result.tar_filename,
         ]),
     )
+
+    # Add PostgREST
+    postgres_url = "postgresql://{}:{}@{}:{}/{}".format(
+        POSTGRES_USER,
+        POSTGRES_PASSWORD,
+        postgres.ip_address,
+        postgres.ports[POSTGRES_PORT_ID].number,
+        POSTGRES_DB,
+    )
+    api = plan.add_service(
+        name = "api", # Naming our PostgREST service "api"
+        config = ServiceConfig(
+            image = "postgrest/postgrest:v10.2.0",
+            env_vars = {
+                "PGRST_DB_URI": postgres_url,
+                "PGRST_DB_ANON_ROLE": POSTGRES_USER,
+            },
+            ports = {POSTGREST_PORT_ID: PortSpec(3000, application_protocol = "http")},
+        )
+    )
wink@3900x 24-04-18T17:30:12.169Z:~/prgs/kt/wyfp/kurtosis-postgres (main)
```

### Re-run the enclave

And as you can see now there is a second service name 'api' with the port 3000 exposed
as http://127.0.0.1:32790 :
```bash
wink@3900x 24-04-18T17:34:14.459Z:~/prgs/kt/wyfp/kurtosis-postgres (main)
$ kurtosis clean -a && kurtosis run --enclave kurtosis-postgres .
INFO[2024-04-18T10:39:30-07:00] Cleaning unused images...                    
INFO[2024-04-18T10:39:30-07:00] Successfully cleaned unused images           
INFO[2024-04-18T10:39:30-07:00] Cleaning old Kurtosis engine containers...   
INFO[2024-04-18T10:39:30-07:00] Successfully cleaned old Kurtosis engine containers 
INFO[2024-04-18T10:39:30-07:00] Cleaning enclaves...                         
INFO[2024-04-18T10:39:31-07:00] Successfully removed the following enclaves: 
87b72e1961eb416398c93940071758c3        kurtosis-postgres
INFO[2024-04-18T10:39:31-07:00] Successfully cleaned enclaves                
INFO[2024-04-18T10:39:31-07:00] Creating a new enclave for Starlark to run inside... 
INFO[2024-04-18T10:39:32-07:00] Enclave### Using the `api` service

Adding service with name 'api' and image 'postgrest/postgrest:v10.2.0'
Service 'api' added with service UUID '576ae74ba34840529f2f41aa0a5eeac0'

Starlark code successfully run. No output was returned.

⭐ us on GitHub - https://github.com/kurtosis-tech/kurtosis
INFO[2024-04-18T10:39:40-07:00] ========================================================== 
INFO[2024-04-18T10:39:40-07:00] ||          Created enclave: kurtosis-postgres          || 
INFO[2024-04-18T10:39:40-07:00] ========================================================== 
Name:            kurtosis-postgres
UUID:            fbc020cfc586
Status:          RUNNING
Creation Time:   Thu, 18 Apr 2024 10:39:31 PDT
Flags:           

========================================= Files Artifacts =========================================
UUID           Name
3ec305d93857   holy-flower

========================================== User Services ==========================================
UUID           Name       Ports                                                Status
576ae74ba348   api        http: 3000/tcp -> http://127.0.0.1:32790             RUNNING
396d648b060f   postgres   postgres: 5432/tcp -> postgresql://127.0.0.1:32789   RUNNING

wink@3900x 24-04-18T17:39:40.533Z:~/prgs/kt/wyfp/kurtosis-postgres (main)
```

### Query the database

Here is a simple GET where we find actors with the first name of "Kevin":

>Note: I used the `jq` command to format the JSON output and also I set an
environment variable `API_PORT` to the port number of the `api` service.
Use the `enclave inspect ENCLAVE_NAME` to get the port numbers.

```bash
$ API_PORT=32790; curl -X GET "http://localhost:$API_PORT/actor?first_name=eq.Kevin" | jq
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   199    0   199    0     0  76450      0 --:--:-- --:--:-- --:--:-- 99500
[
  {
    "actor_id": 25,
    "first_name": "Kevin",
    "last_name": "Bloom",
    "last_update": "2013-05-26T14:47:57.62"
  },
  {
    "actor_id": 127,
    "first_name": "Kevin",
    "last_name": "Garland",
    "last_update": "2013-05-26T14:47:57.62"
  }
]
wink@3900x 24-04-18T20:05:33.029Z:~/prgs/kt/wyfp/kurtosis-postgres (main)
$ 
```

## For PostgREST the swagger output is at the root of the service

You can use `curl -X GET` to access it:
```bash
wink@3900x 24-04-18T17:56:53.817Z:~/prgs/kt/wyfp/kurtosis-postgres (main)
$ curl -X GET "http://127.0.0.1:32790" | jq > PostgREST.json
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 69260    0 69260    0     0  2001k      0 --:--:-- --:--:-- --:--:-- 2049k
wink@3900x 24-04-18T17:57:56.126Z:~/prgs/kt/wyfp/kurtosis-postgres (main)
$ ls -lh PostgREST.json 
-rw-r--r-- 1 wink users 118K Apr 18 10:57 PostgREST.json
wink@3900x 24-04-18T17:59:30.607Z:~/prgs/kt/wyfp/kurtosis-postgres (main)
$ cat PostgREST.json | wc -l
5008
wink@3900x 24-04-18T18:02:31.448Z:~/prgs/kt/wyfp/kurtosis-postgres (main)
```

Results showing the first and last 20 lies of the PostgREST.json file:
```bash
wink@3900x 24-04-18T18:13:25.149Z:~/prgs/kt/wyfp/kurtosis-postgres (main)
$ head -20 PostgREST.json 
{
  "swagger": "2.0",
  "info": {
    "description": "standard public schema",
    "title": "PostgREST API",
    "version": "10.2.0"
  },
  "host": "0.0.0.0:3000",
  "basePath": "/",
  "schemes": [
    "http"
  ],
  "consumes": [
    "application/json",
    "application/vnd.pgrst.object+json",
    "text/csv"
  ],
  "produces": [
    "application/json",
    "application/vnd.pgrst.object+json",
wink@3900x 24-04-18T18:13:31.913Z:~/prgs/kt/wyfp/kurtosis-postgres (main)      
$ tail -20 PostgREST.json 
    "rowFilter.city.country_id": {
      "name": "country_id",
      "required": false,
      "format": "smallint",
      "in": "query",
      "type": "string"
    },
    "rowFilter.city.last_update": {
      "name": "last_update",
      "required": false,
      "format": "timestamp without time zone",
      "in": "query",
      "type": "string"
    }
  },
  "externalDocs": {
    "description": "PostgREST Documentation",
    "url": "https://postgrest.org/en/v10.2/api.html"
  }
}
wink@3900x 24-04-18T18:14:41.734Z:~/prgs/kt/wyfp/kurtosis-postgres (main)
```

## Use the PostgREST API to modify the database

API_PORT=32802; curl -X POST -H "content-type: application/json" http://127.0.0.1:$API_PORT/actor --data '{"first_name": "Kevin", "last_name": "Bacon"}'

### Set API_PORT for the `api` service

```bash
kt enclave inspect kurtosis-postgreswink@3900x 24-04-18T20:26:47.471Z:~/prgs/kt/wyfp/kurtosis-postgres (main)
$ kt enclave inspect kurtosis-postgres
Name:            kurtosis-postgres
UUID:            9c93d04ba13b
Status:          RUNNING
Creation Time:   Thu, 18 Apr 2024 12:57:34 PDT
Flags:           

========================================= Files Artifacts =========================================
UUID           Name
38d255fc527f   winter-ivy

========================================== User Services ==========================================
UUID           Name       Ports                                                Status
bb9e41a2ce36   api        http: 3000/tcp -> http://127.0.0.1:32802             RUNNING
6e5794a11575   postgres   postgres: 5432/tcp -> postgresql://127.0.0.1:32801   RUNNING

wink@3900x 24-04-18T20:27:04.962Z:~/prgs/kt/wyfp/kurtosis-postgres (main)
$ API_PORT=32802
wink@3900x 24-04-18T20:27:23.182Z:~/prgs/kt/wyfp/kurtosis-postgres (main)
```

### Query the database for actors with the first name of "Kevin"

```bash
wink@3900x 24-04-18T20:27:23.182Z:~/prgs/kt/wyfp/kurtosis-postgres (main)
$ curl -X GET "http://localhost:$API_PORT/actor?first_name=eq.Kevin" | jq
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   199    0   199    0     0  10874      0 --:--:-- --:--:-- --:--:-- 11055
[
  {
    "actor_id": 25,
    "first_name": "Kevin",
    "last_name": "Bloom",
    "last_update": "2013-05-26T14:47:57.62"
  },
  {
    "actor_id": 127,
    "first_name": "Kevin",
    "last_name": "Garland",
    "last_update": "2013-05-26T14:47:57.62"
  }
]
wink@3900x 24-04-18T20:29:03.376Z:~/prgs/kt/wyfp/kurtosis-postgres (main)
```

### Add an actor

Adding Kevin Bacon to the database and now there will be 3 entries:
```bash
wink@3900x 24-04-18T20:29:03.376Z:~/prgs/kt/wyfp/kurtosis-postgres (main)
$ curl -X POST -H "content-type: application/json" http://127.0.0.1:$API_PORT/actor --data '{"first_name": "Kevin", "last_name": "Bacon"}'
wink@3900x 24-04-18T20:30:36.197Z:~/prgs/kt/wyfp/kurtosis-postgres (main)
$ curl -X GET "http://localhost:$API_PORT/actor?first_name=eq.Kevin" | jq
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   303    0   303    0     0   120k      0 --:--:-- --:--:-- --:--:--  147k
[
  {
    "actor_id": 25,
    "first_name": "Kevin",
    "last_name": "Bloom",
    "last_update": "2013-05-26T14:47:57.62"
  },
  {
    "actor_id": 127,
    "first_name": "Kevin",
    "last_name": "Garland",
    "last_update": "2013-05-26T14:47:57.62"
  },
  {
    "actor_id": 201,
    "first_name": "Kevin",
    "last_name": "Bacon",
    "last_update": "2024-04-18T20:30:36.178571"
  }
]
wink@3900x 24-04-18T20:30:43.248Z:~/prgs/kt/wyfp/kurtosis-postgres (main)
```

## License

Licensed under either of

- Apache License, Version 2.0 ([LICENSE-APACHE](LICENSE-APACHE) or http://apache.org/licenses/LICENSE-2.0)
- MIT license ([LICENSE-MIT](LICENSE-MIT) or http://opensource.org/licenses/MIT)

### Contribution

Unless you explicitly state otherwise, any contribution intentionally submitted
for inclusion in the work by you, as defined in the Apache-2.0 license, shall
be dual licensed as above, without any additional terms or conditions.
