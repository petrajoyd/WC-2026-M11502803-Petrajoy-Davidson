# OAI v2.4.0 Build Guide

**OAI Version:** v2.4.0  
**Network:** OAI gNB + nrUE + CN5G (RFsimulator)

**VM environment:**
Recommended: Ubuntu 24.04 LTS (64-bit)
Alternative: Ubuntu 22.04 LTS
Other Linux distributions: Permitted, but not officially validated for this course.
Hardware: x86-64 CPU with AVX2 support.
Virtualization: Native Linux is preferred. Virtual machines are allowed if sufficient CPU and memory resources are available.
All students must use OpenAirInterface v2.4.0 and successfully complete the pre-lab environment verification.
Note: The teaching assistant will provide installation instructions and troubleshooting support primarily for Ubuntu 24.04 LTS. Students using other distributions are responsible for resolving distribution-specific compatibility issues.


# Part A — Pre-Lab Installation (Required)

## Step 1. Download OAI v2.4.0

```bash
cd ~
git clone --branch v2.4.0 --recurse-submodules \
  https://gitlab.eurecom.fr/oai/openairinterface5g.git \
  openairinterface5g-v2.4.0

cd ~/openairinterface5g-v2.4.0
git describe --tags --always
```

## Step 2. Install Docker

```bash
sudo apt update
sudo apt install -y git net-tools curl ca-certificates unzip

sudo install -m 0755 -d /etc/apt/keyrings

sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg \
  -o /etc/apt/keyrings/docker.asc

sudo chmod a+r /etc/apt/keyrings/docker.asc

echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "$VERSION_CODENAME") stable" \
  | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update

sudo apt install -y docker-ce docker-ce-cli containerd.io \
  docker-buildx-plugin docker-compose-plugin

sudo usermod -aG docker "$USER"
```

Log out and log in again, or run:

```bash
newgrp docker
docker run --rm hello-world
```

## Step 3. Build OAI gNB and nrUE

```bash
cd ~/openairinterface5g-v2.4.0

source oaienv

cd cmake_targets

./build_oai -I -w SIMU --ninja --gNB --nrUE -C
```

**My build log** — status: ✅ completed

Start of the build (installing dependencies):

```text
root@joy:~# cd openairinterface5g-v2.4.0/
root@joy:~/openairinterface5g-v2.4.0# ls
CHANGELOG.md  ci-scripts      CMakePresets.json  common           doc     executables  maketags  NOTICE.md  openair1  openair3   pre-commit-clang  README.md  tests
charts        CMakeLists.txt  cmake_targets      CONTRIBUTING.md  docker  LICENSE      nfapi     oaienv     openair2  openshift  radio             targets    tools
root@joy:~/openairinterface5g-v2.4.0# source oaienv
root@joy:~/openairinterface5g-v2.4.0# cd cmake_targets
root@joy:~/openairinterface5g-v2.4.0/cmake_targets# ./build_oai -I -w SIMU --ninja --gNB --nrUE -C
Will install external packages
Will compile gNB
Will compile NR UE
OPENAIR_DIR    = /root/openairinterface5g-v2.4.0
Erased all previously producted files
Installing packages
```

CMake configuration finished and compilation of the targets started:

```text
-- Generating done
-- Build files have been written to: /root/openairinterface5g-v2.4.0/cmake_targets/ran_build/build
Running "cmake --build . --target  rfsimulator nr-softmodem nr-cuup nr-uesoftmodem params_libconfig coding rfsimulator dfts params_yaml vrtsim rf_emulator -- -j8"
[10844/10844] Linking CXX executable nr-softmodem
BUILD SHOULD BE SUCCESSFUL
```

The build ran through all 10844 ninja steps and finished, producing the `nr-softmodem` (gNB) and `nr-uesoftmodem` (nrUE) binaries in `cmake_targets/ran_build/build`.

Verify:

```bash
cd ~/openairinterface5g-v2.4.0/cmake_targets/ran_build/build

test -x nr-softmodem && echo "gNB build OK"
test -x nr-uesoftmodem && echo "nrUE build OK"
```

**My verification output** — ✅ both binaries built:

```text
root@joy:~/openairinterface5g-v2.4.0/cmake_targets# cd ~/openairinterface5g-v2.4.0/cmake_targets/ran_build/build
root@joy:~/openairinterface5g-v2.4.0/cmake_targets/ran_build/build# test -x nr-softmodem && echo "gNB build OK"
gNB build OK
root@joy:~/openairinterface5g-v2.4.0/cmake_targets/ran_build/build# test -x nr-uesoftmodem && echo "nrUE build OK"
nrUE build OK
```

