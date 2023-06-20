# Creating Namespaces and Connecting with veth Cable

This documentation provides a step-by-step guide on how to create two namespaces and connect them using a veth (Virtual Ethernet) cable. We'll also demonstrate how to ping from one namespace to another. This setup can be useful for network testing, container networking, or simulating network environments. Let's get started!

## Prerequisites

Before proceeding, ensure that you have the following requirements:

  - A Linux-based operating system (e.g., Ubuntu, CentOS)
  - Root or sudo access on your machine
  - Basic knowledge of Linux networking and terminal commands

## Step 1: Set up the Namespaces

Namespaces allow us to create isolated network environments. In this example, we'll create two namespaces: red and green. Follow these steps:

  - Open a terminal or shell.
  - Create the first namespace red using the ip command:

     ``` bash
      sudo ip netns add red
     ```
  - Create the second namespace green:
      ``` bash
      sudo ip netns add green
     ```
  - Show created namespaces
      ``` bash
        sudo ip netns
      ```
      
![ns-list](https://github.com/linckon/create-network-namspace-and-connect-with-veth-cable/assets/12873582/b03f967c-0701-492e-976e-c0e972300afb)



## Step 2: Connect Namespaces using veth Cable

The veth cable is a virtual Ethernet cable that connects two namespaces. We'll create a pair of veth interfaces and assign each end to a different namespace:
  - Create the veth pair rveth and gveth:
  
    ``` bash
      sudo ip link add rveth type veth peer name gveth
    ```
  ![create-veth-cable](https://github.com/linckon/create-network-namspace-and-connect-with-veth-cable/assets/12873582/ad9e4ba0-9297-4adf-a35b-444640fd9e8d)
  
 - Connect gveth to green and rveth to red:
     ``` bash
      sudo ip link set rveth netns red
      sudo ip link set gveth netns green
     ```

     ![connect-ns-to-veth](https://github.com/linckon/create-network-namspace-and-connect-with-veth-cable/assets/12873582/d98ec89f-2d43-4690-8450-f602105f2075)


  - Bring up the loopback & virthual ethernet interface within the red namespace:
     ``` bash
      sudo ip netns exec red bash
      ip link set dev lo up
      ip link set dev rveth up
     ```

  - In red, assign an IP address to rveth:
     ``` bash
      ip addr add 192.168.1.1 dev rveth
     ```

   - Bring up the loopback & virthual ethernet interface within the green namespace:
     ``` bash
      sudo ip netns exec red bash
      ip link set dev lo up
      ip link set dev rveth up
     ```

  - In red, assign an IP address to gveth:
       ``` bash
        ip addr add 192.168.1.2 dev gveth
       ```

![assign-ip-address](https://github.com/linckon/create-network-namspace-and-connect-with-veth-cable/assets/12873582/69c5aaa2-827d-4458-b3b8-49fc17965018)


 ## Step 3: Communicate within the namespaces

   - Add route table to assign ip into corresponding interface.  It is used by the kernel to determine how to forward network packets to their destination.
       - In red namespace :
     
             ``` bash
               ip route add 192.168.1.2 dev rveth
               ip route
             ```
        - In green namespace :
             ``` bash
               ip route add 192.168.1.1 dev gveth
               ip route
             ```
        - ping
            ``` bash
              sudo ip netns exec red ping 192.168.1.2
              sudo ip netns exec green ping 192.168.1.1
            ```
     If the ping is successful, you should see responses indicating a successful connection.

     We have successfully created two namespaces and connected them using a veth cable. We have also tested the connectivity by pinging between the namespaces.
