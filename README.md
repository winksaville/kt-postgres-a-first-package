# Kurtosis - Write Your First Package

From: https://docs.kurtosis.com/quickstart-write-a-package/

> Note: I've added `alias kt='kurtosis'` to my `.bashrc` so I
can use `kt` instead of `kurtosis` in the command lines below.

## Initialize a trivial package

### Step 1: Create a new directory for your package

```bash
mkdir kt-postgres && cd kt-postgres
```

### Step 2: Initialise a new package

Running the init creates kurtosis.yml and main.star
```bash
wink@3900x 24-04-18T23:24:36.126Z:~/prgs/kt/wyfp/kt-postgres (main)
$ kt package init
wink@3900x 24-04-18T23:24:54.380ZZ:~/prgs/kt/wyfp/kt-postgres (main)
$ ls -l
total 8
-rw-r--r-- 1 wink users 151 Apr 19 10:44 kurtosis.yml
-rw-r--r-- 1 wink users  56 Apr 19 10:44 main.star
wink@3900x 24-04-18T23:25:13.338Z:~/prgs/kt/wyfp/kt-postgres (main)
$ cat kurtosis.yml 
name: github.com/example-org/example-package
description: |-
  # github.com/example-org/example-package
  Enter description Markdown here.
replace: {}
wink@3900x 24-04-18T23:25:18.704Z:~/prgs/kt/wyfp/kt-postgres (main)
$ cat main.star 
def run(plan):
    # TODO
    plan.print("hello world!")
```

Which simply prints a message "hello world!" and exits:
```bash
wink@3900x 24-04-18T23:28:35.140Z:~/prgs/kt/wyfp/kt-postgres (main)
$ kt run --enclave kt-progress main.star
INFO[2024-04-18T16:29:41-07:00] Creating a new enclave for Starlark to run inside... 
INFO[2024-04-18T16:29:43-07:00] Enclave 'kt-progress' created successfully   

Printing a message
hello world!

Starlark code successfully run. No output was returned.

⭐ us on GitHub - https://github.com/kurtosis-tech/kurtosis
INFO[2024-04-18T16:29:45-07:00] ==================================================== 
INFO[2024-04-18T16:29:45-07:00] ||          Created enclave: kt-progress          || 
INFO[2024-04-18T16:29:45-07:00] ==================================================== 
Name:            kt-progress
UUID:            0bd919c50f09
Status:          RUNNING
Creation Time:   Thu, 18 Apr 2024 16:29:41 PDT
Flags:           

========================================= Files Artifacts =========================================
UUID   Name

========================================== User Services ==========================================
UUID   Name   Ports   Status

```

## Change the enclave to run a Postgres container

### Step 1: Remove the "Hello World" enclave

```bash
wink@3900x 24-04-18T23:30:43.340Z:~/prgs/kt/wyfp/kt-postgres (main)
$ kt enclave ls
UUID           Name          Status     Creation Time
0bd919c50f09   kt-progress   RUNNING    Thu, 18 Apr 2024 16:29:41 PDT
wink@3900x 24-04-18T23:31:51.692Z:~/prgs/kt/wyfp/kt-postgres (main)
$ kt enclave rm -f kt-progress 
INFO[2024-04-18T16:32:07-07:00] Destroying enclaves...                       
INFO[2024-04-18T16:32:08-07:00] Enclaves successfully destroyed              
wink@3900x 24-04-18T23:32:08.873Z:~/prgs/kt/wyfp/kt-postgres (main)
$ kt enclave ls
UUID   Name   Status   Creation Time
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

```diff
wink@3900x 24-04-18T23:33:27.609Z:~/prgs/kt/wyfp/kt-postgres (main)
$  git diff
diff --git a/main.star b/main.star
index f547a9c..47697c3 100644
--- a/main.star
+++ b/main.star
@@ -1,3 +1,22 @@
-def run(plan):
-    # TODO
-    plan.print("hello world!")
\ No newline at end of file
+POSTGRES_PORT_ID = "postgres"
+POSTGRES_DB = "app_db"
+POSTGRES_USER = "app_user"
+POSTGRES_PASSWORD = "password"
+
+def run(plan, args):
+    # Add a Postgres server
+    postgres = plan.add_service(
+        name = "postgres",
+        config = ServiceConfig(
+            image = "postgres:15.2-alpine",
+            ports = {
+                POSTGRES_PORT_ID: PortSpec(5432, application_protocol = "postgresql"),
+            },
+            env_vars = {
+                "POSTGRES_DB": POSTGRES_DB,
+                "POSTGRES_USER": POSTGRES_USER,
+                "POSTGRES_PASSWORD": POSTGRES_PASSWORD,
+            },
+        ),
+    )
```

### Step 3: Run the enclave

The run the new main.star file in the enclave:
```bash
wink@3900x 24-04-18T23:33:39.997Z:~/prgs/kt/wyfp/kt-postgres (main)
$ kt run --enclave kt-progres main.star
INFO[2024-04-18T16:35:38-07:00] Creating a new enclave for Starlark to run inside... 
INFO[2024-04-18T16:35:40-07:00] Enclave 'kt-progres' created successfully    

