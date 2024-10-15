
# **Ariel - Script for Checking IP Addresses Against DNSBL Blacklists**

Ariel is a Bash script designed to scan a network of IP addresses against multiple DNS Blackhole Lists (DNSBL). The script automates the process of identifying blacklisted IP addresses, logs the results, and sends email notifications to configured recipients.

---

## ✨ Features

- **IP address scanning:** Scan all IP addresses in a specified network to check if they are listed in DNSBL blacklists.
- **Parallel processing:** Supports different priority levels and parallel processing to speed up scans.
- **Logging:** Logs the results of scans for easy tracking and analysis.
- **Email notifications:** Composes and sends summary emails to multiple recipients foncifugred in `email.conf`.
- **Installation function:** Provides an automated installation function to set up necessary directories and copy configuration files.

---

## 📋 Prerequisites

Before installing Ariel, ensure your system meets the following requirements:

- **Operating System:** Linux-based OS with kernel 5/6+.
- **Permissions:** Root or sudo access for installation.
- **Utilities:** The following utilities should be available:
  - `bash`
  - `shuf`

---

## 🛠️ Installation

Ariel comes with installation module script to simplify the process.

1. **Clone the repository:**

   ```bash
   git clone https://github.com/krasimirstoev/ariel.git
   cd ariel
   ```
2. **Run the installation script:**
   ```bash
   /bin/bash ariel --install
   ```
This script will create the necessary directories and copy the configuration files to the appropriate locations.

---

## ⚙️ Configuration files

### dnsbl.conf

-   **Location**: `/etc/ariel/dnsbl.conf`
-   **Description**: Contains a list of DNSBL domains against which IP addresses are checked.
-   **Format**: One domain per line.

**Example**:
```bash
bl.spamcop.net
zen.spamhaus.org
b.barracudacentral.org
```
## Usage 🚀

### Syntax

```bash
/bin/bash ariel [OPTIONS]
```
### Options

-   `--network=NETWORK`: Specifies the network to scan (required).
-   `--priority=PRIORITY`: Sets the priority level (default: 1). Priorities affect the speed and frequency of requests.
-   `--lock`: Enables locking to prevent concurrent runs of the script.
-   `--log_everything`: Logs all activities, including checks for IP addresses not found in blacklists.
-   `--compose_email`: Composes and sends an email with the scan results.
-   `--list=FILE`: Specifies the file containing the list of DNSBL domains (required).
-   `--max_procs=N`: Sets the maximum number of parallel processes (used with `--priority=0`).
-   `--email=ADDRESS`: Specifies an email address to send results to (can be used multiple times).
-   `--install`: Installs the script and necessary files.
-   `--help`: Displays a help message with available options.

### Examples

#### Standard Execution
```bash
/bin/bash ariel --network=192.168.1.0 --list=/etc/ariel/dnsbl.conf --compose_email --email=admin@example.com
```
#### Using Priority and Parallel Processing ⚡⚡⚡
```bash
/bin/bash ariel --network=192.168.1.0 --priority=0 --max_procs=15 --list=/etc/ariel/dnsbl.conf --compose_email --email=admin@example.com
```

**Important**: Using `--priority=0` with `--max_procs` allows the script to perform multiple checks in parallel, significantly speeding up the process. However, this should be used cautiously to avoid overloading the network or being blocked by DNSBL services due to excessive requests.


### Explanation of Priorities 📊

-   **Priority 0**:
    -   **Sleep Interval**: No pauses between requests.
    -   **Behavior**: Maximum parallelism with `--max_procs` controlling the number of concurrent processes.
    -   **Use Case**: Suitable for quick scans but may strain network resources.
-   **Priority 1**:
    -   **Sleep Interval**: Random sleep between **1 and 5 seconds** between requests.
    -   **Behavior**: Low delay, suitable for general use without overloading resources.
-   **Priority 2**:
    -   **Sleep Interval**: Random sleep between **5 and 60 seconds** between requests.
    -   **Behavior**: Moderate delay to reduce load on network and DNSBL servers.
-   **Priority 3**:
    -   **Sleep Interval**: Random sleep between **60 and 240 seconds** (1 to 4 minutes) between requests.
    -   **Behavior**: Longer delays to minimize impact on resources.
-   **Priority 4**:
    -   **Sleep Interval**: Random sleep between **240 and 600 seconds** (4 to 10 minutes) between requests.
    -   **Behavior**: Low-frequency requests, further reducing load.
