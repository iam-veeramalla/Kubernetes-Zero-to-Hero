Run the below steps on the Master VM and Worker vm.
====================================================

1. SSH into the Master EC2 server.
2. in ubuntu check in IP we have to install iproute2 and give ip address for Ip
3. set hostname to master name like master: hostnamectl set-hostname master
4. Disable Swap using the below commands:
   
        swapoff -a
        sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab

5. Forwarding IPv4 and letting iptables see bridged traffic:


              cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
              overlay
              br_netfilter
              EOF
              
              sudo modprobe overlay
              sudo modprobe br_netfilter

1. overlay module: The overlay kernel module enables OverlayFS, a union filesystem used by container runtimes such as Docker and containerd.
                  Container images are made up of multiple read-only layers, and OverlayFS provides a writable layer on top of them so containers can modify files without changing the original image.


2. br_netfilter module: br_netfilter allows bridged network traffic to pass through the Linux netfilter framework (iptables/nftables).
                        Without this module, packets forwarded through a Linux bridge may bypass firewall rules.
                        Kubernetes CNIs use Linux bridges extensively, so br_netfilter ensures that kube-proxy and firewall rules can inspect and control Pod traffic.
    


              # sysctl params required by setup, params persist across reboots
              cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
              net.bridge.bridge-nf-call-iptables  = 1
              net.bridge.bridge-nf-call-ip6tables = 1
              net.ipv4.ip_forward                 = 1
              EOF
1. net.bridge.bridge-nf-call-iptables = 1 ----->It allows IPv4 packets that pass through a Linux bridge to be processed by iptables rules.
    Without this setting: iptables may not see the packet.
    with this: Packet --> Bridge --> iptables --> Destination
    This is required for:

    kube-proxy rules
    Network Policies
    Service load balancing
    NAT rules

2. net.bridge.bridge-nf-call-ip6tables = 1 ----->Same concept as above, but for IPv6 packets. This ensures Kubernetes can control and filter IPv6 Pod traffic.

3. net.ipv4.ip_forward = 1 ---> It enables IP forwarding so the Linux host can act like a router and forward packets between network interfaces.

Without it:

          Plain Text
          1
          Node1 receives packet
          2
          ↓
          3
          Packet dropped
          Show more lines
          
          With it:
          
          Plain Text
          1
          Node1 receives packet
          2
          ↓
          3
          Node1 forwards packet
          4
          ↓
          5
          Node2 receives packet
          Show more lines
          
          Example:
          
          Plain Text
          1
          Pod1 (Node1)
          2
          |
          3
          v
          4
          Node1 -----> Node2
          5
          |
          6
          v
          7
          Pod2
          Show more lines
          
          Node1 must forward Pod traffic to Node2. Kubernetes networking would not work correctly without IP forwarding.

  net.bridge.bridge-nf-call-iptables=1 allows bridged IPv4 traffic to be inspected by iptables.
 net.bridge.bridge-nf-call-ip6tables=1 does the same for IPv6 traffic.
 net.ipv4.ip_forward=1 enables the Linux kernel to forward packets between interfaces, 
 allowing the node to act as a router for Pod traffic. These settings are required so Kubernetes networking, kube-proxy rules, services, and network policies work correctly.

              # Apply sysctl params without reboot
              sudo sysctl --system
              
              # Verify that the br_netfilter, overlay modules are loaded by running the following commands:
              lsmod | grep br_netfilter
              lsmod | grep overlay



   

# Verify that the net.bridge.bridge-nf-call-iptables, net.bridge.bridge-nf-call-ip6tables, and net.ipv4.ip_forward system variables are set to 1 in your sysctl config by running the following command:
sysctl net.bridge.bridge-nf-call-iptables net.bridge.bridge-nf-call-ip6tables net.ipv4.ip_forward



