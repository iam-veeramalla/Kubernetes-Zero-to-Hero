Installing Containerd, runC and CNIplugin:
==============================================

1. Containerd:
   -----------
   Refer the link :

        https://containerd.io/docs/2.3/getting-started/

   and modify the git hub links as per the: containerd-<VERSION>-<OS>-<ARCH>.tar.gz


       curl -LO https://github.com/containerd/containerd/releases/download/v2.3.4/containerd-2.3.4-linux-amd64.tar.gz
   
       sudo tar Cxzvf /usr/local containerd-2.3.4-linux-amd64.tar.gz
   
        curl -LO https://raw.githubusercontent.com/containerd/containerd/main/containerd.service
   
        sudo mkdir -p /usr/local/lib/systemd/system/
   
        sudo mv containerd.service /usr/local/lib/systemd/system/
   
        sudo mkdir -p /etc/containerd
   
       containerd config default | sudo tee /etc/containerd/config.toml
   
       sudo sed -i 's/SystemdCgroup \= false/SystemdCgroup \= true/g' /etc/containerd/config.toml
   
        sudo systemctl daemon-reload
   
       sudo systemctl enable --now containerd

   # Check that containerd service is up and running
     
        systemctl status containerd


3. Install runc
   ------------

   curl -LO https://github.com/opencontainers/runc/releases/download/v1.5.1/runc.amd64



       sudo install -m 755 runc.amd64 /usr/local/sbin/runc


5. CNI Pluin:
   ----------

   curl -LO https://github.com/containernetworking/plugins/releases/download/v1.9.1/cni-plugins-linux-amd64-v1.9.1.tgz

            sudo mkdir -p /opt/cni/bin
            sudo tar Cxzvf /opt/cni/bin cni-plugins-linux-amd64-v1.9.1.tgz
         
