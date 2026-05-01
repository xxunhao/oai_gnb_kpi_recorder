# OAI Robot Arm

[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)

**OAI Robot Arm** is the 5G base station configuration section of the network remote control project. It supports **millisecond-level base station dataset collection and uplink/downlink PRB configuration**.

## 🚀 Installation Guide

### Prerequisites

The prerequisites vary by component:

- **User Equipment (UE):**
  - Ubuntu 22.04 and ros2 humble
  - RM520N_GL for 5G terminal module

- **gNB (Base Station):**
  - Ubuntu 22.04 and ros2 humble
  - Compatible CPU (e.g., Intel i7 or higher)
  - USRP hardware (e.g., B210)
  - **UHD libraries and drivers** (for USRP support)

- **Core Network (CN) with Edge Server:**
  - Ubuntu 20.04
  - Docker 28.1.1 

- **ROS2 based on OAI:**
  - zenoh RMW (both gNB and UE)
  ```sh
    sudo apt install ros-humble-rmw-zenoh-cpp
   ```

### Installation Steps

#### 1. Core Network (CN)

For detailed guidance, refer to the [NR_SA_Tutorial_OAI_CN5G](oai_custom/doc/NR_SA_Tutorial_OAI_CN5G.md) document. After installation, you need to configure the router.

1. **IP config**
   ```sh
    sudo sysctl net.ipv4.conf.all.forwarding=1
    sudo iptables -P FORWARD ACCEPT
    sudo iptables -F FORWARD
   ```

#### 2. gNB (Base Station)

The gNB installation is based on a modified version of OpenAirInterface (OAI) for this project. For dependency installation, you can check [NR_SA_Tutorial_OAI_COST_UE](oai_custom/doc/NR_SA_Tutorial_COTS_UE.md) document. After the dependencies are installed:

1. **Clone the OAI Robot Arm gNB repository and then**
   ```sh
   cd oai_custom
   ```

4. **Set up the environment**
   ```sh
   source oaienv
   cd cmake_targets
   ```

5. **Compile gNB**
   ```sh
   ./build_oai -I
   ./build_oai -w USRP --gNB --ninja -c
   ```
   **Note:** The compilation may fail due to an asn1c version mismatch. In this case, you can first compile a newer version of OAI gnb, and then compile this project again. In this case, you don't need to run `./build_oai -I`.

6. **Configure gNB**
   - Edit the configuration file (e.g., `robort_arm_b210.conf`) to set the correct frequency band, hardware parameters, and UHD options.

6. **Configure gNB IP**
   ```sh
    sudo sysctl net.ipv4.conf.all.forwarding=1
    sudo iptables -P FORWARD ACCEPT
    sudo iptables -F FORWARD
    sudo ip route add 192.168.70.128/26 via CN_host_IP
   ```

7. **Time synchronization, new terminal**
   ```sh
    sudo systemctl restart chrony
    sudo systemctl enable chrony
    chronyc sources -v
    chronyc tracking
   ```

8. **Run gNB**
   ```sh
   cd ran_build_build
   sudo ./nr-softmodem -O ../../../targets/PROJECTS/GENERIC-NR-5GC/CONF/robort_arm_b210.conf --sa -E
   ```

9. **New terminal, run the zenoh rmw**
   ```sh
    source /opt/ros/humble/setup.bash
    ros2 daemon stop || true
    export ZENOH_CONFIG_OVERRIDE='listen/endpoints=["tcp/0.0.0.0:7447"]'
    ros2 run rmw_zenoh_cpp rmw_zenohd
   ```

10. **New terminal, run the ROS2 related node**
   ```sh
   export RMW_IMPLEMENTATION=rmw_zenoh_cpp
   Waitting for your own ros2 node/topic....
   ```

#### 3. UE
You need to access the installed oai_cn5g/database file and edit the SIM card according to the configuration in the database. Details can be found at [NR_SA_Tutorial_OAI_COST_UE](oai_custom/doc/NR_SA_Tutorial_COTS_UE.md) document. Alternatively, other LTE/5G SIM card editing software can also be used. After the UE successfully accesses the core network (ping CN_hoost_IP from UE for test), then：

1. **Time synchronization**
   ```sh
    sudo systemctl restart chrony
    sudo systemctl enable chrony
    chronyc sources -v
    chronyc tracking
   ```

2. **zenoh rmw, new terminal**
   ```sh
    source /opt/ros/humble/setup.bash
    ros2 daemon stop || true
    export RMW_IMPLEMENTATION=rmw_zenoh_cpp
    export ZENOH_CONFIG_OVERRIDE='mode="client";connect/endpoints=["tcp/192.168.18.196:7447"]'
   ```

2. **ROS2 node start, the same terminal*
   ```sh
   Waitting....
   ```

## 🛠 API Usage

If you want to change the number of PRBs allocated to the UE to further study the impact of communication resource allocation on latency, you can do so using the following two scripts：
   ```sh
   cd oai_custom/
   python gNBControllerUplink.py (only for Uplink)
   python gNBController.py (Downlink is ok) 
   ```


## 📄 License

This project is basd on OAI, licensed under the **MIT License**.

## 🔗 References

- 🌐 This project is based on [OAI](https://gitlab.eurecom.fr/oai/openairinterface5g/) and modifies it, referencing some implementations from the NEU-INTEL Labs.