## Step 4. Download OAI 5G Core

```bash
cp -a ~/openairinterface5g-v2.4.0/doc/tutorial_resources/oai-cn5g \
  ~/oai-cn5g

cd ~/oai-cn5g

docker compose config --quiet
docker compose pull
```

**My output** — ✅ compose file valid (`config --quiet` printed no errors) and all images pulled:

```text
root@joy:~/openairinterface5g-v2.4.0/cmake_targets/ran_build/build# cp -a ~/openairinterface5g-v2.4.0/doc/tutorial_resources/oai-cn5g \
  ~/oai-cn5g
root@joy:~/openairinterface5g-v2.4.0/cmake_targets/ran_build/build# cd ~/oai-cn5g
root@joy:~/oai-cn5g# docker compose config --quiet
root@joy:~/oai-cn5g# docker compose pull
[+] pull 94/94
 ✔ Image oaisoftwarealliance/oai-amf:develop    Pulled   51.7s
 ✔ Image oaisoftwarealliance/oai-udm:develop    Pulled   45.6s
 ✔ Image oaisoftwarealliance/oai-upf:develop    Pulled   52.1s
 ✔ Image mysql:8.0                              Pulled   63.8s
 ✔ Image oaisoftwarealliance/trf-gen-cn5g:jammy Pulled   34.3s
 ✔ Image oaisoftwarealliance/oai-udr:develop    Pulled   33.7s
 ✔ Image oaisoftwarealliance/oai-nrf:develop    Pulled   41.0s
 ✔ Image oaisoftwarealliance/ims:latest         Pulled   48.6s
 ✔ Image oaisoftwarealliance/oai-ausf:develop   Pulled   48.9s
 ✔ Image oaisoftwarealliance/oai-smf:develop    Pulled   51.7s
```

## Step 5. Verify CN5G

```bash
cd ~/oai-cn5g

docker compose up -d
docker compose ps
```

Check the Docker network:

```bash
ip -4 addr show dev oai-cn5g
```

Stop the containers after verification:

```bash
docker compose down
```

**My output** — ✅ all 10 CN5G containers started:

```text
root@joy:~/oai-cn5g# docker compose up -d
[+] up 11/11
 ✔ Network oai-cn5g-public-net Created   0.6s
 ✔ Container oai-nrf           Started   3.5s
 ✔ Container oai-ext-dn        Started   2.6s
 ✔ Container ims               Started   1.9s
 ✔ Container mysql             Started   3.0s
 ✔ Container oai-udr           Started   3.4s
 ✔ Container oai-udm           Started   3.6s
 ✔ Container oai-ausf          Started   3.9s
 ✔ Container oai-amf           Started   4.3s
 ✔ Container oai-smf           Started   4.7s
 ✔ Container oai-upf           Started   5.3s
root@joy:~/oai-cn5g# docker compose ps
NAME         IMAGE                                    COMMAND                  SERVICE      CREATED         STATUS                            PORTS
ims          oaisoftwarealliance/ims:latest           "asterisk -fp"           ims          8 seconds ago   Up 7 seconds (healthy)
mysql        mysql:8.0                                "docker-entrypoint.s…"   mysql        8 seconds ago   Up 5 seconds (health: starting)   3306/tcp, 33060/tcp
oai-amf      oaisoftwarealliance/oai-amf:develop      "/openair-amf/bin/oa…"   oai-amf      7 seconds ago   Up 3 seconds (health: starting)   80/tcp, 5342-5344/tcp, 8080/tcp, 9090/tcp, 38412/sctp
oai-ausf     oaisoftwarealliance/oai-ausf:develop     "/openair-ausf/bin/o…"   oai-ausf     8 seconds ago   Up 4 seconds (health: starting)   80/tcp, 5342-5344/tcp, 8080/tcp
oai-ext-dn   oaisoftwarealliance/trf-gen-cn5g:jammy   "/bin/bash -c ' ip r…"   oai-ext-dn   8 seconds ago   Up 6 seconds (healthy)
oai-nrf      oaisoftwarealliance/oai-nrf:develop      "/openair-nrf/bin/oa…"   oai-nrf      8 seconds ago   Up 5 seconds (health: starting)   80/tcp, 5342-5344/tcp, 8080/tcp
oai-smf      oaisoftwarealliance/oai-smf:develop      "/openair-smf/bin/oa…"   oai-smf      7 seconds ago   Up 3 seconds (health: starting)   80/tcp, 5342-5344/tcp, 8080/tcp, 9090/tcp, 8805/udp
oai-udm      oaisoftwarealliance/oai-udm:develop      "/openair-udm/bin/oa…"   oai-udm      8 seconds ago   Up 4 seconds (health: starting)   80/tcp, 5342-5344/tcp, 8080/tcp
oai-udr      oaisoftwarealliance/oai-udr:develop      "/openair-udr/bin/oa…"   oai-udr      8 seconds ago   Up 4 seconds (health: starting)   80/tcp, 8080/tcp
oai-upf      oaisoftwarealliance/oai-upf:develop      "sh /openair-upf/bin…"   oai-upf      7 seconds ago   Up 2 seconds (health: starting)   2152/udp, 8805/udp, 5342-5344/tcp
```

