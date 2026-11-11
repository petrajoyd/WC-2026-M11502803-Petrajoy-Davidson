# OAI v2.4.0 Build Guide

**OAI Version:** v2.4.0  
**Network:** OAI gNB + nrUE + CN5G (RFsimulator)

**VM environment:**
- Recommended: Ubuntu 24.04 LTS (64-bit)
- Alternative: Ubuntu 22.04 LTS
- Other Linux distributions: Permitted, but not officially validated for this course.
- Hardware: x86-64 CPU with AVX2 support.
- Virtualization: Native Linux is preferred. Virtual machines are allowed if sufficient CPU and memory resources are available.

All students must use OpenAirInterface v2.4.0 and successfully complete the pre-lab environment verification.

> **Note:** The teaching assistant will provide installation instructions and troubleshooting support primarily for Ubuntu 24.04 LTS. Students using other distributions are responsible for resolving distribution-specific compatibility issues.

---

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

Verify:

```bash
cd ~/openairinterface5g-v2.4.0/cmake_targets/ran_build/build

test -x nr-softmodem && echo "gNB build OK"
test -x nr-uesoftmodem && echo "nrUE build OK"
```

## Step 4. Download OAI 5G Core

```bash
cp -a ~/openairinterface5g-v2.4.0/doc/tutorial_resources/oai-cn5g \
  ~/oai-cn5g

cd ~/oai-cn5g

docker compose config --quiet
docker compose pull
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

## Pre-Lab Checkpoint

Before Lab 3, confirm that:

- [ ] OAI version is `v2.4.0`.
- [ ] `nr-softmodem` and `nr-uesoftmodem` are successfully built.
- [ ] Docker and Docker Compose work.
- [ ] CN5G containers start successfully.

---

# Part B — Lab 3 Commands (Optional Before Class)

The following commands will be used during Lab 3.

## Terminal 1 — Start CN5G

```bash
cd ~/oai-cn5g

docker compose up -d
docker compose ps
```

## Terminal 2 — Start gNB

```bash
cd ~/openairinterface5g-v2.4.0/cmake_targets/ran_build/build

sudo ./nr-softmodem \
  -O ../../../targets/PROJECTS/GENERIC-NR-5GC/CONF/gnb.sa.band78.fr1.106PRB.usrpb210.conf \
  --gNBs.[0].min_rxtxtime 6 \
  --rfsim
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

## Terminal 4 — Test Connectivity

```bash
ip addr show oaitun_ue1
```

```bash
ping -I oaitun_ue1 -c 5 192.168.70.135
```

## Stop the Experiment

Stop the gNB and nrUE using `Ctrl+C`.

Then:

```bash
cd ~/oai-cn5g
docker compose down
```

---

> **Note:** Use the official OAI v2.4.0 release for this lab. Do not use the develop branch or a modified OAI source tree. The CN5G images should be the versions verified by the instructor before class.
