# 10. Connected Redistribute Filtering

> Route-map / Prefix-list 를 사용하여 특정 Connected 네트워크만 선택적 재분배

---

## EX1-1) Connected → OSPF (기본)

> R2는 Fa0/1의 `13.13.12.0/24`를 OSPF 환경으로 재분배 (Metric 고정)

### 🔧 R2

```bash
router ospf 1
 redistribute connected subnets
```

### ✅ 확인

```bash
R1#sh ip ro ospf
O    13.13.2.0  [110/65] via 13.13.9.2, ..., Serial1/0.123
O    13.13.3.0  [110/65] via 13.13.10.3, ..., Serial1/1
                [110/65] via 13.13.9.3, ..., Serial1/0.123
O E2 13.13.12.0 [110/20] via 13.13.9.2, ..., Serial1/0.123
```

---

## EX1-2) 특정 Loopback 제외 (3가지 방법)

### 📋 요구사항

- R2에 Loopback 100 (`100.10.1.0/24`), Loopback 200 (`200.20.2.0/24`) 생성
- 두 네트워크는 **R2 Connected로만 확인**되어야 하며 R1, R3로 전달되지 않아야 함

---

### 방법 1) match interface (permit)

```bash
# R2
interface loopback 100
 ip address 100.10.1.2 255.255.255.0
interface loopback 200
 ip address 200.20.2.2 255.255.255.0
!
route-map OSPF_CONN_REDI permit 10
 match interface fa0/1
!
router ospf 1
 redistribute connected route-map OSPF_CONN_REDI subnets
```

### 방법 2) match interface (deny)

```bash
# R2
interface loopback 100
 ip address 100.10.1.2 255.255.255.0
interface loopback 200
 ip address 200.20.2.2 255.255.255.0
!
route-map OSPF_CONN_REDI deny 10
 match interface loopback 100 loopback 200
route-map OSPF_CONN_REDI permit 20
!
router ospf 1
 redistribute connected route-map OSPF_CONN_REDI subnets
```

### 방법 3) prefix-list 활용

```bash
# R2
interface loopback 100
 ip address 100.10.1.2 255.255.255.0
interface loopback 200
 ip address 200.20.2.2 255.255.255.0
!
ip prefix-list FA0/1 permit 13.13.12.0/24
!
route-map OSPF_CONN_REDI permit 10
 match ip address prefix-list FA0/1
!
router ospf 1
 redistribute connected route-map OSPF_CONN_REDI subnets
```

### ✅ 확인

```bash
R3#sh ip route ospf
O    13.13.1.0  [110/65] via 13.13.10.1, ...
                [110/65] via 13.13.9.1, ...
O    13.13.2.0  [110/65] via 13.13.9.2, ...
O E2 13.13.12.0 [110/20] via 13.13.9.2, ...
```

---

## EX1-3) R4 Loopback 선택적 재분배

### 📋 요구사항

- R4 Loopback 161 ~ 164 생성
  - 161: `161.11.1.0/24`
  - 162: `161.12.2.0/24`
  - 163: `161.13.3.0/24`
  - 164: `161.14.4.0/24`
- **Loopback 163, 164는 R1으로 라우팅되지 않아야 함**

### 🔧 R4

```bash
interface loopback 161
 ip address 161.11.1.3 255.255.255.0
interface loopback 162
 ip address 161.12.2.3 255.255.255.0
interface loopback 163
 ip address 161.13.3.3 255.255.255.0
interface loopback 164
 ip address 161.14.4.3 255.255.255.0
!
route-map RIP_REDI_CONN permit 10
 match interface loopback 161 loopback 162
!
router rip
 redistribute connected metric 2 route-map RIP_REDI_CONN
```

### ✅ 확인

```bash
R4# clear ip route *
R1# clear ip route *

R1#sh ip ro rip | include 161
     161.11.0.0/24 is subnetted, 1 subnets
R    161.11.1.0 [120/2] via 150.1.13.4, ..., FastEthernet0/0
     161.12.0.0/24 is subnetted, 1 subnets
R    161.12.2.0 [120/2] via 150.1.13.4, ..., FastEthernet0/0
```