-   **Priority 5**:
    -   **Sleep Interval**: Random sleep between **240 and 3600 seconds** (4 minutes to 1 hour) between requests.
    -   **Behavior**: Very low frequency, minimal impact on network and DNSBL servers.

## How the Script Works 📝

1.  **Initialization**: The script reads command-line parameters and configuration files.
2.  **IP Address Checking**: For each IP address in the specified network, the script performs DNS queries against all DNSBL domains listed in `dnsbl.conf`.
3.  **Result Processing**: If an IP address is found in a blacklist, the result is recorded and included in the summary email.
4.  **Email Notification**: After completing the checks, the script sends a summary email to all addresses specified via `--email=` argument.
5.  **Logging**: Throughout execution, the script logs information to log files for easy tracking.

## Setting Up Cron Jobs 🕒

Automate the execution of the Ariel script by setting up cron jobs. You can add entries to the `/etc/cron.d/ariel_tasks` file to schedule scans at regular intervals.

### Example Cron Jobs

 **Daily Scan at 2 AM** 
```bash
0 2 * * * root /bin/bash /root/scripts/ariel --network=192.168.1.0 --list=/etc/ariel/dnsbl.conf --compose_email --email=admin@example.com --lock
```

**Weekly Scan with High Priority on Sundays at 3 AM**
```bash
0 3 * * 0 root /bin/bash /root/scripts/ariel --network=192.168.2.0 --priority=2 --list=/etc/ariel/dnsbl.conf --compose_email --email=admin@example.com --lock
```
**Note**: Ensure that the `ariel` script is executable and located in `/root/scripts/` as per the installation instructions.
---

## Logs 📂

### General Information

-   **Location**: `/var/log/ariel/`
-   **Files**:
    -   `general.log`: Contains general information about script execution.
    -   `blacklisted.log`: Contains details about IP addresses found in blacklists.

### Log Entry Examples

#### general.log
```bash
[16.10.2024 10:00:00] Script started for network 192.168.1.0 (Priority: 1, Run type: cron job)
[16.10.2024 10:05:00] No blacklisted IP addresses found for network 192.168.1.0, but email was sent to: admin@example.com security@example.com
[16.10.2024 10:05:01] Script finished for network 192.168.1.0
```
#### blacklisted.log
```bash
[16.10.2024 10:02:30] 192.168.1.15 is blacklisted on bl.spamcop.net
[16.10.2024 10:03:45] 192.168.1.20 is blacklisted on zen.spamhaus.org
```
## ⚠️ Attention When Using `--priority=0` and `--max_procs` ⚠️

Using these options allows the script to perform multiple checks simultaneously, which can speed up the scanning process. **However**:

-   **Network Load**: Running many parallel requests can overload your network or DNS servers.
-   **Blocking by DNSBL Services**: Some DNSBL services may block your IP if they receive too many requests in a short period.

**Recommendation**: Use `--priority=0` and `--max_procs` carefully. Monitor network load and responses from DNSBL servers to ensure you're not exceeding acceptable usage limits.

## Sample Configuration Files 📄

### dnsbl.conf

Located at `/etc/ariel/dnsbl.conf`, this file should contain DNSBL domains, one per line.

**Example**:
```bash
bl.spamcop.net
zen.spamhaus.org
b.barracudacentral.org
dnsbl.sorbs.net
```
### email.conf

Located at `/etc/ariel/email.conf`, this file should contain email addresses, one per line. Empty lines and lines starting with `#` are ignored.

**Example**:
```bash
admin@example.com
security@example.com
support@example.com
# Uncomment the following line to add the IT department
# it@example.com
```
## Sample Email Output 📧

### When Blacklisted IPs Are Found
```
Subject: Scan results for network 192.168.1.0/24
To: admin@example.com, security@example.com

Hello,

The following IP addresses have been found blacklisted:
192.168.1.15 - bl.spamcop.net
192.168.1.20 - zen.spamhaus.org

Please take the necessary actions.

Best regards,
Your automated system
```
### When no blacklisted IPs are found:
```
Subject: Scan results for network 192.168.1.0/24
To: admin@example.com, security@example.com

Hello,

No blacklisted IP addresses were found in the network 192.168.1.0/24.

No further action is required.

Best regards,
Your automated system
```

## Final words 🎉

Ariel is a powerful tool for automated checking of IP addresses against DNSBL blacklists. With proper configuration and usage, it can help maintain the good reputation of your network and promptly address potential issues.

**For questions or assistance**, please contact the developer or submit a Pull Request in the GitHub repository.

----------

**Note**: Ensure that you have the necessary permissions to run the script and that you comply with the usage policies of the DNSBL services you query.