Container images used in this run:
> postgres:15.2-alpine - locally cached

Adding service with name 'postgres' and image 'postgres:15.2-alpine'
Service 'postgres' added with service UUID '08ecf18783c049d89cba48308a464c55'

Starlark code successfully run. No output was returned.

⭐ us on GitHub - https://github.com/kurtosis-tech/kurtosis
INFO[2024-04-18T16:35:44-07:00] =================================================== 
INFO[2024-04-18T16:35:44-07:00] ||          Created enclave: kt-progres          || 
INFO[2024-04-18T16:35:44-07:00] =================================================== 
Name:            kt-progres
UUID:            9beb39150242
Status:          RUNNING
Creation Time:   Thu, 18 Apr 2024 16:35:38 PDT
Flags:           

========================================= Files Artifacts =========================================
UUID   Name

========================================== User Services ==========================================
UUID           Name       Ports                                                Status
08ecf18783c0   postgres   postgres: 5432/tcp -> postgresql://127.0.0.1:32813   RUNNING
```

And we also see it using `enclave ls`:
```
wink@3900x 24-04-18T23:35:44.374Z:~/prgs/kt/wyfp/kt-postgres (main)
$ kt enclave ls
UUID           Name         Status     Creation Time
9beb39150242   kt-progres   RUNNING    Thu, 18 Apr 2024 16:35:38 PDT
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
wink@3900x 24-04-18T23:38:49.789Z:~/prgs/kt/wyfp/kt-postgres (main)
$ git diff
diff --git a/main.star b/main.star
index 47697c3..8f3f147 100644
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
@@ -17,6 +24,20 @@ def run(plan, args):
                 "POSTGRES_USER": POSTGRES_USER,
                 "POSTGRES_PASSWORD": POSTGRES_PASSWORD,
             },
+            files = {
+                SEED_DATA_DIRPATH: data_package_module_result.files_artifact,
+            }
         ),
     )
 
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

### Step 2: Remove and re-run the enclave

The result will be a new enclave with the database populated.

>Note: By running with "." is allowed because the current directory
has a `kurtosis.yml` file and therefore is a kurtosis package.


