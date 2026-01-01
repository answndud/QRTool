# 📱 QR Tool - 생성 & 스캔

QR 코드를 생성하고 스캔하는 올인원 웹 도구입니다. 다양한 데이터 유형을 지원하며, URL 안전 검사와 스캔 히스토리 관리 기능을 제공합니다.

![QR Code](https://img.shields.io/badge/QR-Generator-8b5cf6)
![Scanner](https://img.shields.io/badge/QR-Scanner-22c55e)
![Client Side](https://img.shields.io/badge/Processing-Client--Side-orange)

## ✨ 주요 기능

### 📝 QR 코드 생성
- **텍스트**: 일반 텍스트 메시지
- **URL**: 웹사이트 링크
- **연락처**: vCard 형식의 연락처 정보
- **WiFi**: 네트워크 연결 정보
- **이메일**: mailto 링크

### 🎨 커스터마이징
- QR 코드 색상 선택
- 배경 색상 선택
- 크기 조절 (128px ~ 512px)

### 📥 다운로드
- PNG 형식 다운로드
- SVG 형식 다운로드 (벡터)

### 📷 QR 코드 스캔
- 카메라를 통한 실시간 스캔
- 자동 데이터 유형 감지
- 진동 피드백 (지원 기기)

### 🔒 URL 안전 검사
- HTTPS 프로토콜 확인
- 단축 URL 감지
- IP 주소 직접 사용 감지
- 의심스러운 키워드 확인
- 위험한 TLD 경고

### 📜 스캔 히스토리
- 스캔 결과 자동 저장
- 최대 50개 기록 보관
- CSV 파일로 내보내기
- 개별/전체 삭제

## 🚀 사용 방법

### QR 코드 생성
1. "생성" 탭을 선택합니다
2. 데이터 유형을 선택합니다 (텍스트, URL, 연락처 등)
3. 내용을 입력합니다
4. 원하는 색상과 크기를 설정합니다
5. "QR 코드 생성" 버튼을 클릭합니다
6. PNG 또는 SVG로 다운로드합니다

### QR 코드 스캔
1. "스캔" 탭을 선택합니다
2. "스캔 시작" 버튼을 클릭합니다
3. 카메라 권한을 허용합니다
4. QR 코드를 카메라에 비춥니다
5. 스캔 결과를 확인합니다
6. URL인 경우 안전 검사 결과를 확인합니다

## 🔐 URL 안전 검사 기준

| 상태 | 설명 |
|------|------|
| ✅ 안전 | HTTPS 사용, 의심 패턴 없음 |
| ⚠️ 주의 | HTTP 사용, 단축 URL, 긴 URL 등 |
| 🚨 위험 | 스팸 TLD, 피싱 의심 패턴 |

### 검사 항목
- 프로토콜 (HTTP vs HTTPS)
- 단축 URL 서비스 (bit.ly, tinyurl 등)
- IP 주소 직접 사용
- 민감한 키워드 (login, password, bank 등)
- @ 문자 포함 (피싱 기법)
- URL 길이 (200자 초과)
- 위험 TLD (.tk, .ml, .ga 등)

## 📱 지원 데이터 형식

### 연락처 (vCard)
```
BEGIN:VCARD
VERSION:3.0
N:홍길동
FN:홍길동
TEL:010-1234-5678
EMAIL:hong@example.com
ORG:회사명
END:VCARD
```

### WiFi
```
WIFI:T:WPA;S:네트워크명;P:비밀번호;;
```

### 이메일
```
mailto:email@example.com?subject=제목&body=본문
```

## 🛠️ 기술 스택

| 분류 | 기술 |
|------|------|
| **프론트엔드** | HTML5, CSS3, Vanilla JavaScript |
| **QR 생성** | qrcode.js |
| **QR 스캔** | html5-qrcode |
| **폰트** | Inter, JetBrains Mono |
| **배포** | GitHub Pages |

## 📁 프로젝트 구조

```
QRTool/
├── index.html      # 메인 애플리케이션
├── README.md       # 프로젝트 문서
└── TECHNICAL.md    # 기술 상세 문서
```

## 🌐 브라우저 호환성

| 브라우저 | 생성 | 스캔 |
|----------|:----:|:----:|
| Chrome | ✅ | ✅ |
| Firefox | ✅ | ✅ |
| Safari | ✅ | ✅ |
| Edge | ✅ | ✅ |

> 📷 스캔 기능은 카메라 권한이 필요합니다

## ⚠️ 주의사항

- 스캔 기능은 HTTPS 환경에서만 작동합니다
- 카메라 권한을 허용해야 스캔이 가능합니다
- 히스토리는 브라우저 LocalStorage에 저장됩니다
- 모든 처리는 클라이언트 측에서 이루어집니다

## 📜 라이선스

MIT License - 자유롭게 사용, 수정, 배포 가능합니다.

## 🔗 관련 링크

- [qrcode.js](https://github.com/soldair/node-qrcode)
- [html5-qrcode](https://github.com/mebjas/html5-qrcode)
- [QR Code - Wikipedia](https://en.wikipedia.org/wiki/QR_code)

---

Made with 📱 for convenience

