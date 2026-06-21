# 11. RIPv2 → OSPF Filtering

> Prefix-list / Route-map 을 사용한 조건별 재분배

---

## EX2) RIPv2 → OSPF 조건별 재분배

### 📋 요구사항

| 번호 | 조건 |
|------|------|
| 2-1 | 사설 네트워크는 OSPF 환경으로 재분배되지 않아야 함 |
| 2-2 | `150.1.13.0/24`, `13.13.4.0/24`, `13.13.14.0/24` → Metric Type-1, 초기 Metric **100** |
| 2-3 | `199.171.0.0/24 ~ 199.171.7.0/24` → 고정 Metric, **Tag 199** |
| 2-4 | 나머지 네트워크는 기본값 (E2, Metric=20) 으로 재분배 |

---

### 🔧 R1

```bash
ip prefix-list PRIVATE permit 10.0.0.0/8 le 32
ip prefix-list PRIVATE permit 172.16.0.0/12 le 32
ip prefix-list PRIVATE permit 192.168.0.0/16 le 32
!
ip prefix-list METRIC_TYPE_E1 permit 150.1.13.0/24
ip prefix-list METRIC_TYPE_E1 permit 13.13.4.0/24
ip prefix-list METRIC_TYPE_E1 permit 13.13.14.0/24
!
ip prefix-list TAG199 permit 199.171.0.0/21 le 32
!
route-map RIPv2-OSPF_REDI deny 10
 match ip address prefix-list PRIVATE
route-map RIPv2-OSPF_REDI permit 20
 match ip address prefix-list METRIC_TYPE_E1
 set metric-type type-1
 set metric 100
route-map RIPv2-OSPF_REDI permit 30
 match ip address prefix-list TAG199
 set tag 199
route-map RIPv2-OSPF_REDI permit 40
!
router ospf 1
 redistribute rip subnets route-map RIPv2-OSPF_REDI
```

### ✅ 확인 - 사설 네트워크 차단

```bash
R2#sh ip ro ospf | include 172
R2#sh ip ro ospf | include 10.1
O    13.13.10.0 [110/128] via 13.13.9.3, ..., Serial1/0.123
               [110/128] via 13.13.9.1, ..., Serial1/0.123
```

### ✅ 확인 - Metric Type-1, Metric 100

```bash
R2#sh ip ro ospf | include E1
O E1 13.13.4.0  [110/164] via 13.13.9.1, ..., Serial1/0.123
O E1 13.13.14.0 [110/164] via 13.13.9.1, ..., Serial1/0.123
O E1 150.1.13.0 [110/164] via 13.13.9.1, ..., Serial1/0.123
```

### ✅ 확인 - Tag 199 / 기본 재분배

```bash
R2#sh ip ro ospf | include E2
O E2 199.171.11.0/24 [110/20] via 13.13.9.1, ...
O E2 199.171.10.0/24 [110/20] via 13.13.9.1, ...
O E2 199.171.9.0/24  [110/20] via 13.13.9.1, ...
O E2 199.171.1.0/24  [110/20] via 13.13.9.1, ...
O E2 161.11.1.0      [110/20] via 13.13.9.1, ...
O E2 161.12.2.0      [110/20] via 13.13.9.1, ...
O E2 13.13.11.0      [110/20] via 13.13.9.1, ...
O E2 150.100.1.0     [110/20] via 13.13.9.1, ...
```

### ✅ OSPF Database 확인 (Tag)

```bash
R2#sh ip ospf database

  Type-5 AS External Link States
Link ID         ADV Router  Age   Seq#       Checksum Tag
13.13.4.0       13.13.1.1   873   0x80000002 0x00286D 0
13.13.11.0      13.13.1.1   886   0x80000001 0x003D22 0
13.13.14.0      13.13.1.1   873   0x80000002 0x00B9D1 0
150.1.13.0      13.13.1.1   873   0x80000002 0x0059B5 0
199.171.1.0     13.13.1.1   49    0x80000002 0x00CA7C 199
199.171.2.0     13.13.1.1   49    0x80000002 0x00BF86 199
199.171.3.0     13.13.1.1   50    0x80000002 0x00B490 199
199.171.4.0     13.13.1.1   49    0x80000002 0x00A99A 199
199.171.5.0     13.13.1.1   49    0x80000002 0x009EA4 199
199.171.6.0     13.13.1.1   49    0x80000002 0x0093AE 199
199.171.7.0     13.13.1.1   49    0x80000002 0x0088B8 199
```
