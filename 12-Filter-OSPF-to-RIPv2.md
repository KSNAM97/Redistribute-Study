# 12. OSPF → RIPv2 Filtering (Offset-list)

> Offset-list 를 사용하여 Metric 보정

---

## EX3-1) OSPF → RIPv2 Metric 통일

### 📋 요구사항

- R4에서 OSPF 환경에서 재분배된 모든 네트워크 정보의 **Metric = 4** 로 확인되어야 함

### 💡 access-list 추가 이유

> R1 기준 OSPF 환경 분류:
> - `13.13.1.0`  → connected
> - `13.13.2.0`  → ospf
> - `13.13.3.0`  → ospf
> - `13.13.9.0`  → connected
> - `13.13.10.0` → connected
> - `13.13.12.0` → ospf
>
> Connected 네트워크는 RIPv2로 넘어가면 Metric 1로 보이므로
> **Offset-list로 +3 적용** 하여 통일된 Metric 4로 맞춤

### 🔧 R1

```bash
access-list 1 permit 13.13.1.0 0.0.0.255
access-list 1 permit 13.13.9.0 0.0.0.255
access-list 1 permit 13.13.10.0 0.0.0.255
!
router rip
 redistribute ospf 1 metric 4
 offset-list 1 out 3 fa0/0
```

### ✅ 확인

```bash
R1# clear ip route *

R4#sh ip ro rip
     13.0.0.0/24 is subnetted, 9 subnets
R    13.13.1.0  [120/4] via 150.1.13.1, ..., FastEthernet0/0
R    13.13.2.0  [120/4] via 150.1.13.1, ..., FastEthernet0/0
R    13.13.3.0  [120/4] via 150.1.13.1, ..., FastEthernet0/0
R    13.13.9.0  [120/4] via 150.1.13.1, ..., FastEthernet0/0
R    13.13.10.0 [120/4] via 150.1.13.1, ..., FastEthernet0/0
R    13.13.11.0 [120/1] via 150.1.13.1, ..., FastEthernet0/0
R    13.13.12.0 [120/4] via 150.1.13.1, ..., FastEthernet0/0
```
