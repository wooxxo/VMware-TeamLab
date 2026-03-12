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

### 1. WS-ESXi 호스트에 Management Network 설정하기 

이게 그 실제로는 하나의 Workstation이지만, ESXi를 설치를 하고, 그 위에 여러 또 ESXi와 다른 VM을 설치

실제로는 하나이지만 내부에서는 마치 두개의 지역 (Area)가 있다고 생각을 하고 구성

그래서 지역을 연결하는 라우터도 필요하고, 이것저것 필요하고 DNS NAS도 각각 지역에 하나씩

### 2. Jeju Area용 vSwitch 및 Management Network 설정하기

설치 과정에서 Seoul Area의 Management Network를 설정 했으니, Jeju 용 Switch를 만들어서 Jeju Area의 Management Network도 설정하기

### 3. Windows Server 2022 VM 생성 및 DNS 서버 역할 설치하기

차후에 vCenter Server Appliance 설치할 때, DNS 서버가 필요하기 때문에, Windows Server 2022 VM을 만들어서 DNS 서버 역할을 설치하기

### 4. VyOS VM 생성 및 Seoul Area 및 Jeju Area 간 라우팅 설정하기

위의 작업과 동시에 진행, 라우터 역할을 하는 VyOS VM을 두대 만들고, 각각 Seoul Area와 Jeju Area에 하나씩 배치, 그리고 라우팅 설정을 해서 두 지역 간 통신이 가능하도록 설정하기

강의실 외부 인터넷도 연결해서, NAT를 통해서 Management Network에서 인터넷도 사용할 수 있도록 설정하기

두 라우터를 연결하기 위해서 스위치를 사용, 스위치를 사용한 특별한 이유는 ESXi에서 VM끼리 직접 케이블로 연결하는게 안되기 때문에, 라우터 사이의 연결 (WAN 구간)을 해야하는데, 이때 같은 네트워크면 상관이 없기 때문에 마치 Switch를 Repeater 처럼 사용해서 연결

### 5. 실습용 Seoul-ESXi 및 Jeju-ESXi 호스트 설치 및 구성하기

일단 WS-ESXi 호스트위에 각 지역별로 3개씩 ESXi 호스트를 설치하고 총 6개의 ESXi 호스트를 구성

그리고 위에서 설정한 라우팅이 잘 되는지 확인하고, 인터넷도 되는지 확인하고 조아쓰

### 6. vCenter Server Appliance 설치 및 구성하기

밥 먹기 전에 설치를 눌러두고, Stage I이 잘 되기를 기도하고, Stage II를 진행하면서 설치가 잘 되는지 확인하기

이때 Seoul Area는 SSO 도메인을 생성

이후 Jeju Area는 Seoul Area의 SSO 도메인에 Join 하는 방식으로 설치 진행

트러블슈팅 : 자기애가 넘치는 나머지 도메인을 seungmin.min으로 설정해버린 나 대단해

### 7. 각 지역용 Switch Port Group 설정하기

라우터 연결되는 포트 그룹은 그 트렁크로

다른 지역은 VLAN Tagging으로 근데 그 우리 공유하는 실제 물리 스위치에서 VLAN 설정을 할 수 없기 때문에 Management Network는 VLAN 없이, 그리고 다른 지역은 VLAN Tagging으로 설정하기

### 8. WS-ESXi 호스트와 Seoul-ESXi 연결 (Jeju-ESXi도 동일하게 연결)

그 각각의 ESXi에서 용도에 따른 스위치를 생성 (3개, Management, Storage, vMotion) 그리고 각각의 스위치에 포트 그룹을 만들어서 연결하기

이때 태그를 붙이면 이중 태그가 되서 그 어떠한 스위치에서도 태그는 붙이지 않도록 설정하기

태그는 L2 단계에서 하나의 기기가 붙인다고 생각하기

### 9. TrueNAS VM 생성 및 iSCSI Target 설정하기

이 친구는 지역별 ESXi에서 공유해서 사용할 스토리지 역할을 하고

그러니까 iSCSI Target 역할을 하도록 구성

### 10. ESXi 호스트에서 iSCSI Initiator 설정하기

그리고 ESXi 호스트에서 iSCSI Initiator 설정을 해서 TrueNAS의 iSCSI Target에 연결하기

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