Docker network — the host side of the `oai-cn5g` bridge is `192.168.70.129/26`:

```text
root@joy:~/oai-cn5g# ip -4 addr show dev oai-cn5g
225: oai-cn5g: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    inet 192.168.70.129/26 brd 192.168.70.191 scope global oai-cn5g
       valid_lft forever preferred_lft forever
```

Stopped after verification:

```text
root@joy:~/oai-cn5g# docker compose down
[+] down 11/11
 ✔ Container oai-ext-dn        Removed   0.7s
 ✔ Container oai-upf           Removed   0.9s
 ✔ Container ims               Removed   0.9s
 ✔ Container oai-smf           Removed   0.9s
 ✔ Container oai-amf           Removed   1.2s
 ✔ Container oai-ausf          Removed   0.8s
 ✔ Container oai-udm           Removed   0.7s
 ✔ Container oai-udr           Removed   0.7s
 ✔ Container oai-nrf           Removed   0.7s
 ✔ Container mysql             Removed   1.4s
 ✔ Network oai-cn5g-public-net Removed   0.3s
```

**Pre-Lab Checkpoint** — ✅ Part A completed

Before Lab 3, confirm that:

- [x] OAI version is `v2.4.0` (cloned with `--branch v2.4.0` into `~/openairinterface5g-v2.4.0`).
- [x] `nr-softmodem` and `nr-uesoftmodem` are successfully built (`gNB build OK`, `nrUE build OK`).
- [x] Docker and Docker Compose work (`docker compose config/pull/up/down` all succeeded).
- [x] CN5G containers start successfully (all 10 containers `Up`; `ims` and `oai-ext-dn` already `healthy`).

---

# Part B — Lab 3 Commands (Optional Before Class)

The following commands will be used during Lab 3.

## Terminal 1 — Start CN5G

```bash
cd ~/oai-cn5g

docker compose up -d
docker compose ps
```

**My output** — ✅ all 10 CN5G containers up:

