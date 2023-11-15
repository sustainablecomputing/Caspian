# Caspian

Caspian is a controller for multi-cluster kubernetes environment that decides on scheduling and placement of workloads so that the total carbon footprint generated by executing the workloads is minimized. Caspian lives in a hub cluster and through a multi-cluster management platform applies its placement and scheduling decisions on workloads to the destination spoke clusters.

![caspian-architecture](https://github.com/sustainablecomputing/Caspian/assets/34821570/fd23f538-9837-43ba-a247-0ce50498e518)


Caspian works in a time slotted manner and has the following main components:
- **Carbon Monitoring:** It periodically fetches the predicted values of carbon intensity of spoke clusters.  
- **Green Scheduler:**   Based on the current and future values of carbon intensity, the status of workloads in the system, and available capacity of spoke clusters in the next T time slots, it calls an optimizer to obtain the best scheduling/placement for the workloads.  Once the Scheduler obtains the solution, it updates the spec of workloads to notify the multi-cluster manager about its decisions. The optimizer uses [clp package](https://github.com/lanl/clp) in its core. Clp  provides a Go interface to the COIN-OR Linear Programming (CLP) librarywhich is part of the [COIN-OR](https://www.coin-or.org/) suite.


## Caspian and MCAD
Caspian uses MicroMCAD as a Workload queueing and multi-cluster management platform to dispatch workloads to the destination clusters. A summary of the interactions between MicroMCAD and Caspian is depicted below.

![caspian-mcad](https://github.com/sustainablecomputing/caspian/assets/34821570/32d1b7e5-c7eb-4cc8-8ce8-d1f1a87b0901)

##  Installation and Setup
### Running outside a cluster
This section explains how to run MCAD and Caspian locally. You’ll need a Kubernetes cluster (as hub) to run against. You can use KIND to get a local cluster for testing, or run against a remote cluster. You will also need a minimum one kubernetes cluster (as spoke) for dispatching workloads.

- **Step 1: Clone multicluster branch of MCAD repository.**
```
git clone git@github.com:tardieu/mcad.git -b multicluster
```

- **Step 2: Clone Caspian repository.**
```
git clone git@github.com:sustainablecomputing/caspian.git
```

- **Step 3: Run MCAD against the hub cluster in dispatching mode.**
```
cd mcad
go run ./cmd/main.go --kube-context=kind-hub --mode=dispatcher --metrics-bind-address=127.0.0.1:8080 --health-probe-bind-address=127.0.0.1:8081

```

- **Step 4: Run MicroMCAD against each spoke cluster in runner mode.**
```
cd mcad
go run ./cmd/main.go --kube-context=kind-spoke1 --mode=runner --metrics-bind-address=127.0.0.1:8082 --health-probe-bind-address=127.0.0.1:8083 --clusterinfo-name=spoke1
```

- **Step 5: Run syncer/syncers to syncing between hub cluster and  spoke cluster/clusters.**
```
cd mcad/syncer
node syncer.js kind-hub kind-spoke1 default spoke1
```

- **Step 5: Run Caspian against the hub cluster.**
```
go run ./caspian/cmd/main.go --kube-context=kind-hub 
```



You’ll need a Kubernetes cluster to run against. You can use KIND to get a local cluster for testing, or run against a remote cluster. 
### Running on cluster

##  How to use it
Once Caspian and MCAD are installed, you can deploy appwrappers in the hub clusters and watch their status.
Caspian looks at the specifications of each appwrapper to determine the total CPU/GPU requirement, the run time, and the deadline for executing the appwrapper. The example below shows an example of an appwrapper. Under sustainale filed, you can specify the run time (in hours) and the deadline. If users does not fill these filed, Caspian by default assumes that the run time of the appwrapper is one hour and there is no deadline for finishing the appwrapper. 

```
apiVersion: workload.codeflare.dev/v1beta1
kind: AppWrapper
metadata:
  namespace: default
  name: aw1
spec:
  priority: 1
  schedulingSpec:
    minAvailable: 1
    requeuing:
      maxNumRequeuings: 5
  sustainable:
    runTime: 3
    deadline: 2023-11-07T17:09:23-08:00
  resources:
    GenericItems:
    - custompodresources:
      - requests:
          cpu: 3
        replicas: 1
      generictemplate:
        apiVersion: v1
        kind: Pod
        metadata:
          namespace: default
          name: aw1-1
          labels:
            workload.codeflare.dev/namespace: default
            workload.codeflare.dev: aw1
        spec:
          restartPolicy: Never
          containers:
            - name: busybox
              image: busybox
              command: ["sh", "-c", "sleep 45"]
              resources:
                requests:
                  cpu: 3
                limits:
                  cpu: 3
 

```


## Publications

- Scheduling on a single cluster:
  - Tayebeh Bahreini, Asser Tantawi and Alaa Youssef, "An Approximation Algorithm for Minimizing the Cloud Carbon Footprint through Workload Scheduling", Proc. of the _*IEEE International Conference on Cloud Computing (IEEE CLOUD)*_, 2022 ([link](https://ieeexplore.ieee.org/abstract/document/9860626/)).
  
 
- Scheduling and placement over multi clusters:
  - Tayebeh Bahreini, Asser Tantawi and Alaa Youssef, "A Carbon-aware Workload Dispatcher in Cloud Computing Systems", Proc. of the _*IEEE International Conference on Cloud Computing (IEEE CLOUD)*_, 2023 ([link](https://ieeexplore.ieee.org/abstract/document/10255008)).
  
 
