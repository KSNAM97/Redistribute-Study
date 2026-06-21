# 02. IGP - OSPF 구성 (Frame-Relay)

> R1, R2, R3 Frame-Relay 구간에 OSPF Area 0 구성

---

## EX1-1) OSPF (R1, R2, R3 Frame-Relay)

### 📋 요구사항

- OSPF Process = 1, Area = 0
- Router-ID = `13.13.X.X` (X = Router 번호)
- R1, R2, R3의 Loopback 0 / Frame-Relay S1/0 네트워크는 Area 0 포함
- 필요한 인터페이스로만 OSPF Packet 송신
- **R2가 DR로 선출되며 BDR은 선출하지 않음**

### 🔧 R2 (Hub / DR)

```bash
router ospf 1
 router-id 13.13.2.2
 passive-interface default
 no passive-interface s1/0.123
 network 13.13.2.0 0.0.0.255 area 0
 network 13.13.9.0 0.0.0.255 area 0
 neighbor 13.13.9.1
 neighbor 13.13.9.3
!
interface s1/0.123
 ip ospf priority 255
```

### 🔧 R1 (Spoke)

```bash
router ospf 1
 router-id 13.13.1.1
 passive-interface default
 no passive-interface s1/0.123
 network 13.13.1.0 0.0.0.255 area 0
 network 13.13.9.0 0.0.0.255 area 0
!
interface s1/0.123
 ip ospf priority 0
```

### 🔧 R3 (Spoke)

```bash
router ospf 1
 router-id 13.13.3.3
 passive-interface default
 no passive-interface s1/0.123
 network 13.13.3.0 0.0.0.255 area 0
 network 13.13.9.0 0.0.0.255 area 0
!
interface s1/0.123
 ip ospf priority 0
```

### ✅ 확인

```bash
R2#sh ip ospf neighbor
Neighbor ID   Pri  State          Dead Time  Address     Interface
13.13.1.1       0  FULL/DROTHER   00:01:32   13.13.9.1   Serial1/0.123
13.13.3.3       0  FULL/DROTHER   00:01:36   13.13.9.3   Serial1/0.123

R1#sh ip ospf neighbor
Neighbor ID   Pri  State          Dead Time  Address     Interface
13.13.2.2     255  FULL/DR        00:01:00   13.13.9.2   Serial1/0.123

R3#sh ip ospf neighbor
Neighbor ID   Pri  State          Dead Time  Address     Interface
13.13.2.2     255  FULL/DR        00:00:42   13.13.9.2   Serial1/0.123
```

---

## EX1-2) OSPF (R1, R3 Frame-Relay 구간)

### 📋 요구사항

- R1, R3 S1/1 네트워크를 Area 0에 포함
- **DR/BDR은 선출하지 않음** (point-to-point)

### 🔧 R1

```bash
router ospf 1
 no passive-interface s1/1
 network 13.13.10.0 0.0.0.255 area 0
!
interface s1/1
 ip ospf network point-to-point
```

### 🔧 R3

```bash
router ospf 1
 no passive-interface s1/1
 network 13.13.10.0 0.0.0.255 area 0
!
interface s1/1
 ip ospf network point-to-point
```

### ✅ 확인

```bash
R1#sh ip ospf neighbor
Neighbor ID   Pri  State         Dead Time  Address      Interface
13.13.3.3       0  FULL/  -      00:00:31   13.13.10.3   Serial1/1
13.13.2.2       0  FULL/DR       00:01:34   13.13.9.2    Serial1/0.123

R3#sh ip ospf neighbor
Neighbor ID   Pri  State         Dead Time  Address      Interface
13.13.1.1       0  FULL/  -      00:00:34   13.13.10.1   Serial1/1
13.13.2.2       0  FULL/DR       00:01:55   13.13.9.2    Serial1/0.123
```

---

## EX1-3) Loopback SubnetMask 보정

> Loopback이 `/32`가 아닌 인터페이스 실제 SubnetMask로 광고되도록 보정

```bash
# R1 ~ R3
interface loopback 0
 ip ospf network point-to-point
```