```text
root@joy:~/oai-cn5g# docker compose up -d
[+] up 11/11
 ✔ Network oai-cn5g-public-net Created   0.1s
 ✔ Container oai-nrf           Started   2.1s
 ✔ Container oai-ext-dn        Started   2.7s
 ✔ Container mysql             Started   1.7s
 ✔ Container ims               Started   1.3s
 ✔ Container oai-udr           Started   2.9s
 ✔ Container oai-udm           Started   3.7s
 ✔ Container oai-ausf          Started   4.6s
 ✔ Container oai-amf           Started   5.1s
 ✔ Container oai-smf           Started   5.5s
 ✔ Container oai-upf           Started   6.0s
root@joy:~/oai-cn5g# docker compose ps
NAME         IMAGE                                    COMMAND                  SERVICE      CREATED         STATUS                            PORTS
ims          oaisoftwarealliance/ims:latest           "asterisk -fp"           ims          8 seconds ago   Up 6 seconds (healthy)
mysql        mysql:8.0                                "docker-entrypoint.s…"   mysql        8 seconds ago   Up 6 seconds (health: starting)   3306/tcp, 33060/tcp
oai-amf      oaisoftwarealliance/oai-amf:develop      "/openair-amf/bin/oa…"   oai-amf      7 seconds ago   Up 2 seconds (health: starting)   80/tcp, 5342-5344/tcp, 8080/tcp, 9090/tcp, 38412/sctp
oai-ausf     oaisoftwarealliance/oai-ausf:develop     "/openair-ausf/bin/o…"   oai-ausf     7 seconds ago   Up 3 seconds (health: starting)   80/tcp, 5342-5344/tcp, 8080/tcp
oai-ext-dn   oaisoftwarealliance/trf-gen-cn5g:jammy   "/bin/bash -c ' ip r…"   oai-ext-dn   8 seconds ago   Up 5 seconds (health: starting)
oai-nrf      oaisoftwarealliance/oai-nrf:develop      "/openair-nrf/bin/oa…"   oai-nrf      8 seconds ago   Up 5 seconds (health: starting)   80/tcp, 5342-5344/tcp, 8080/tcp
oai-smf      oaisoftwarealliance/oai-smf:develop      "/openair-smf/bin/oa…"   oai-smf      7 seconds ago   Up 2 seconds (health: starting)   80/tcp, 5342-5344/tcp, 8080/tcp, 9090/tcp, 8805/udp
oai-udm      oaisoftwarealliance/oai-udm:develop      "/openair-udm/bin/oa…"   oai-udm      7 seconds ago   Up 4 seconds (health: starting)   80/tcp, 5342-5344/tcp, 8080/tcp
oai-udr      oaisoftwarealliance/oai-udr:develop      "/openair-udr/bin/oa…"   oai-udr      8 seconds ago   Up 5 seconds (health: starting)   80/tcp, 8080/tcp
oai-upf      oaisoftwarealliance/oai-upf:develop      "sh /openair-upf/bin…"   oai-upf      7 seconds ago   Up 1 second (health: starting)    2152/udp, 8805/udp, 5342-5344/tcp
```

## Terminal 2 — Start gNB

```bash
cd ~/openairinterface5g-v2.4.0/cmake_targets/ran_build/build

sudo ./nr-softmodem \
  -O ../../../targets/PROJECTS/GENERIC-NR-5GC/CONF/gnb.sa.band78.fr1.106PRB.usrpb210.conf \
  --gNBs.[0].min_rxtxtime 6 \
  --rfsim
```

**My output (key excerpts)** — ✅ gNB registered with the AMF, accepted the UE, and set up a PDU session.

Startup, build version and TDD configuration:

```text
CMDLINE: "./nr-softmodem" "-O" "../../../targets/PROJECTS/GENERIC-NR-5GC/CONF/gnb.sa.band78.fr1.106PRB.usrpb210.conf" "--gNBs.[0].min_rxtxtime" "6" "--rfsim"
[UTIL]   running in SA mode (no --phy-test, --do-ra, --nsa option present)
[HW]     Version: Branch: HEAD Abrev. Hash: 91e7030cf8 Date: Thu Dec 4 07:55:07 2025 +0000
[GNB_APP] minTXRXTIME 6
[RRC]    Read in ServingCellConfigCommon (PhysCellId 0, ABSFREQSSB 641280, DLBand 78, ABSFREQPOINTA 640008, DLBW 106,RACH_TargetReceivedPower -96
[RRC]    absoluteFrequencySSB 641280 corresponds to 3619200000 Hz
[NR_MAC] Set TDD configuration period to: 8 DL slots, 3 UL slots, 10 slots per period (NR_TDD_UL_DL_Pattern is 7 DL slots, 2 UL slots, 6 DL symbols, 4 UL symbols)
[NR_PHY] TDD period configuration: slot 7 is FLEXIBLE: DDDDDDFFFFUUUU
[NR_MAC] Command line parameters for OAI UE: -C 3619200000 -r 106 --numerology 1 --band 78 --ssb 516
```

NG Setup with the AMF (`192.168.70.129`) and GTP-U:

```text
[GNB_APP] Parsed IPv4 address for NG AMF: 192.168.70.129
[NGAP]   Send NGSetupRequest to AMF
[NGAP]   Served GUAMIs for AMF OAI-AMF (assoc_id=3674):
[NGAP]      PLMN: MCC=001, MNC=01
[NGAP]   Supported slice (PLMN 0): SST=0x01 SD=000
[NGAP]   Received NGSetupResponse from AMF
[GTPU]   Configuring GTPu address : 192.168.70.129, port : 2152
[NR_RRC] cell PLMN 001.01 Cell ID 12345678 is in service
[HW]     Running as server waiting opposite rfsimulators to connect
```

