# VMware-TeamLab

## 💻 구성도 빡!!!

대충 이미지 네트워크 구성도 빡!

고? 사용한 기술 빡!!

## 📘 Index

[👩🏻‍💻 About Team Members](#-about-team-members)

[🤝 Day 01 - Strategic Planning & Conceptual Design](#-day-01---strategic-planning--conceptual-design)

[🌐 Day 02 - Building Core Infrastructure & Management Plane](#-day-02---building-core-infrastructure--management-plane)

[🧑🏻‍💻 Day 03 - Detailing Our Infrastructure - I](#-day-03---detailing-our-infrastructure---i)

[👩🏻‍💻 Day 04 - Detailing Our Infrastructure - II](#-day-04---detailing-our-infrastructure---ii)

[👩🏻‍💻 Day 05 - Finale (All for vSAN)](#-day-05---finale-all-for-vsan)

## 👩🏻‍💻 About Team Members

추가좀 해주실분?????

실습한거 중에 이미지가 많은건 따로 파일 만들어서 링크하면 좋을듯!

## 🤝 Day 01 - Strategic Planning & Conceptual Design

> **2026.03.06 금요일**

### 1. 그래서 우리의 목표는 무엇인가

### 2. 그래서 왜 클러스터를 했지??

### 3. 그렇게 우리는 영원히 ESXi를 설치하려고 시도를 햇따

## 🌐 Day 02 - Building Core Infrastructure & Management Plane

> **2026.03.10 화요일**

### 0. Day 02 구축 목표 및 핵심 전략

Day 02의 핵심은 실제 기업 환경과 동일한 논리적 격리 및 통합 관리 체계의 완성입니다. 오늘 구축하는 인프라는 이후 진행될 모든 가상화 실습의 견고한 기반이 됩니다.

* 멀티 사이트 논리 격리: Seoul과 Jeju라는 두 개의 독립된 Area를 가정하고, 각 지역의 Management Network을 완전히 분리하여 보안성과 실무 적합성을 확보합니다.

* L3 게이트웨이 및 연결성 확보: VyOS 라우터를 선제적으로 구축하여 개인 Laptop, ESXi 호스트 간의 원활한 통신을 구현하고, NAT 설정을 통해 모든 내부 자원의 외부 Internet 접근 경로를 확보합니다.

* FQDN 기반 관리 체계 수립: 지역별 독립 DNS Server를 구축하고 VCSA (vCenter) 레코드를 사전 등록하여, IP가 아닌 도메인 이름 기반의 전문적인 인프라 관리 환경을 조성합니다.

* Nested 가상화 최적화: WS-ESXi의 Port Group을 정밀하게 설계하여 지역별 vSwitch를 물리적으로 분리된 것처럼 모사하고, 각 지역의 호스트가 지정된 전용 통로로만 통신하도록 강제합니다.

* 고가용성 공유 스토리지 구현: TrueNAS 기반의 iSCSI Storage Network를 구축하여, 향후 vMotion 및 HA 실습의 핵심 자원인 공유 Datastore 환경을 완성합니다.

### 1. WS-ESXi 호스트: Seoul Area Management Network 설정 (DCUI)

워크스테이션에 설치된 기초 하이퍼바이저인 WS-ESXi의 첫 번째 Management Port (`vmk0`)를 Seoul Area 대역으로 설정합니다.

* 설정 방식: 물리 서버 콘솔의 DCUI 인터페이스 활용

* 네트워크 상세 정보

    ```Plain Text
    IPv4 Address: 10.30.10.101
    Subnet Mask: 255.255.255.0
    Default Gateway: 10.30.10.1
    Primary DNS: 10.30.10.77 (Seoul DNS 예정)
    VLAN ID: 0 (Untagged)
    ```

### 2. WS-ESXi 호스트: Jeju Area용 vSwitch 및 Management Network 추가 (Host Client)

WS-ESXi 내부에 Jeju Area를 위한 별도의 vSwitch를 만들고, 두 번째 Management IP를 가진 VMkernel Adapter (`vmk1`)를 생성합니다.

* 작업 단계

    * vSwitch 생성: Jeju 트래픽을 분리할 신규 vSwitch (`Jeju`) 생성

    * VMkernel Adapter 추가: `vmk1`을 생성하고 해당 vSwitch에 연결

    * 서비스 활성화: 반드시 Management 트래픽 체크박스를 활성화

* 네트워크 상세 정보

    ```Plain Text
    IPv4 Address: 172.30.10.101
    Subnet Mask: 255.255.255.0
    Default Gateway: 172.30.10.1
    Primary DNS: 172.30.10.77 (Jeju DNS 예정)
    VLAN ID: 0 (Untagged)
    ```

* WS-ESXi에서 Seoul (`vmk0`)과 Jeju (`vmk1`) Management Network을 독립된 인터페이스로 구성한 모습

    ![WS-ESXi Management Network](/Images/day02_01.png)

### 3. Area별 Windows DNS Server 구축 및 VCSA 레코드 등록

WS-ESXi 위에서 구동될 각 Area의 VM (Windows Server 2022)들을 독립된 DNS Server로 설정합니다.

* DNS Server Network Configuration

    | Area | Port Group | VLAN | IP |
    | --- | --- | --- | --- |
    | Seoul | `Seoul-VM` | 0 (Untagged) | 10.30.10.77 |
    | Jeju | `Jeju-VM` | 0 (Untagged) | 172.30.10.77 |

* `seung.fisa` 도메인 하위에 각 Area의 VCSA 호스트명과 IP를 등록하여, 향후 vCenter 설치 시 DNS 이름으로 접근 가능하도록 준비합니다.

    | Area | Hostname (FQDN) | IP |
    | --- | --- | --- |
    | Seoul | `seoul.seung.fisa` | `10.30.10.102` |
    | Jeju | `jeju.seung.fisa` | `172.30.10.102` |

### 4. VyOS 기반 Site-to-Site 라우팅 및 NAT 환경 구축

Seoul과 Jeju 라우터를 vSwitch 기반의 WAN 구간으로 연결합니다. 특히 Internal Network Interface는 Untagged와 Tagged 트래픽을 동시에 처리하는 하이브리드 구조로 설계하여 확장성을 극대화합니다.

#### 4.1 네트워크 구성

* Shared WAN Switch (Transit): ESXi 내부에 `KT-WAN`를 생성하고, 두 라우터의 `eth1`을 여기에 연결합니다. 이 vSwitch는 두 Area를 잇는 전용회선 역할을 하며, 각 라우터의 `eth1`에 직접 IP를 할당하여 통신합니다.

* Hybrid LAN Interface (`eth0`):

    * Untagged (Native): `eth0` 자체에 IP를 할당하여 기본 Management Network로 사용합니다.

    * Tagged (VLAN): `eth0.20`, `eth0.30`, `eth0.40` 등 서브 인터페이스를 생성하여 향후 추가될 구역별 트래픽을 격리합니다.

* Internet Access (NAT): 라우터의 `eth2`를 외부 Internet과 연결하고, Source NAT 설정을 통해 Internal Network VM이 Internet을 사용할 수 있도록 합니다.

#### 4.2 상세 설정 단계

* 설정 모드 진입: `configure`

* WAN 구간 연결

    * 두 라우터의 `eth1` 인터페이스에 동일 대역의 IP를 할당합니다. 이 구간을 통해 서로의 내부 대역으로 가기 위한 라우팅 정보를 교환합니다.

        ```Bash
        # Seoul Router
        set interfaces ethernet eth1 address 192.168.30.129/24
        set interfaces ethernet eth1 description Router-VM

        # Jeju Router
        set interfaces ethernet eth1 address 192.168.30.130/24
        set interfaces ethernet eth1 description Router-VM
        ```


* Hybrid LAN 인터페이스 설정

    * `eth0` 인터페이스에 기본 IP를 할당하여 물리 포트 자체를 활성화합니다.

        ```Bash
        # Seoul Router
        set interfaces ethernet eth0 address 10.30.10.1/24
        set interfaces ethernet eth0 description Seoul-VM
        set system name-server 10.30.10.77

        # Jeju Router
        set interfaces ethernet eth0 address 172.30.10.1/24
        set interfaces ethernet eth0 description Jeju-VM
        set system name-server 172.30.10.77
        ```


    * 동시에 `vif 20`, `vif 30`, `vif 40` 설정을 통해 802.1Q 태깅이 적용된 서브 인터페이스를 생성합니다.

        ```Bash
        # Seoul Router
        set interfaces ethernet eth0 vif 20 address 10.30.20.1/24
        set interfaces ethernet eth0 vif 30 address 10.30.30.1/24
        set interfaces ethernet eth0 vif 40 address 10.30.40.1/24
        set interfaces ethernet eth0 vif 20 description Seoul-Storage
        set interfaces ethernet eth0 vif 30 description Seoul-vMotion
        set interfaces ethernet eth0 vif 40 description Seoul-FT

        # Jeju Router
        set interfaces ethernet eth0 vif 20 address 172.30.20.1/24
        set interfaces ethernet eth0 vif 30 address 172.30.30.1/24
        set interfaces ethernet eth0 vif 40 address 172.30.40.1/24
        set interfaces ethernet eth0 vif 20 description Jeju-Storage
        set interfaces ethernet eth0 vif 30 description Jeju-vMotion
        set interfaces ethernet eth0 vif 40 description Jeju-FT

        ```

* 라우팅 설정

    * Management Network에 대해서 WAN 인터페이스를 통해 라우팅이 가능하도록 설정합니다.

        ```Bash
        # Seoul Router
        set protocols static route 172.30.10.0/24 next-hop 192.168.30.130

        # Jeju Router
        set protocols static route 10.30.10.0/24 next-hop 192.168.30.129
        ```

* NAT 설정

    * 외부 (`eth2`)로 나가는 트래픽에 대해 Masquerade 설정을 적용합니다.

        ```Bash
        # Seoul Router
        set nat source rule 100 outbound-interface eth2
        set nat source rule 100 source address 10.30.10.0/24
        set nat source rule 100 translation address masquerade

        # Jeju Router
        set nat source rule 100 outbound-interface eth2
        set nat source rule 100 source address 172.30.10.0/24
        set nat source rule 100 translation address masquerade
        ```

* 설정 적용: `commit` → `save`

### 5. 실습용 Seoul 및 Jeju Area ESXi 호스트 (Nested) 설치 및 구성

실제 서비스가 운영될 가상 환경의 핵심인 ESXi 호스트를 각 지역별로 배치합니다. 이 과정은 Nested Virtualization (중첩 가상화) 기술을 활용하여, 이미 구축된 메인 호스트 (WS-ESXi) 위에 Seoul과 Jeju 지역을 대표하는 총 6대의 가상 ESXi 호스트를 올리는 작업입니다.

#### 5.1 ESXi 호스트 배치 및 네트워크 설정

| Area | ESXi Host | Port Group | VLAN | IP | Gateway | DNS Server |
| --- | --- | --- | --- | --- | --- | --- |
| Seoul | Seoul-ESXi-01 | `Seoul-VM` | 0 (Untagged) | `10.30.10.10` | `10.30.10.1` | `10.30.10.77` |
| Seoul | Seoul-ESXi-02 | `Seoul-VM` | 0 (Untagged) | `10.30.10.20` | `10.30.10.1` | `10.30.10.77` |
| Seoul | Seoul-ESXi-03 | `Seoul-VM` | 0 (Untagged) | `10.30.10.30` | `10.30.10.1` | `10.30.10.77` |
| Jeju | Jeju-ESXi-01 | `Jeju-VM` | 0 (Untagged) | `172.30.10.10` | `172.30.10.1` | `172.30.10.77` |
| Jeju | Jeju-ESXi-02 | `Jeju-VM` | 0 (Untagged) | `172.30.10.20` | `172.30.10.1` | `172.30.10.77` |
| Jeju | Jeju-ESXi-03 | `Jeju-VM` | 0 (Untagged) | `172.30.10.30` | `172.30.10.1` | `172.30.10.77` |

#### 5.2 Laptop 네트워크 설정

| Area | Owner | IP | Gateway | DNS Server |
| --- | --- | --- | --- | --- |
| Seoul | min | `10.30.10.11` | `10.30.10.1` | `10.30.10.77` |
| Seoul | soon | `10.30.10.21` | `10.30.10.1` | `10.30.10.77` |
| Seoul | dong | `10.30.10.31` | `10.30.10.1` | `10.30.10.77` |
| Jeju | woo | `172.30.10.11` | `172.30.10.1` | `172.30.10.77` |
| Jeju | jee | `172.30.10.21` | `172.30.10.1` | `172.30.10.77` |
| Jeju | jin | `172.30.10.31` | `172.30.10.1` | `172.30.10.77` |

### 6. vCenter Server Appliance (vCSA) 설치 및 SSO 연동

Seoul과 Jeju의 vCenter를 하나로 묶어 관리하는 Enhanced Linked Mode (ELM) 구성을 목표로 합니다.

* vCSA 설치: 각 지역의 ESXi 호스트에 vCenter Server Appliance를 배포합니다.

    * Seoul VCSA: `seoul.seung.fisa` (IP: `10.30.10.102`)

    * Seoul SSO Domain: `seoul.seung.fisa` (새로운 SSO 도메인 생성)

    * Jeju VCSA: `jeju.seung.fisa` (IP: `172.30.10.102`)

    * Jeju SSO Domain: `seoul.seung.fisa` (Seoul과 동일한 SSO 도메인으로 설정하여 ELM 구성)

### 7. Area별 vSwitch 및 Port Group 하이브리드 구성

Physical Switch에서 VLAN 설정을 할 수 없는 환경을 고려하여, 가상화 레이어에서 Untagged (Management)와 Tagged (Storage, vMotion, FT) 트래픽을 동시에 처리하는 하이브리드 네트워크를 구성합니다.

* Seoul Area의 Seoul-Router가 Trunk (VLAN 4095) 모드로 작동 중이며, 중첩 가상화 패킷 전달을 위해 보안 정책이 적용된 모습

    ![vSwitch for Seoul Area](/Images/day02_02.png)

#### 7.1 네트워크 구성 전략

* Management Network (Untagged): 외부 Internet 및 실제 Physical Switch와의 통신을 위해 VLAN ID를 설정하지 않습니다 (VLAN 0). 이를 통해 별도의 Physical Switch 설정 없이도 외부 접근성을 확보합니다.

* Internal Service Networks (Tagged): 각 Area 내부 트래픽 (Storage, vMotion, FT)은 VLAN 20, 30, 40으로 격리하여 보안과 효율성을 높입니다.

* Router Trunk Port: VyOS 라우터가 연결되는 Port Group은 모든 VLAN 태그를 수용할 수 있도록 Trunk 모드 (VLAN 4095)로 설정합니다.

#### 7.2 vSwitch 및 Port Group 상세 설계

| Area | Purpose | Port Group | VLAN |
| --- | --- | --- | --- |
| Seoul | Management | `Seoul-VM` | 0 (Untagged) |
| Seoul | Storage | `Seoul-Storage` | 20 |
| Seoul | vMotion | `Seoul-vMotion` | 30 |
| Seoul | Fault Tolerance | `Seoul-FT` | 40 |
| Seoul | Router | `Seoul-Router` | 4095 (Trunk) |
| Jeju | Management | `Jeju-VM` | 0 (Untagged) |
| Jeju | Storage | `Jeju-Storage` | 20 |
| Jeju | vMotion | `Jeju-vMotion` | 30 |
| Jeju | Fault Tolerance | `Jeju-FT` | 40 |
| Jeju | Router | `Jeju-Router` | 4095 (Trunk) |

#### 7.3 주요 설정 포인트

* 대역폭 확보: 단일 vSwitch에 모든 트래픽을 몰지 않고, 역할별로 분리하여 논리적인 대역폭 간섭을 최소화합니다.

* 하이브리드 라우팅: VyOS 라우터는 eth0로 Untagged Management Network 통신을 수행하고, 동일한 인터페이스의 서브 인터페이스 (eth0.20 등)로 Tagged Internal Network 라우팅을 동시에 처리합니다.

* Nested ESXi 환경 주의사항: WS-ESXi의 해당 Port Group에서는 Nested VM 내부 트래픽 전달을 위해 다음 보안 정책을 활성화해야 합니다.

  * Promiscuous Mode / MAC Address Changes / Forged Transmits → `Accept`

### 8. Nested ESXi 호스트별 전용 vSwitch 및 1:1 매핑 설정

Seoul과 Jeju 각 Area에 배치된 6대의 Nested ESXi 호스트 내부 설정을 진행합니다. 물리적 WS-ESXi에서 제공하는 서비스별 Port Group을 Nested ESXi가 그대로 받아 쓸 수 있도록, 호스트 내부에 용도별 전용 vSwitch를 생성하고 1:1로 매핑합니다.

* 각 Nested ESXi 호스트 내부에서 Management, Storage, vMotion, FT 트래픽을 논리적으로 분리하고 IP 옥텟 규칙을 적용한 모습

    ![Seoul Area VMKernel adapters](/Images/day02_03.png)

* Nested Host 내부에서 특정 트래픽이 전용 물리 어댑터를 통해 흐르도록 1:1 매핑을 완료한 가상 스위치 토폴로지

    ![Seoul Area Virtual switches (Partially)](/Images/day02_04.png)

#### 8.1 네트워크 매핑 및 태깅 원칙

* 1:1 매핑 구조: WS-ESXi에서 분리해둔 Port Group을 Nested ESXi의 각 업링크 (`vmnic`)와 직접 연결합니다.

* No Double Tagging: 이미 상위 레이어 (VyOS 또는 WS-ESXi Port Group)에서 VLAN 태깅이 처리되고 있습니다.

    * Nested ESXi 내부 Port Group에서 또 태그를 붙이면 이중 태깅이 되어 통신이 불가능해집니다.

    * Nested ESXi 내부의 모든 Port Group은 VLAN 0으로 설정하여 태그 없이 패킷을 통과시킵니다.

* L2 태깅 단일화: 태그는 한 번만 붙인다는 원칙에 따라, Nested ESXi는 L2 단계에서 단순히 패킷을 전달하는 역할만 수행합니다.

#### 8.2 vSwitch 및 Port Group 생성

* Nested ESXi 호스트에서 다음과 같이 vSwitch 및 VMkernel Adapter를 구성합니다. 

* 구분 원칙에 따라, Nested ESXi의 각 vSwitch는 WS-ESXi에서 전달된 특정 트래픽 전용 vmnic 하나만을 업링크로 사용해야 합니다. 

* 모든 내부 Port Group의 VLAN은 0으로 설정합니다.

* 호스트 식별을 위해 IP 주소의 마지막 옥텟을 다음과 같이 규칙화하여 설정합니다.

    * Host-01: `XXX.XXX.XXX.10`
    
    * Host-02: `XXX.XXX.XXX.20`

    * Host-03: `XXX.XXX.XXX.30`

    | Area | Purpose | Port Group | IP (Host-01) | Gateway |
    | --- | --- | --- | --- | --- |
    | Seoul | Management | `Seoul-VM` | `10.30.10.10` | `10.30.10.1` |
    | Seoul | Storage | `Seoul-Storage` | `10.30.20.10` | `10.30.20.1` |
    | Seoul | vMotion | `Seoul-vMotion` | `10.30.30.10` | `10.30.30.1` |
    | Seoul | Fault Tolerance | `Seoul-FT` | `10.30.40.10` | `10.30.40.1` |
    | Jeju | Management | `Jeju-VM` | `172.30.10.10` | `172.30.10.1` |
    | Jeju | Storage | `Jeju-Storage` | `172.30.20.10` | `172.30.20.1` |
    | Jeju | vMotion | `Jeju-vMotion` | `172.30.30.10` | `172.30.30.1` |
    | Jeju | Fault Tolerance | `Jeju-FT` | `172.30.40.10` | `172.30.40.1` |

### 9. TrueNAS VM 생성 및 iSCSI Target 설정

각 Area의 Storage Network (VLAN 20)에 연결된 TrueNAS VM을 생성합니다.

#### 9.1 TrueNAS 네트워크

| Area | Port Group | IP |
| --- | --- | --- | --- |
| Seoul | `Seoul-Storage`| 10.30.20.40 |
| Jeju | `Jeju-Storage` | 172.30.20.40 |

#### 9.2 iSCSI Target 구성

* Storage Pool 생성

    * `Storage` → `Pools` 메뉴에서 추가한 가상 디스크를 사용하여 새로운 ZFS Pool을 생성합니다.

    * 이 Pool은 향후 생성될 모든 데이터의 기반이 됩니다.

* zvol 생성

    * 생성한 Pool 내부에 zvol을 생성합니다.

* iSCSI 서비스 활성화 및 공유 설정

    * `Services` 메뉴에서 iSCSI 서비스를 Running 상태로 변경하고,` Start Automatically`를 체크합니다.

    * `Sharing` → `Block Shares (iSCSI)` → `WIZARD`를 실행하여 설정을 마무리합니다.

        * `Target`: `Create New Target` 선택

        * `Extent`: 앞서 생성한 zvol 선택

        * `Protocol 선택 사항`: IP 주소에 모든 인터페이스를 허용하거나, 특정 Storage Network IP만 허용하도록 설정

### 10. ESXi 호스트 iSCSI Initiator 설정 및 공유 Datastore 구성

각 Area의 ESXi 호스트들이 TrueNAS에서 제공하는 대용량 Storage를 인식하도록 설정합니다. vMotion과 HA 구성을 위해서는 모든 호스트가 동일한 Storage를 바라보는 공유 Storage 환경이 필수적입니다.

#### 10.1 iSCSI 어댑터 활성화 및 포트 바인딩

iSCSI 트래픽이 일반 관리망과 섞이지 않고, 전용 Storage Network를 통해서만 흐르도록 강제하는 과정입니다.

* 어댑터 추가: 각 호스트의 `Configure` → `Storage Adapters` 메뉴에서 `Add Software Adapter` → `Add iSCSI Adapter`를 선택하여 Initiator를 생성합니다.

* 포트 바인딩: 생성된 어댑터의 `Network Port Binding` 탭에서 앞서 만들어둔 Storage 전용 Port Group을 선택합니다.

    * Seoul Area: `Seoul-Storage` / Jeju Area: `Jeju-Storage`

#### 10.2 Target Server 등록 및 장치 인식

* `Dynamic Discovery`: TrueNAS의 Storage용 IP 주소를 추가합니다.

    * Seoul Area: `10.30.20.40` / Jeju Area: `172.30.20.40`

* 장치 확인: 설정을 마친 후 어댑터를 Rescan하여 `Devices` 탭에 TrueNAS에서 할당한 LUN이 정상적으로 나타나는지 확인합니다.

#### 10.3 Area별 공유 Datastore 생성

인식된 LUN을 실제 파일 시스템으로 포맷하여 가상 머신이 저장될 공간을 확보합니다. 나머지 호스트는 Rescan 시 자동 인식하도록 합니다.

* 생성 원칙: 데이터 정합성을 위해 지역별로 단 한 대의 호스트에서만 Datastore를 생성합니다.

* 세부 설정

    * 파일 시스템: VMFS 6

    * 이름 예시: DS-Seoul-Shared-01 / DS-Jeju-Shared-01

    * 용량: 할당된 LUN의 전체 용량 사용

## 🧑🏻‍💻 Day 03 - Detailing Our Infrastructure - I

> **2026.03.11 수요일**

### 1. ESXi Account

루트 계정 비활성화도 해보고 그리고 계정도 하나 만들어서 그 계정으로 로그인도 해보고

### 2. Lockdown Mode

| 모드 | 사용자 구분 | DCUI | SSH / Shell | Host Client | vCenter |
| --- | --- | --- | --- | --- | --- |
| 비활성 | 일반 / 예외 | O | O | O | O |
| 노말 잠금 (Normal) | 일반 사용자 | X | X | X | O |
|  | 예외 사용자 | O | O | O | O |
| 엄격 잠금 (Strict) | 일반 사용자 | X | X | X | O |
|  | 예외 사용자 | X (서비스 정지) | O | O | O |

### 3. VM Tag

태그를 붙이면 내부 관리가 쉽다~

그런데 검색이 매우 귀찮은

### 4. VM Template

윈도우는 삽질을 했고

리눅스는 귀찮다

### 5. Content Library

엘리트 집단이라 미리한거

서울이 만들면 제주는 구독하는 방법

### 6. Alarm

아침에 듣기 싫은거

### 7. DRS: Distributed Resource Scheduler

VM이 어느 호스트에서 실행되는게 좋을지 알아서 결정해주는 기능

F1의 그 DRS 아님

### 8. Autostart, Shutdown

ESXi가 호스트가 켜질 때 자동으로 켜지는 Vm 설정

그리고 종료가 될 때 어떻게 할껀지

### 9. Affinity Rule

깐부

## 👩🏻‍💻 Day 04 - Detailing Our Infrastructure - II

> **2026.03.12 목요일**

### 1. Resource Pool

내일 하자

### 2. HA: High Availability

### 3. FT: Fault Tolerance

### 4. vApp 

### 5. vCenter Backup

### 6. About vSAN

## 👩🏻‍💻 Day 05 - Finale (All for vSAN)