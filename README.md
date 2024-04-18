# Kurtosis - Write Yoyr First Package

From: https://docs.kurtosis.com/quickstart-write-a-package/

## Step 1: Create a new directory for your package

```bash
mkdir kurtosis-postgres && cd kurtosis-postgres
```

## Step 2: Run the Kurtosis CLI to create a new package

Running the init creates kurtosis.yml and main.star
```bash
wink@3900x 24-04-18T15:01:45.188Z:~/prgs/kt/wyfp/kurtosis-postgres (main)
$ kurtosis package init 
wink@3900x 24-04-18T15:01:53.776Z:~/prgs/kt/wyfp/kurtosis-postgres (main)
$ ls
kurtosis.yml  main.star
wink@3900x 24-04-18T15:01:55.333Z:~/prgs/kt/wyfp/kurtosis-postgres (main)
```

## Step 3: Run the enclave

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

## License

Licensed under either of

- Apache License, Version 2.0 ([LICENSE-APACHE](LICENSE-APACHE) or http://apache.org/licenses/LICENSE-2.0)
- MIT license ([LICENSE-MIT](LICENSE-MIT) or http://opensource.org/licenses/MIT)

### Contribution

Unless you explicitly state otherwise, any contribution intentionally submitted
for inclusion in the work by you, as defined in the Apache-2.0 license, shall
be dual licensed as above, without any additional terms or conditions.
