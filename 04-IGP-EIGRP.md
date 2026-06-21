# 04. IGP - EIGRP (R3 ↔ R5)

> R3, R5 구간에서 EIGRP AS 100 라우팅 업데이트

---

## EX3) EIGRP 구성

### 📋 요구사항

- AS = 100, 자동 요약 비활성화
- 필요한 인터페이스로만 EIGRP Packet 송신
- R3의 Fa0/1 네트워크 포함
- R5는 EIGRP 설정 완료된 상태

### 🔧 R3

```bash
router eigrp 100
 no auto-summary
 passive-interface default
 no passive-interface fa0/0
 network 13.13.13.0 0.0.0.255
 network 150.3.13.0 0.0.0.255
```

### ✅ 확인

```bash
R3# show ip route eigrp
D   13.13.5.0/24  [90/...] via 150.3.13.5, ...
D   13.13.15.0/24 [90/...] via 150.3.13.5, ...
```
