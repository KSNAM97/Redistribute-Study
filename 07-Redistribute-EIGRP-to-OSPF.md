# 07. EIGRP → OSPF Redistribute

> R3에서 EIGRP 정보를 OSPF 환경으로 재분배

---

## EX6) EIGRP → OSPF

### 📋 요구사항

- OSPF로 재분배된 EIGRP 네트워크는 **Metric 누적** (E1)
- EIGRP에 포함된 Connected 네트워크도 함께 재분배됨

### 🔧 R3

```bash
router ospf 1
 redistribute eigrp 100 metric-type 1 subnets
```

### ✅ 확인

```bash
R2# show ip route ospf | include E1
O E1 4.1.1.0        [110/84] via 13.13.9.3, ...
O E1 128.28.2.0     [110/84] via 13.13.9.3, ...
O E1 128.128.1.0    [110/84] via 13.13.9.3, ...
O E1 198.2.3.0/24   [110/84] via 13.13.9.3, ...
O E1 198.198.1.0/24 [110/84] via 13.13.9.3, ...
O E1 13.13.5.0      [110/84] via 13.13.9.3, ...
O E1 13.13.13.0     [110/84] via 13.13.9.3, ...
O E1 13.13.15.0     [110/84] via 13.13.9.3, ...
O E1 150.3.13.0     [110/84] via 13.13.9.3, ...
```
