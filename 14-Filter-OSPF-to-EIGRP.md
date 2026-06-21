# 14. OSPF → EIGRP Filtering (Tag)

> Prefix-list 와 Tag를 사용한 선택적 재분배

---

## EX5-1) OSPF → EIGRP 조건별 재분배

### 📋 요구사항

| 조건 | 처리 |
|------|------|
| `199.171.0.0/24 ~ 199.171.7.0/24` | **Tag 100** 부여하여 재분배 |
| 그 외 `199.171.x.0/24` | 재분배 거부 |
| 그 외 모든 네트워크 | EIGRP 환경으로 정상 재분배 |

---

### 🔧 R3

```bash
ip prefix-list TAG100 permit 199.171.0.0/21 le 32
ip prefix-list DENY_LOOP199 permit 199.171.0.0/16 le 32
!
route-map OSPF-EIGRP_REDI permit 10
 match ip address prefix-list TAG100
 set tag 100
route-map OSPF-EIGRP_REDI deny 20
 match ip address prefix-list DENY_LOOP199
route-map OSPF-EIGRP_REDI permit 30
!
router eigrp 100
 redistribute ospf 1 metric 100000 10 255 1 1500 route-map OSPF-EIGRP_REDI
```

### ✅ 확인 - Tag 100 적용 네트워크

```bash
R5#sh ip route eigrp | include 199
D EX 199.171.3.0/24 [170/30720] via 150.3.13.3, ..., FastEthernet0/0
D EX 199.171.2.0/24 [170/30720] via 150.3.13.3, ...
D EX 199.171.1.0/24 [170/30720] via 150.3.13.3, ...
D EX 199.171.7.0/24 [170/30720] via 150.3.13.3, ...
D EX 199.171.6.0/24 [170/30720] via 150.3.13.3, ...
D EX 199.171.5.0/24 [170/30720] via 150.3.13.3, ...
D EX 199.171.4.0/24 [170/30720] via 150.3.13.3, ...

R5#sh ip eigrp topology | include tag is 100
P 199.171.4.0/24, 1 successors, FD is 30720, tag is 100
P 199.171.5.0/24, 1 successors, FD is 30720, tag is 100
P 199.171.6.0/24, 1 successors, FD is 30720, tag is 100
P 199.171.7.0/24, 1 successors, FD is 30720, tag is 100
P 199.171.1.0/24, 1 successors, FD is 30720, tag is 100
P 199.171.2.0/24, 1 successors, FD is 30720, tag is 100
P 199.171.3.0/24, 1 successors, FD is 30720, tag is 100
```

### ✅ 확인 - 그 외 네트워크 (199.171.x 제외)

```bash
R5#sh ip route eigrp | exclude 199
     161.11.0.0/24 is subnetted, 1 subnets
D EX 161.11.1.0  [170/30720] via 150.3.13.3, ..., FastEthernet0/0
     161.12.0.0/24 is subnetted, 1 subnets
D EX 161.12.2.0  [170/30720] via 150.3.13.3, ...
     13.0.0.0/24 is subnetted, 12 subnets
D EX 13.13.1.0   [170/30720] via 150.3.13.3, ...
D EX 13.13.2.0   [170/30720] via 150.3.13.3, ...
D EX 13.13.3.0   [170/30720] via 150.3.13.3, ...
D EX 13.13.4.0   [170/30720] via 150.3.13.3, ...
D EX 13.13.9.0   [170/30720] via 150.3.13.3, ...
D EX 13.13.10.0  [170/30720] via 150.3.13.3, ...
D EX 13.13.11.0  [170/30720] via 150.3.13.3, ...
D EX 13.13.12.0  [170/30720] via 150.3.13.3, ...
D    13.13.13.0  [90/30720]  via 150.3.13.3, ...
D EX 13.13.14.0  [170/30720] via 150.3.13.3, ...
     150.100.0.0/24 is subnetted, 1 subnets
D EX 150.100.1.0 [170/30720] via 150.3.13.3, ...
     150.1.0.0/24 is subnetted, 1 subnets
D EX 150.1.13.0  [170/30720] via 150.3.13.3, ...
```
