# 09. OSPF → EIGRP Redistribute

> R3에서 OSPF 정보를 EIGRP 환경으로 재분배

---

## EX) OSPF → EIGRP

### 📋 요구사항

- EIGRP는 5가지 Metric 필수
  - **Bandwidth / Delay / Reliability / Load / MTU**
- OSPF에 포함된 Connected 네트워크도 함께 재분배됨

### 🔧 R3

```bash
router eigrp 100
 redistribute ospf 1 metric 100000 10 255 1 1500
```

> 참고: R3 Fa0/0 인터페이스 정보
> ```
> Internet address is 150.3.13.3/24
> MTU 1500 bytes, BW 100000 Kbit, DLY 100 usec,
> reliability 255/255, txload 1/255, rxload 1/255
> ```

### ✅ 확인

```bash
R5# show ip route eigrp
D EX 199.171.11.0/24 [170/284160] via 150.3.13.3, ...
D EX 192.168.8.0/24  [170/284160] via 150.3.13.3, ...
     172.16.0.0/24 is subnetted, 4 subnets
D EX 172.16.4.0     [170/284160] via 150.3.13.3, ...
     10.0.0.0/24 is subnetted, 4 subnets
D EX 10.1.0.0       [170/284160] via 150.3.13.3, ...
     13.0.0.0/24 is subnetted, 12 subnets
D EX 13.13.1.0      [170/284160] via 150.3.13.3, ...
D EX 13.13.2.0      [170/284160] via 150.3.13.3, ...
D EX 13.13.3.0      [170/284160] via 150.3.13.3, ...
D EX 13.13.4.0      [170/284160] via 150.3.13.3, ...
D EX 13.13.11.0     [170/284160] via 150.3.13.3, ...
D EX 13.13.12.0     [170/284160] via 150.3.13.3, ...
D    13.13.13.0     [90/307200]  via 150.3.13.3, ...
     150.1.0.0/24 is subnetted, 1 subnets
D EX 150.1.13.0     [170/284160] via 150.3.13.3, ...
```

---

## 🗑️ 재분배 설정 삭제

```bash
# R1
router ospf 1
 no redistribute rip
router rip
 no redistribute ospf 1

# R2
router ospf 1
 no redistribute connected

# R3
router eigrp 100
 no redistribute ospf 1
router ospf 1
 no redistribute eigrp 100
```