UE connects over RFsimulator → Random Access → RRC connected:

```text
[HW]     Client connects from ::ffff:127.0.0.1:51355
[NR_PHY] [RAPROC] 443.19 Initiating RA procedure with preamble 44, energy 43.3 dB (I0 0, thres 120), delay 0 start symbol 0 freq index 0
[NR_MAC] 443.19 UE RA-RNTI 010b TC-RNTI a666: initiating RA procedure
[NR_MAC]  444.19 PUSCH with TC_RNTI 0xa666 received correctly
[NR_RRC] [DL] (cellID bc614e, UE ID 1 RNTI a666) Send RRC Setup
[NR_MAC]  445.17 UE a666: Received Ack of Msg4. CBRA procedure succeeded (UE Connected)
[NR_RRC] [UL] (cellID bc614e, UE ID 1 RNTI a666) Received RRCSetupComplete (RRC_CONNECTED reached)
[NGAP]   UE 1: Selected AMF 'OAI-AMF' (assoc_id 3674) through selected PLMN MCC=001 MNC=01
```

Security, UE capabilities and PDU session setup:

```text
[NR_RRC] [--] (cellID bc614e, UE ID 1 RNTI a666) Selected security algorithms: ciphering 0, integrity 2
[NR_RRC] [UL] (cellID bc614e, UE ID 1 RNTI a666) Received Security Mode Complete
[NR_RRC] [UL] (cellID bc614e, UE ID 1 RNTI a666) Received UE capabilities
[NR_RRC] Send message to sctp: NGAP_InitialContextSetupResponse
[NGAP]   PDUSESSIONSetup initiating message
[NR_RRC] Added PDU Session 10, (total nb of sessions = 1)
[NR_RRC] Bearer Context Setup: PDU Session ID=10, incoming TEID=0x00000001, Addr=192.168.70.134
[GTPU]   [91] UE ID 1: Create tunnel TEID incoming 0x3c20137a outgoing 0x1 to remote IPv4 192.168.70.134, IPv6 ::, port 2152
[NR_RRC] [DL] (cellID bc614e, UE ID 1 RNTI a666) Generate RRCReconfiguration (bytes 339, xid 3)
[NR_RRC] [UL] (cellID bc614e, UE ID 1 RNTI a666) Received RRCReconfigurationComplete
[NR_RRC] PDU Session Setup: ID=10, outgoing TEID=0x3c20137a, Addr=192.168.70.129
[NR_MAC] DU received confirmation of successful RRC Reconfiguration
```

Steady-state UE statistics (in-sync, no HARQ errors):

```text
[NR_MAC] Frame.Slot 640.0
UE RNTI a666 CU-UE-ID 1 in-sync PH 48 dB PCMAX 20 dBm, average RSRP -44 (16 meas)
UE a666: UL-RI 1, TPMI 0
UE a666: dlsch_rounds 101/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.00002 MCS (0) 0 CCE fail 4
UE a666: ulsch_rounds 823/0/0/0, ulsch_errors 0, ulsch_DTX 0, BLER 0.00000 MCS (0) 0 (Qm 2 deltaMCS 0 dB) NPRB 5  SNR 51.0 dB CCE fail 0
UE a666: MAC:    TX           2861 RX          18017 bytes
```

Shutdown (CN5G stopped first → SCTP shutdown; then the UE was stopped → `Lost socket` / UL failure; then gNB `Ctrl+C`):

```text
[SCTP]   Received SCTP SHUTDOWN EVENT
[NGAP]   Received unsuccessful result for SCTP association (1), instance 0, cnx_id 1
[GNB_APP] [gNB 0] Received NGAP_DEREGISTERED_GNB_IND: associated AMF 0
...
[HW]     Lost socket
[NR_MAC]  234. 7 UE a666: Detected UL Failure on PUSCH after 10 PUSCH DTX, stopping scheduling
...
^C
** Caught SIGTERM, shutting down
[GNB_APP] stopping nr-softmodem
[NR_MAC] Remove NR rnti 0xa666
Bye.
```

## Terminal 3 — Start nrUE

```bash
cd ~/openairinterface5g-v2.4.0/cmake_targets/ran_build/build

sudo ./nr-uesoftmodem \
  -r 106 \
  --numerology 1 \
  --band 78 \
  -C 3619200000 \
  --uicc0.imsi 001010000000001 \
  --rfsim \
  --rfsimulator.serveraddr 127.0.0.1
```