```bash
wink@3900x 24-04-19T00:01:48.895Z:~/prgs/kt/wyfp/kt-postgres (main)
$ kt enclave rm -f kt-postgres && kt run --enclave kt-postgres .
INFO[2024-04-18T17:01:57-07:00] Destroying enclaves...                       
INFO[2024-04-18T17:01:58-07:00] Enclaves successfully destroyed              
INFO[2024-04-18T17:01:58-07:00] Creating a new enclave for Starlark to run inside... 
INFO[2024-04-18T17:02:00-07:00] Enclave 'kt-postgres' created successfully   
INFO[2024-04-18T17:02:00-07:00] Executing Starlark package at '/home/wink/prgs/kt/wyfp/kt-postgres' as the passed argument '.' looks like a directory 
INFO[2024-04-18T17:02:00-07:00] Compressing package 'github.com/example-org/example-package' at '.' for upload 
INFO[2024-04-18T17:02:00-07:00] Uploading and executing package 'github.com/example-org/example-package' 

Container images used in this run:
> postgres:15.2-alpine - locally cached

Uploading file './dvd-rental-data.tar' to files artifact 'restless-driftwood'
Files with artifact name 'restless-driftwood' uploaded with artifact UUID '45b3e904d5e547a8bc7b32b444dc4012'

Adding service with name 'postgres' and image 'postgres:15.2-alpine'
Service 'postgres' added with service UUID '77ef20a95ef948a998f15a502e99628c'

Executing command on service 'postgres'
Command returned with exit code '0' with no output

Starlark code successfully run. No output was returned.

⭐ us on GitHub - https://github.com/kurtosis-tech/kurtosis
INFO[2024-04-18T17:02:07-07:00] ==================================================== 
INFO[2024-04-18T17:02:07-07:00] ||          Created enclave: kt-postgres          || 
INFO[2024-04-18T17:02:07-07:00] ==================================================== 
Name:            kt-postgres
UUID:            d155f3f10a85
Status:          RUNNING
Creation Time:   Thu, 18 Apr 2024 17:01:58 PDT
Flags:           

========================================= Files Artifacts =========================================
UUID           Name
45b3e904d5e5   restless-driftwood

========================================== User Services ==========================================
UUID           Name       Ports                                                Status
77ef20a95ef9   postgres   postgres: 5432/tcp -> postgresql://127.0.0.1:32819   RUNNING

```

Verify there are tables and data in the database:

```bash
wink@3900x 24-04-19T00:02:59.863Z:~/prgs/kt/wyfp/kt-postgres (main)
$ kt service shell kt-postgres postgres
Found bash on container; creating bash shell...
7481362959aa:/# psql -U app_user -d app_db -c '\dt'
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

7481362959aa:/# 
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
wink@3900x 24-04-19T17:04:41.336Z:~/prgs/kt/wyfp/kt-postgres (main)
$ kt enclave rm -f kt-postgres && kt run --enclave kt-postgres .
INFO[2024-04-19T10:04:59-07:00] Destroying enclaves...                       
INFO[2024-04-19T10:04:59-07:00] Enclaves successfully destroyed              
INFO[2024-04-19T10:04:59-07:00] Creating a new enclave for Starlark to run inside... 
INFO[2024-04-19T10:05:02-07:00] Enclave 'kt-postgres' created successfully   
INFO[2024-04-19T10:05:02-07:00] Executing Starlark package at '/home/wink/prgs/kt/wyfp/kt-postgres' as the passed argument '.' looks like a directory 
INFO[2024-04-19T10:05:02-07:00] Compressing package 'github.com/example-org/example-package' at '.' for upload 
INFO[2024-04-19T10:05:02-07:00] Uploading and executing package 'github.com/example-org/example-package' 

Container images used in this run:
> postgrest/postgrest:v10.2.0 - locally cached
> postgres:15.2-alpine - locally cached

Uploading file './dvd-rental-data.tar' to files artifact 'celestial-deer'
Files with artifact name 'celestial-deer' uploaded with artifact UUID '36dcc7dd2bb84c599e3816acd9095361'

Adding service with name 'postgres' and image 'postgres:15.2-alpine'
Service 'postgres' added with service UUID 'e768d852b42d461091fd857e55e750ee'

Executing command on service 'postgres'
Command returned with exit code '0' with no output

Adding service with name 'api' and image 'postgrest/postgrest:v10.2.0'
Service 'api' added with service UUID '2f608b345e7449bdb22e6ad1f803750f'

Starlark code successfully run. No output was returned.

⭐ us on GitHub - https://github.com/kurtosis-tech/kurtosis
INFO[2024-04-19T10:05:10-07:00] ==================================================== 
INFO[2024-04-19T10:05:10-07:00] ||          Created enclave: kt-postgres          || 
INFO[2024-04-19T10:05:10-07:00] ==================================================== 
Name:            kt-postgres
UUID:            a2caa3acaeb9
Status:          RUNNING
Creation Time:   Fri, 19 Apr 2024 10:04:59 PDT
Flags:           

========================================= Files Artifacts =========================================
UUID           Name
36dcc7dd2bb8   celestial-deer

========================================== User Services ==========================================
UUID           Name       Ports                                                Status
2f608b345e74   api        http: 3000/tcp -> http://127.0.0.1:32770             RUNNING
e768d852b42d   postgres   postgres: 5432/tcp -> postgresql://127.0.0.1:32769   RUNNING

```

