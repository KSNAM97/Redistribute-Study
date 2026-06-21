# 03. IGP - RIPv2 (R1 ↔ R4)

> R1, R4 구간에서 RIPv2를 사용한 라우팅 업데이트

---

## EX2) RIPv2 구성

### 📋 요구사항

- 자동 요약 비활성화
- 필요한 인터페이스로만 RIPv2 Packet 송신
- R1의 Fa0/1 네트워크 포함
- R4는 RIPv2 설정 완료된 상태

### 🔧 R1

```bash
router rip
 version 2
 no auto-summary
 passive-interface default
 no passive-interface fa0/0
 network 13.0.0.0
 network 150.1.0.0
```

### ✅ 확인

```bash
R1# show ip route rip
R   13.13.4.0/24  [120/1] via 150.1.13.4, ...
R   13.13.14.0/24 [120/1] via 150.1.13.4, ...
```