**My output (key excerpts)** — ✅ UE synchronized, registered with the 5GC and got IP `10.0.0.2`.

Startup and SIM profile:

```text
CMDLINE: "./nr-uesoftmodem" "-r" "106" "--numerology" "1" "--band" "78" "-C" "3619200000" "--uicc0.imsi" "001010000000001" "--rfsim" "--rfsimulator.serveraddr" "127.0.0.1"
[HW]     Version: Branch: HEAD Abrev. Hash: 91e7030cf8 Date: Thu Dec 4 07:55:07 2025 +0000
[SIM]    UICC simulation: IMSI=001010000000001, IMEISV=6754567890123413, ..., DNN=oai, SST=0x01, SD=0xffffff
```

RFsimulator connection, cell search and synchronization:

```text
[HW]     Running as client: will connect to a rfsimulator server side
[HW]     Trying to connect to 127.0.0.1:4043
[HW]     Connection to 127.0.0.1:4043 established
[NR_PHY] Starting cell search with center freq: 3619200000, bandwidth: 106. Scanning for 1 number of GSCN.
[PHY]    Initial sync: pbch decoded sucessfully, ssb index 0
[NR_PHY] Cell Detected with GSCN: 0, SSB SC offset: 516, SSB Ref: 0.000000, PSS Corr peak: 99 dB, PSS Corr Average: 61
[PHY]    Initial sync successful, PCI: 0
[PHY]    UE synchronized! decoded_frame_rx=392 UE->init_sync_frame=1 trashed_frames=48
[NR_RRC] SIB1 decoded
[NR_MAC] Set TDD configuration period to: 8 DL slots, 3 UL slots, 10 slots per period (NR_TDD_UL_DL_Pattern is 7 DL slots, 2 UL slots, 6 DL symbols, 4 UL symbols)
```

Random Access (4-step CBRA) and RRC connection:

```text
[MAC]    Initialization of 4-Step CBRA procedure
[PHY]    PRACH [UE 0] in frame.slot 443.19, placing PRACH in position 2828, Msg1/MsgA-Preamble frequency start 0 (k1 0), preamble_offset 11, first_nonzero_root_idx 0, preambleIndex = 44
[PHY]    [UE 0] RAR-Msg2 decoded
[NR_MAC] [UE 0][RAPROC][444.10] Found RAR with the intended RAPID 44
[NR_MAC] [RAPROC][444.19] RA-Msg3 transmitted
[MAC]    [UE 0][445.10][RAPROC] 4-Step RA procedure succeeded. CBRA: Contention Resolution is successful.
[NR_RRC] [UE0][RAPROC] Logical Channel DL-CCCH (SRB0), Received NR_RRCSetup
[NR_RRC] State = NR_RRC_CONNECTED
[NAS]    Generate Initial NAS Message: Registration Request
```

NAS registration (authentication → security mode → registration accept):

```text
[NAS]    [UE 0] Received NAS_DOWNLINK_DATA_IND type FGS_AUTHENTICATION_REQUEST with length 42
[NAS]    [UE 0] Received NAS_DOWNLINK_DATA_IND type FGS_SECURITY_MODE_COMMAND with length 21
[NR_RRC] Security algorithm is set to nea0
[NR_RRC] Integrity protection algorithm is set to nia2
[NR_RRC] Received Capability Enquiry (gNB 0)
[NAS]    [UE 0] Received NAS_DOWNLINK_DATA_IND type FGS_REGISTRATION_ACCEPT with length 53
[NAS]    Received Registration Accept with result 3GPP
[NR_RRC] 5G-GUTI: AMF pointer 1, AMF Set ID 1, 5G-TMSI 3758669508
[NAS]    Send NAS_UPLINK_DATA_REQ message(RegistrationComplete)
```

PDU session and tunnel interface:

```text
[NAS]    Send NAS_UPLINK_DATA_REQ message(PduSessionEstablishRequest)
[RLC]    Added drb 1 to UE 0
[PDCP]   Added DRB 1 to UE ID 0
[NR_RRC] rrcReconfigurationComplete Encoded 10 bits (2 bytes)
[NAS]    Received PDU Session Establishment Accept, UE IPv4: 10.0.0.2
[OIP]    Interface oaitun_ue1 successfully configured, IPv4 10.0.0.2, IPv6 (null)
```

