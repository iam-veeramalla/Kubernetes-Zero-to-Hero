Helm-chats
===========

1. What is Helm Chart?
2. Why is Kubernetes essential?
3. The role of Helm Chart in managing K8s
4. Step-by-step guide to installing Helm Chart


What is Helm Chart?
---------------------
* By using this Helm chats we can manage the yaml file's in very efficient way.

* Without Helm chat we can write deployment.yaml and service.yaml files seperetly and run those yaml's with the help of kubectl create cmd.
* But with helm we will use charts.yaml and deployment.yaml and other some helm yaml files and with a single cmd helm chat install we can deploy our workloads with required yaml files.

To Install the helm chat go to official doc.
---------------------------------------------

1. Download your desired version
2. Verify the binary. See Verifying Helm Binaries on this page.
3. Unpack it (tar -zxvf helm-v4.0.0-linux-amd64.tar.gz)
4. Find the helm binary in the unpacked directory, and move it to its desired destination (mv linux-amd64/helm /usr/local/bin/helm)

CMD To check the helm version: helm version

