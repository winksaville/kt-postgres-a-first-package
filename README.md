# Kurtosis - Write Yoyr First Package

From: https://docs.kurtosis.com/quickstart-write-a-package/

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


## License

Licensed under either of

- Apache License, Version 2.0 ([LICENSE-APACHE](LICENSE-APACHE) or http://apache.org/licenses/LICENSE-2.0)
- MIT license ([LICENSE-MIT](LICENSE-MIT) or http://opensource.org/licenses/MIT)

### Contribution

Unless you explicitly state otherwise, any contribution intentionally submitted
for inclusion in the work by you, as defined in the Apache-2.0 license, shall
be dual licensed as above, without any additional terms or conditions.
