# 🔁 Port Scanner Results Comparator (Python)

## Overview
A concise Python utility that runs multiple port scanners (`nmap`, `masscan`, `recon-ng`) against a single target and **compares the results** to highlight overlaps and discrepancies.  

This tool is built for lab use, tool validation, and result-parity analysis — useful when you want to know *which scanners agree* on open ports and where differences occur.

---

## Key Features
- 🧰 **Multi-tool orchestration**: Runs `nmap`, `masscan`, and `recon-ng` from a single Python wrapper.  
- 🔎 **Result normalization & comparison**: Extracts open ports from each tool’s output and reports per-tool findings plus the intersection (ports all tools found).  
- 🧪 **Quick parity checks**: Helps evaluate scanner accuracy and detect false positives/negatives across tools.  
- 🧩 **Extendable**: Easy to add more scanners or robust parsing logic (JSON/structured outputs).

---

## Why this script is useful
- Ideal for validating scan results in a controlled environment (lab, training, or tool evaluation).  
- Helps inform decisions on which scanner to trust for a given network or use case.  
- Great for documenting differences during pentest write-ups or research notes.

---

## How it works (high level)
1. The script executes each scanner as a subprocess and captures stdout.  
2. It parses the output to extract lines marked as `open` and collects the ports found by each tool.  
3. It prints each tool’s findings and computes the **common ports** present in the output of all three scanners.

---

## Usage
1. Ensure the required tools are installed and accessible in `PATH`: `nmap`, `masscan`, and `recon-ng`.  
2. Run the script:
```bash
python3 compare_scanner_results.py
# then enter the target host when prompted
````

**Example output**

```
Nmap found the following open ports: ['22', '80', '443']
Masscan found the following open ports: ['80', '443']
Recon-ng found the following open ports: ['22', '443']
The following ports were found open by all three tools: ['443']
```

---

## Caveats & Parsing Notes (important)

* 🧩 **Output formats vary**: The script uses simple text parsing (`split`, `in`, indices). Different tool versions, locales, or verbosity flags can break parsing.
* ⏱️ **Execution environment**: `nmap` and `masscan` often require elevated privileges for raw packet scans; `masscan` can be high-impact on networks.
* ⚖️ **Different scan behaviors**: `nmap` (detailed & slower), `masscan` (very fast, asynchronous), and `recon-ng` (framework module) use different techniques — results are not inherently comparable without normalization (e.g., consistent port lists, timing, and scan types).
* 🔍 **False positives/negatives**: Network conditions, firewall rules, rate limits, and timing can cause variation; interpret results accordingly.

---

## Safety & Ethics (must-read)

* **Only scan systems you own or have explicit permission to test.** Unauthorized scanning is illegal and can disrupt services.
* Use conservative settings on production networks. Notify defenders/stakeholders when running broad or fast scans.

---

## Implementation Note

This script is intentionally simple and readable — a practical starting point for deeper tool-comparison work. Treat it as a lab utility and a foundation for building robust benchmarking pipelines.

---


**Author:** Rasheed Jimoh

**Language:** Python 3

**Intended use:** Tool comparison, lab benchmarking, authorized security testing only.



## Script
```
import subprocess

def run_scan(target_host, scan_tool):
 if scan_tool == "nmap":
 result = subprocess.run(["nmap", "-sS", target_host],
capture_output=True, text=True)
 ports = []
 for line in result.stdout.split("\n"):
 if "open" in line:
 port = line.split("/")[0]
 ports.append(port)
 return ports
 elif scan_tool == "masscan":
 result = subprocess.run(["masscan", target_host],
capture_output=True, text=True)
 ports = []
 for line in result.stdout.split("\n"):
 if "open" in line:
 port = line.split(" ")[3].split("/")[0]
 ports.append(port)
 return ports
 elif scan_tool == "recon-ng":
 result = subprocess.run(["recon-ng", "--no-check", "-m",
"scanner/portscan/tcp", "--workspace", "default", "-e", f"RHOSTS=
{target_host}"], capture_output=True, text=True)
 ports = []
 for line in result.stdout.split("\n"):
 if "open" in line:
 port = line.split(" ")[2]
 ports.append(port)
 return ports
 else:
 return []


def compare_scans(target_host):
 nmap_ports = run_scan(target_host, "nmap")
 masscan_ports = run_scan(target_host, "masscan")
 recon_ports = run_scan(target_host, "recon-ng")
 print(f"Nmap found the following open ports: {nmap_ports}")
 print(f"Masscan found the following open ports: {masscan_ports}")
 print(f"Recon-ng found the following open ports: {recon_ports}")
 common_ports = set(nmap_ports) & set(masscan_ports) & set
(recon_ports)
 if common_ports:
 print(f"The following ports were found open by all three tools:
{list(common_ports)}")
 else:
 print("No common open ports were found by all three tools.")

 
if __name__ == "__main__":
 target_host = input("Enter the target host: ")
 compare_scans(target_host)