Steady-state stats and deregistration on `Ctrl+C`:

```text
[NR_MAC] UE 0 RNTI a666 stats sfn: 640.8, cumulated bad DCI 0
    DL harq: 101/0
    UL harq: 824/0 avg code rate 0.1, avg bit/symbol 2.0, avg per TB: (nb RBs 8.1, nb symbols 11.8)
^C[NAS]    [UE 0] Received NAS_DEREGISTRATION_REQ
Press ^C again to trigger immediate shutdown
...
^C[NR_PHY] Returned from ITTI signal handler
[NR_PHY] UE main thread is ending
```

> The UE log also prints derived keys (kausf, kseaf, kamf, knas_int, knas_enc, KgNB) and a full UE-NR-Capability XML dump. I left them out here for readability; they come from the simulated test SIM.

## Terminal 4 — Test Connectivity

```bash
ip addr show oaitun_ue1
```

```bash
ping -I oaitun_ue1 -c 5 192.168.70.135
```

**My output** — ✅ `oaitun_ue1` has `10.0.0.2/24` and the ping to the external DN (`192.168.70.135`) succeeded with 0% loss:

```text
root@joy:/home/joy# ip addr show oaitun_ue1
257: oaitun_ue1: <POINTOPOINT,NOARP,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UNKNOWN group default qlen 500
    link/none
    inet 10.0.0.2/24 scope global oaitun_ue1
       valid_lft forever preferred_lft forever
    inet6 fe80::8103:31b3:8f64:95ba/64 scope link stable-privacy
       valid_lft forever preferred_lft forever
root@joy:/home/joy# ping -I oaitun_ue1 -c 5 192.168.70.135
PING 192.168.70.135 (192.168.70.135) from 10.0.0.2 oaitun_ue1: 56(84) bytes of data.
64 bytes from 192.168.70.135: icmp_seq=1 ttl=63 time=27.7 ms
64 bytes from 192.168.70.135: icmp_seq=2 ttl=63 time=17.1 ms
64 bytes from 192.168.70.135: icmp_seq=3 ttl=63 time=16.8 ms
64 bytes from 192.168.70.135: icmp_seq=4 ttl=63 time=21.4 ms
64 bytes from 192.168.70.135: icmp_seq=5 ttl=63 time=21.3 ms

--- 192.168.70.135 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4006ms
rtt min/avg/max/mdev = 16.765/20.852/27.707/3.965 ms
```

## Stop the Experiment

Stop the gNB and nrUE using `Ctrl+C`.

Then:

```bash
cd ~/oai-cn5g
docker compose down
```

**My output** — ✅ all containers and the network removed:

```text
root@joy:~/oai-cn5g# docker compose down
[+] down 11/11
 ✔ Container oai-ext-dn        Removed   1.1s
 ✔ Container oai-upf           Removed   1.6s
 ✔ Container ims               Removed   1.5s
 ✔ Container oai-smf           Removed   1.1s
 ✔ Container oai-amf           Removed   1.8s
 ✔ Container oai-ausf          Removed   1.2s
 ✔ Container oai-udm           Removed   1.0s
 ✔ Container oai-udr           Removed   1.0s
 ✔ Container oai-nrf           Removed   1.4s
 ✔ Container mysql             Removed   2.6s
 ✔ Network oai-cn5g-public-net Removed   0.3s
```

> In my run, `docker compose down` happened before the gNB was stopped, so the gNB log shows the SCTP association to the AMF closing (`SCTP SHUTDOWN EVENT`) before its own `Ctrl+C`. The guide's order is to stop the gNB and nrUE first, then the core.

## Part B Summary — ✅ Requirement completed

| Check | Result |
|---|---|
| CN5G up | 10 containers started |
| gNB ↔ AMF | `NGSetupResponse` received |
| UE sync | PCI 0 at 3619.2 MHz (band n78, 106 PRB, 30 kHz SCS) |
| Random access | 4-step CBRA succeeded (preamble 44, RNTI `a666`) |
| Registration | Registration Accept, 5G-GUTI assigned |
| PDU session | Session ID 10, UE IP `10.0.0.2` on `oaitun_ue1` |
| Connectivity | 5/5 pings to `192.168.70.135`, 0% loss, avg RTT 20.85 ms |

---

**Note:** Use the official OAI v2.4.0 release for this lab. Do not use the develop branch or a modified OAI source tree. The CN5G images should be the versions verified by the instructor before class.