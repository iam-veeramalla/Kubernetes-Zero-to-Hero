Kind-Cluster-Installtion.(Kubernets in Docker)
================================================

step: 1 
-----

install kubectl from docs: https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/



step: 2
-------

install docker : https://docs.docker.com/engine/install/ubuntu/



Step: 3
-------

install kind cluster from docs: https://kind.sigs.k8s.io/docs/user/quick-start/#installing-from-release-binaries


* After installtion was completed we have to create a cluster in two ways.
  1. imparative way
  2. declarative way

  1. imparative way
     --------------
     * if you use command: " kind create cluster --name cluster_name --image=releasebinarios/docker image "
     * the cluster will be created with one single controle plane it will act both worker and controle plane.
    
  2. declarative way
     ---------------
     * if you use declarative way then we will specify how many controle planes we need and how many worker palnes we need.
     * for that we have to use yaml file.
       EG: 1 # three node (two workers) cluster config:


              kind: Cluster
              apiVersion: kind.x-k8s.io/v1alpha4
              nodes:
              - role: control-plane
              - role: worker
              - role: worker
      
       Eg: 2 # a cluster with 3 control-plane nodes and 3 workers

                  kind: Cluster
                  apiVersion: kind.x-k8s.io/v1alpha4
                  nodes:
                  - role: control-plane
                  - role: control-plane
                  - role: control-plane
                  - role: worker
                  - role: worker
                  - role: worker

     * We can map extra ports to host mechine by mapping these extra ports we will use "Node-Port" service type in KIND.
        
     *You can map extra ports from the nodes to the host machine with extraPortMappings:
        Mapping ports to the host machine 🔗︎

           kind: Cluster
            apiVersion: kind.x-k8s.io/v1alpha4
            nodes:
            - role: control-plane
              extraPortMappings:
              - containerPort: 80
                hostPort: 80
                listenAddress: "0.0.0.0" # Optional, defaults to "0.0.0.0"
                protocol: udp # Optional, defaults to tcp

    *  This can be useful if using NodePort services or daemonsets exposing host ports.

  Note: binding the listenAddress to 127.0.0.1 may affect your ability to access the service.

Note:
----
Kind will not support Network policies for that we have to desable kind Default CNI at the time of cluster creation. we have to include that in yaml file.




  