### Query the database

To prove there is real data we'll do a simple GET where
we find actors with the first name of "Kevin":

>Note: I used the `jq` command to format the JSON output and also I set an
environment variable `API_PORT` to the port number of the `api` service.
Use the `enclave inspect ENCLAVE_NAME` to get the port numbers.

```bash
wink@3900x 24-04-19T17:08:02.804Z:~/prgs/kt/wyfp/kt-postgres (main)
$ API_PORT=32770; curl -X GET "http://localhost:$API_PORT/actor?first_name=eq.Kevin" | jq
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   199    0   199    0     0  15301      0 --:--:-- --:--:-- --:--:-- 16583
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
```

### Add an actor

Adding another Kevin, Kevin Bacon, to the database
and now there will be 3 entries:

> Note: The `API_PORT` is already set to 32770 we don't need to set it again.
```bash
wink@3900x 24-04-19T17:24:03.748Z:~/prgs/kt/wyfp/kt-postgres (main)
$ echo $API_PORT
32770
wink@3900x 24-04-19T17:24:11.524Z:~/prgs/kt/wyfp/kt-postgres (main)
$ curl -X POST -H "content-type: application/json" http://127.0.0.1:$API_PORT/actor --data '{"first_name": "Kevin", "last_name": "Bacon"}'
wink@3900x 24-04-19T17:24:52.080Z:~/prgs/kt/wyfp/kt-postgres (main)
$ curl -X GET "http://localhost:$API_PORT/actor?first_name=eq.Kevin" | jq
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   303    0   303    0     0   114k      0 --:--:-- --:--:-- --:--:--  147k
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
    "last_update": "2024-04-19T17:24:52.072359"
  }
]
```

## PostgREST OpenAPI description

Beyond the goals of this I accidentally stumbled across
this when I did a GET on the root of the service. So I've
added it here. Here is a relavent page
https://postgrest.org/en/v12/references/api/openapi.html.
```bash
wink@3900x 24-04-19T17:35:50.341Z:~/prgs/kt/wyfp/kt-postgres (main)
$ curl -X GET "http://localhost:$API_PORT" | jq > PostgREST.json
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 69260    0 69260    0     0   511k      0 --:--:-- --:--:-- --:--:--  512k
wink@3900x 24-04-19T17:35:58.434Z:~/prgs/kt/wyfp/kt-postgres (main)
$ cat PostgREST.json | wc -l
5008
```

Here are the first and last 20 lies of the PostgREST.json file:
```bash
wink@3900x 24-04-19T17:36:12.254Z:~/prgs/kt/wyfp/kt-postgres (main)
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
wink@3900x 24-04-19T17:37:31.314Z:~/prgs/kt/wyfp/kt-postgres (main)
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
```

## License

Licensed under either of

- Apache License, Version 2.0 ([LICENSE-APACHE](LICENSE-APACHE) or http://apache.org/licenses/LICENSE-2.0)
- MIT license ([LICENSE-MIT](LICENSE-MIT) or http://opensource.org/licenses/MIT)

### Contribution

Unless you explicitly state otherwise, any contribution intentionally submitted
for inclusion in the work by you, as defined in the Apache-2.0 license, shall
be dual licensed as above, without any additional terms or conditions.
