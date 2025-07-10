# ThreatOps Platform (TOP)

 - An end-to-end threat detection and response platform for real-time anomaly detection and automated compliance audits.
 - This platform is built in OCaml and Python, hooking directly into Linux kernels via eBPF

## 💪🏼 Capabilities
1. Real‑time anomaly detection on socket‑level events and in‑cluster traffic (sub‑millisecond insights).
2. Automated compliance auditing against NIST CSF / ISO 27001 controls via policy‑as‑code (OPA) and mTLS proofs.
3. Auto‑generates audit evidence (CIS Benchmarks, control‑status heatmaps) for continuous assurance.

## 📦 Contents

```text
TOP/
├── README.md
├── docs/
│   └── technical_walkthrough.md
├── src/
│   ├── ocaml_detector/
│   │   ├── dune-project
│   │   ├── dune
│   │   ├── detector.ml
│   │   ├── Dockerfile
│   │   └── k8s/
│   │       └── ocaml_detector_deployment.yaml
│   ├── python_orchestrator/
│   │   └── load_probe.py
│   └── go_exporter/
│       ├── go.mod
│       ├── go.sum
│       └── exporter.go
├── demo/
│   ├── quickstart.sh
│   ├── ebpf_probe.c
│   ├── ebpf_probe.o
│   ├── k8s_manifests/
│   │   └── kind-config.yaml
│   ├── prometheus-grafana.yml
│   └── vault_config/
│       ├── vault-helm-values.yaml
│       └── init-vault.sh
├── dashboard/
│   └── react_app/
└── videos/
    └── demo.mp4
```

## 🤏🏼 Pre-requisites
- **Kubernetes** (minikube or kind) ≥ 1.25  
- **OCaml** ≥ 4.12 and `dune` build tool  
- **BCC / libbpf** for eBPF development  
- **Prometheus** & **Grafana** (locally or via Docker Compose)  
- **HashiCorp Vault** ≥ 1.9  
- **Node.js** ≥ 14 for the React dashboard  
- `kubectl`, `helm`, and `jq` CLI tools  

## ⚡ Quickstart
1. Clone the repo:  
   ```
   git clone https://github.com/tyishu/TOP.git
   cd TOP
   ```
   
2. Spin up and run the demo cluster and services:
```
chmod +x demo/quickstart.sh
./demo/quickstart.sh
```
4. Load the eBPF probe and start the OCaml detector:
``` make -C src/ocaml_detector load-probe run ```

5. View dashboards of:
   - Grafana: ```http://localhost:3000``` (admin/admin)
   - GRC Dashboard: ```http://localhost:4000```
     
6. Trigger a policy violation to see audit-report generation:
```
curl -X POST http://localhost:8080/api/violations \
  -H 'Content-Type: application/json' \
  -d '{"policy":"no-plain-tls"}'
```
