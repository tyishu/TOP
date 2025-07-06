# Technical Walkthrough

Step-by-step technical guide to implement our ThreatOps Platform.

## Table of Contents

1. Environment Setup & eBPF Probe
2. OCaml Detection Engine  
3. Prometheus Exporter & Visualization  
4. Policy-as-Code & Vault Integration  
5. GRC Dashboard & Demo Packaging


## 1. Environment Setup & eBPF Probe

a. Provision a local 3-node Kubernetes cluster.  
b. Install all language runtimes and CLI tools.  
c. Develop, compile, and load a minimal eBPF probe to trace `tcp_connect`/`tcp_close`.  
d. Load prove and verify event collection.

### a. Provisioning a local 3-node Kubernetes cluster.

**Install Kind**  
```bash
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.20.0/kind-linux-amd64
chmod +x ./kind
mv kind /usr/local/bin/
```
**Create cluster**
```bash
kind create cluster --name top-cluster --config demo/k8s_manifests/kind-config.yaml
```
In ***demo/k8s_manifests/kind-config.yaml:***
```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
  - role: control-plane
  - role: worker
  - role: worker
```


### b. Installing all language runtimes and CLI tools.  
```bash
# OCaml & Dune
sudo apt-get update
sudo apt-get install -y ocaml ocaml-native-compilers camlp4-extra opam
opam init --disable-sandboxing
opam switch create 4.12.0
opam install dune

# BCC / libbpf
sudo apt-get install -y bpfcc-tools libbpfcc libbpf-dev

# Vault
wget https://releases.hashicorp.com/vault/1.9.0/vault_1.9.0_linux_amd64.zip
unzip vault_1.9.0_linux_amd64.zip
sudo mv vault /usr/local/bin/

# Prometheus & Grafana
# (uses demo/prometheus-grafana.yml)
docker-compose -f demo/prometheus-grafana.yml up -d

# Node.js (for React)
curl -fsSL https://deb.nodesource.com/setup_14.x | sudo -E bash -
sudo apt-get install -y nodejs

# CLI Tools
sudo apt-get install -y kubectl helm jq
```

### c. Develop, compile, and load a minimal eBPF probe.
In ***demo/ebpf_probe.c:***
```c
#include <uapi/linux/ptrace.h>
#include <net/sock.h>
BPF_PERF_OUTPUT(events);

int trace_connect(struct pt_regs *ctx, struct sock *sk) {
    events.perf_submit(ctx, sk, sizeof(*sk));
    return 0;
}

int trace_close(struct pt_regs *ctx, struct sock *sk) {
    events.perf_submit(ctx, sk, sizeof(*sk));
    return 0;
}
```
**Compile**
```bash
clang -O2 -g -target bpf -c demo/ebpf_probe.c -o demo/ebpf_probe.o
```

### d. Load prove and verify event collection.
In ***src/python_orchestrator/load_probe.py***
```python
from bcc import BPF

# Load and attach
b = BPF(src_file="demo/ebpf_probe.c")
b.attach_kprobe(event="tcp_connect", fn_name="trace_connect")
b.attach_kprobe(event="tcp_close", fn_name="trace_close")

# Print events
def print_event(cpu, data, size):
    print("Event:", b["events"].event(data))

b["events"].open_perf_buffer(print_event)
while True:
    b.perf_buffer_poll()
```
```bash
python3 src/python_orchestrator/load_probe.py
```

**Verify**
```bash
sudo cat /sys/kernel/debug/tracing/trace_pipe
# or generate traffic:
curl http://www.example.com
```
Check events ```trace_connect/trace_close```.
