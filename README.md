# Kubernetes
Kubernetes learnings

But more precisely the full flow:


YOU
 ↓
API Server    → "Is this request valid?"
 ↓
etcd          → "Store it"
 ↓
Controller Manager → "I see a change! What should I do?"
 ↓
Scheduler     → "Which node should this pod go to?"
 ↓
kubelet       → "Let me actually run it"
 ↓
Container     → Your app is running!

Controller manager work
Every single controller does this loop forever:

WATCH  → "what is the desired state?" (from etcd)
CHECK  → "what is the actual state?" (from cluster)
ACT    → "make actual = desired"
REPEAT → go back to WATCH

This is called the RECONCILIATION LOOP
It never stops!

Controller Manager ├── Deployment Controller ├── ReplicaSet Controller ├── StatefulSet Controller ├── Job Controller ├── CronJob Controller ├── DaemonSet Controller ├── Namespace Controller ├── Service Account Controller ├── Node Controller └── PersistentVolume Controller

YOU (kubectl apply)
        ↓
API SERVER (validates request)
        ↓
etcd (stores the object)
        ↓
CONTROLLER MANAGER (watches etcd via API server)
"what kind of object is this?"
        │
        ├── Deployment?
        │     ↓
        │   creates ReplicaSet
        │     ↓
        │   ReplicaSet creates Pods
        │     ↓
        │   Scheduler places Pods on Nodes
        │     ↓
        │   kubelet runs containers
        │
        ├── StatefulSet?
        │     ↓
        │   creates Pods with stable names (pod-0, pod-1)
        │     ↓
        │   each Pod gets its own storage
        │     ↓
        │   Scheduler places Pods on Nodes
        │     ↓
        │   kubelet runs containers
        │
        ├── Job?
        │     ↓
        │   creates Pod
        │     ↓
        │   Scheduler places Pod on Node
        │     ↓
        │   kubelet runs container ONCE
        │     ↓
        │   Pod completes and stops
        │
        ├── DaemonSet?
        │     ↓
        │   creates ONE Pod on EVERY node
        │   (bypasses Scheduler!)
        │     ↓
        │   kubelet runs container on each node
        │
        └── Service?
              ↓
            kube-proxy creates network rules
            on every node
            (no pods created, no scheduler involved!)



kubectl apply -f deployment.yaml
        ↓
kubectl converts YAML → JSON
        ↓
HTTP POST → API Server
        ↓
API Server: Authentication, Authorization, Validation
        ↓
API Server converts JSON → protobuf
        ↓
API Server writes to etcd
        ↓
etcd stores and confirms "saved!"
        ↓
API Server returns "deployment created" to kubectl
        ↓
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
YOUR KUBECTL IS DONE HERE!
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
        ↓
SEPARATELY AND SIMULTANEOUSLY:
        ↓
etcd notifies API Server
"hey a new Deployment object was written!"
        ↓
API Server notifies Deployment Controller
(part of Controller Manager)
        ↓
Deployment Controller creates ReplicaSet
writes to etcd via API Server
        ↓
etcd notifies API Server
"hey a new ReplicaSet object was written!"
        ↓
API Server notifies ReplicaSet Controller
        ↓
ReplicaSet Controller creates Pod objects
writes to etcd via API Server
        ↓
etcd notifies API Server
"hey new unassigned Pod objects written!"
        ↓
API Server notifies Scheduler
        ↓
Scheduler picks nodes
updates Pod objects with nodeName
writes to etcd via API Server
        ↓
etcd notifies API Server
"hey Pod objects updated with nodeName!"
        ↓
API Server notifies kubelet on that node
        ↓
kubelet starts container!
        ↓
App is running!
