# 📱 QR Tool - 기술 문서

이 문서는 QR 코드 생성 및 스캔 도구의 기술적 구현 세부사항을 설명합니다. QR 코드의 구조와 인코딩 원리, 카메라 API를 통한 스캔, URL 안전 검사 알고리즘 등 핵심 기술들을 다룹니다.

---

## 📚 목차

1. [QR 코드 기초](#1-qr-코드-기초)
2. [QR 코드 생성](#2-qr-코드-생성)
3. [데이터 인코딩 형식](#3-데이터-인코딩-형식)
4. [QR 코드 스캔](#4-qr-코드-스캔)
5. [URL 안전 검사](#5-url-안전-검사)
6. [히스토리 관리](#6-히스토리-관리)
7. [성능 최적화](#7-성능-최적화)
8. [보안 고려사항](#8-보안-고려사항)

---

## 1. QR 코드 기초

### 1.1 QR 코드란?

**QR 코드 (Quick Response Code)**는 1994년 일본 덴소 웨이브(Denso Wave)가 개발한 2차원 바코드입니다.

```
QR 코드 구조:
┌─────────────────────────────────┐
│ ■■■■■■■ □□□□□ ■■■■■■■ │ ← 위치 탐지 패턴
│ ■ □□□ ■ □□□□□ ■ □□□ ■ │
│ ■ ■■■ ■ □□□□□ ■ ■■■ ■ │
│ ■ ■■■ ■ □□□□□ ■ ■■■ ■ │
│ ■ ■■■ ■ □□□□□ ■ ■■■ ■ │
│ ■ □□□ ■ □□□□□ ■ □□□ ■ │
│ ■■■■■■■ □■□■□ ■■■■■■■ │ ← 타이밍 패턴
│ □□□□□□□ □□□□□ □□□□□□□ │
│ □■□□□□□ □□□□□ □□□□□□□ │ ← 데이터 영역
│ ...                           │
│ □□□□□□□ ■■■■■             │
│ ■■■■■■■ □□□□□             │ ← 위치 탐지 패턴
└─────────────────────────────────┘
```

### 1.2 QR 코드 버전

QR 코드는 버전 1부터 40까지 있으며, 버전이 높을수록 더 많은 데이터를 저장할 수 있습니다.

| 버전 | 모듈 수 | 최대 문자 (L) |
|------|---------|--------------|
| 1 | 21×21 | 25 |
| 5 | 37×37 | 154 |
| 10 | 57×57 | 395 |
| 20 | 97×97 | 1,249 |
| 40 | 177×177 | 4,296 |

### 1.3 오류 정정 레벨

| 레벨 | 복구 능력 | 용도 |
|------|----------|------|
| L (Low) | ~7% | 일반 사용 |
| M (Medium) | ~15% | 권장 |
| Q (Quartile) | ~25% | 로고 삽입 |
| H (High) | ~30% | 극한 환경 |

### 1.4 인코딩 모드

```javascript
const ENCODING_MODES = {
    NUMERIC: {
        pattern: /^[0-9]+$/,
        bitsPerChar: 3.33,  // 10비트당 3자리
        description: '숫자만 (0-9)'
    },
    ALPHANUMERIC: {
        pattern: /^[0-9A-Z $%*+\-./:]+$/,
        bitsPerChar: 5.5,   // 11비트당 2자
        description: '숫자, 대문자, 일부 특수문자'
    },
    BYTE: {
        pattern: /.*/,
        bitsPerChar: 8,     // 문자당 8비트
        description: 'UTF-8 인코딩'
    },
    KANJI: {
        bitsPerChar: 13,    // 한자당 13비트
        description: '일본어 한자'
    }
};
```

---

## 2. QR 코드 생성

### 2.1 qrcode.js 라이브러리

```html
<script src="https://cdn.jsdelivr.net/npm/qrcode@1.5.3/build/qrcode.min.js"></script>
```

### 2.2 Canvas 렌더링

```javascript
/**
 * QR 코드를 Canvas에 렌더링
 */
async function generateQRToCanvas(data, options) {
    const canvas = document.createElement('canvas');
    
    await QRCode.toCanvas(canvas, data, {
        width: options.size || 256,
        margin: options.margin || 2,
        color: {
            dark: options.foreground || '#000000',
            light: options.background || '#ffffff'
        },
        errorCorrectionLevel: options.errorCorrection || 'M'
    });
    
    return canvas;
}

// 사용 예시
const canvas = await generateQRToCanvas('https://example.com', {
    size: 300,
    foreground: '#8b5cf6',
    background: '#ffffff'
});
```

### 2.3 SVG 생성

```javascript
/**
 * QR 코드를 SVG 문자열로 생성
 */
async function generateQRToSVG(data, options) {
    const svgString = await QRCode.toString(data, {
        type: 'svg',
        width: options.size || 256,
        margin: options.margin || 2,
        color: {
            dark: options.foreground || '#000000',
            light: options.background || '#ffffff'
        }
    });
    
    return svgString;
}

// SVG 다운로드
function downloadSVG(svgString, filename) {
    const blob = new Blob([svgString], { type: 'image/svg+xml' });
    const url = URL.createObjectURL(blob);
    
    const link = document.createElement('a');
    link.download = filename;
    link.href = url;
    link.click();
    
    URL.revokeObjectURL(url);
}
```

### 2.4 PNG 다운로드

```javascript
/**
 * Canvas를 PNG로 다운로드
 */
function downloadPNG(canvas, filename) {
    const dataUrl = canvas.toDataURL('image/png');
    
    const link = document.createElement('a');
    link.download = filename;
    link.href = dataUrl;
    link.click();
}
```

---

## 3. 데이터 인코딩 형식

### 3.1 vCard (연락처)

```javascript
/**
 * vCard 3.0 형식 생성
 */
function createVCard(contact) {
    return `BEGIN:VCARD
VERSION:3.0
N:${contact.lastName};${contact.firstName}
FN:${contact.fullName}
TEL;TYPE=CELL:${contact.phone}
EMAIL:${contact.email}
ORG:${contact.organization}
TITLE:${contact.title}
ADR:;;${contact.address}
URL:${contact.website}
NOTE:${contact.note}
END:VCARD`;
}

// 사용 예시
const vcard = createVCard({
    fullName: '홍길동',
    phone: '010-1234-5678',
    email: 'hong@example.com',
    organization: '회사명'
});
```

### 3.2 WiFi 연결 정보

```javascript
/**
 * WiFi QR 코드 형식
 * WIFI:T:<auth>;S:<ssid>;P:<password>;H:<hidden>;;
 */
function createWiFiString(config) {
    const parts = [
        `T:${config.authType}`,      // WPA, WEP, nopass
        `S:${escapeSpecialChars(config.ssid)}`,
        `P:${escapeSpecialChars(config.password)}`
    ];
    
    if (config.hidden) {
        parts.push('H:true');
    }
    
    return `WIFI:${parts.join(';')};;`;
}

function escapeSpecialChars(str) {
    // 특수 문자 이스케이프: \ ; , " :
    return str.replace(/([\\;,":"])/g, '\\$1');
}

// 사용 예시
const wifi = createWiFiString({
    authType: 'WPA',
    ssid: 'MyNetwork',
    password: 'password123',
    hidden: false
});
// 결과: "WIFI:T:WPA;S:MyNetwork;P:password123;;"
```

### 3.3 이메일 (mailto)

```javascript
/**
 * mailto URI 생성
 */
function createMailtoURI(email) {
    const params = new URLSearchParams();
    
    if (email.subject) {
        params.set('subject', email.subject);
    }
    if (email.body) {
        params.set('body', email.body);
    }
    if (email.cc) {
        params.set('cc', email.cc);
    }
    if (email.bcc) {
        params.set('bcc', email.bcc);
    }
    
    const queryString = params.toString();
    return `mailto:${email.to}${queryString ? '?' + queryString : ''}`;
}

// 사용 예시
const mailto = createMailtoURI({
    to: 'contact@example.com',
    subject: '문의사항',
    body: '안녕하세요'
});
// 결과: "mailto:contact@example.com?subject=%EB%AC%B8%EC%9D%98%EC%82%AC%ED%95%AD&body=%EC%95%88%EB%85%95%ED%95%98%EC%84%B8%EC%9A%94"
```

### 3.4 전화번호

```javascript
/**
 * 전화번호 URI
 */
function createTelURI(phoneNumber) {
    // 숫자와 +만 남기기
    const cleaned = phoneNumber.replace(/[^\d+]/g, '');
    return `tel:${cleaned}`;
}

// 사용 예시: "tel:+821012345678"
```

### 3.5 SMS

```javascript
/**
 * SMS URI
 */
function createSMSURI(phone, message) {
    const uri = `sms:${phone}`;
    if (message) {
        // iOS: &body=  /  Android: ?body=
        return `${uri}?body=${encodeURIComponent(message)}`;
    }
    return uri;
}
```

---

## 4. QR 코드 스캔

### 4.1 html5-qrcode 라이브러리

```html
<script src="https://unpkg.com/html5-qrcode@2.3.8/html5-qrcode.min.js"></script>
```

### 4.2 카메라 초기화

```javascript
/**
 * QR 스캐너 초기화 및 시작
 */
async function initScanner(elementId, onSuccess, onError) {
    const html5QrCode = new Html5Qrcode(elementId);
    
    const config = {
        fps: 10,                    // 초당 프레임 수
        qrbox: { 
            width: 250, 
            height: 250 
        },
        aspectRatio: 1.0,           // 정사각형
        disableFlip: false,         // 전면 카메라 미러링
        experimentalFeatures: {
            useBarCodeDetectorIfSupported: true  // 네이티브 API 사용
        }
    };
    
    try {
        await html5QrCode.start(
            { facingMode: "environment" },  // 후면 카메라
            config,
            onSuccess,
            onError
        );
        return html5QrCode;
    } catch (error) {
        console.error('카메라 접근 실패:', error);
        throw error;
    }
}
```

### 4.3 카메라 권한 처리

```javascript
/**
 * 카메라 권한 상태 확인
 */
async function checkCameraPermission() {
    try {
        const result = await navigator.permissions.query({ name: 'camera' });
        return result.state;  // 'granted', 'denied', 'prompt'
    } catch (error) {
        // permissions API 미지원
        return 'unknown';
    }
}

/**
 * 카메라 권한 요청
 */
async function requestCameraPermission() {
    try {
        const stream = await navigator.mediaDevices.getUserMedia({ 
            video: { facingMode: 'environment' } 
        });
        
        // 권한 획득 후 스트림 종료
        stream.getTracks().forEach(track => track.stop());
        return true;
    } catch (error) {
        if (error.name === 'NotAllowedError') {
            console.log('카메라 권한이 거부되었습니다');
        } else if (error.name === 'NotFoundError') {
            console.log('카메라를 찾을 수 없습니다');
        }
        return false;
    }
}
```

### 4.4 스캔 결과 처리

```javascript
/**
 * 스캔 성공 콜백
 */
function onScanSuccess(decodedText, decodedResult) {
    // 진동 피드백
    if (navigator.vibrate) {
        navigator.vibrate(200);
    }
    
    // 데이터 유형 감지
    const contentType = detectContentType(decodedText);
    
    // 결과 표시
    displayResult({
        content: decodedText,
        type: contentType,
        format: decodedResult.result.format.formatName,
        timestamp: new Date()
    });
}

/**
 * 콘텐츠 유형 감지
 */
function detectContentType(text) {
    const patterns = {
        url: /^https?:\/\//i,
        email: /^mailto:/i,
        phone: /^tel:/i,
        sms: /^sms:/i,
        wifi: /^WIFI:/i,
        vcard: /^BEGIN:VCARD/i,
        geo: /^geo:/i
    };
    
    for (const [type, pattern] of Object.entries(patterns)) {
        if (pattern.test(text)) {
            return type;
        }
    }
    
    // URL 패턴 (프로토콜 없는 경우)
    if (/^[\w-]+\.[\w.-]+/.test(text)) {
        return 'url';
    }
    
    return 'text';
}
```

### 4.5 카메라 전환

```javascript
/**
 * 전면/후면 카메라 전환
 */
async function switchCamera(html5QrCode, currentFacingMode) {
    await html5QrCode.stop();
    
    const newFacingMode = currentFacingMode === 'environment' 
        ? 'user' 
        : 'environment';
    
    await html5QrCode.start(
        { facingMode: newFacingMode },
        config,
        onSuccess,
        onError
    );
    
    return newFacingMode;
}
```

---

## 5. URL 안전 검사

### 5.1 검사 알고리즘

```javascript
/**
 * URL 안전성 검사
 */
function checkUrlSafety(urlString) {
    const result = {
        level: 'safe',      // 'safe', 'warning', 'danger'
        score: 100,
        issues: [],
        details: []
    };
    
    try {
        const url = new URL(urlString);
        
        // 1. 프로토콜 검사
        if (url.protocol === 'http:') {
            result.issues.push({
                type: 'protocol',
                severity: 'warning',
                message: 'HTTPS가 아닌 HTTP 사용'
            });
            result.score -= 20;
        }
        
        // 2. 단축 URL 검사
        const shorteners = [
            'bit.ly', 'tinyurl.com', 't.co', 'goo.gl',
            'ow.ly', 'is.gd', 'buff.ly', 'adf.ly'
        ];
        if (shorteners.includes(url.hostname)) {
            result.issues.push({
                type: 'shortener',
                severity: 'warning',
                message: '단축 URL 서비스 사용 - 실제 목적지 확인 불가'
            });
            result.score -= 15;
        }
        
        // 3. IP 주소 직접 사용
        const ipPattern = /^\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}$/;
        if (ipPattern.test(url.hostname)) {
            result.issues.push({
                type: 'ip-address',
                severity: 'warning',
                message: 'IP 주소 직접 사용 - 피싱 가능성'
            });
            result.score -= 25;
        }
        
        // 4. 의심스러운 키워드
        const suspiciousKeywords = [
            'login', 'signin', 'account', 'password',
            'verify', 'secure', 'update', 'confirm',
            'bank', 'paypal', 'wallet'
        ];
        const urlLower = urlString.toLowerCase();
        suspiciousKeywords.forEach(keyword => {
            if (urlLower.includes(keyword)) {
                result.issues.push({
                    type: 'keyword',
                    severity: 'warning',
                    message: `민감한 키워드 "${keyword}" 포함`
                });
                result.score -= 10;
            }
        });
        
        // 5. @ 문자 (피싱 기법)
        if (url.username || urlString.includes('@')) {
            result.issues.push({
                type: 'at-symbol',
                severity: 'danger',
                message: '@ 문자 포함 - 피싱 기법 의심'
            });
            result.score -= 30;
        }
        
        // 6. 위험 TLD
        const dangerousTlds = ['.tk', '.ml', '.ga', '.cf', '.gq', '.xyz'];
        if (dangerousTlds.some(tld => url.hostname.endsWith(tld))) {
            result.issues.push({
                type: 'dangerous-tld',
                severity: 'danger',
                message: '스팸에 자주 사용되는 도메인'
            });
            result.score -= 35;
        }
        
        // 7. URL 길이
        if (urlString.length > 200) {
            result.issues.push({
                type: 'long-url',
                severity: 'warning',
                message: '비정상적으로 긴 URL'
            });
            result.score -= 10;
        }
        
        // 8. 서브도메인 과다
        const subdomains = url.hostname.split('.');
        if (subdomains.length > 4) {
            result.issues.push({
                type: 'many-subdomains',
                severity: 'warning',
                message: '과도한 서브도메인 사용'
            });
            result.score -= 15;
        }
        
    } catch (e) {
        result.level = 'danger';
        result.score = 0;
        result.issues.push({
            type: 'invalid',
            severity: 'danger',
            message: '잘못된 URL 형식'
        });
    }
    
    // 최종 레벨 결정
    if (result.score >= 80) {
        result.level = 'safe';
    } else if (result.score >= 50) {
        result.level = 'warning';
    } else {
        result.level = 'danger';
    }
    
    return result;
}
```

### 5.2 호모그래프 공격 감지

```javascript
/**
 * 호모그래프 공격 감지 (유사 문자 사용)
 * 예: gооgle.com (키릴 문자 'о' 사용)
 */
function detectHomographAttack(domain) {
    // 비 ASCII 문자 포함 확인
    const hasNonAscii = /[^\x00-\x7F]/.test(domain);
    
    if (hasNonAscii) {
        // 퓨니코드 변환
        const punycode = domain.split('.').map(part => {
            try {
                return punycode.toASCII(part);
            } catch {
                return part;
            }
        }).join('.');
        
        return {
            suspicious: true,
            original: domain,
            punycode: punycode,
            message: '유니코드 도메인 - 피싱 가능성'
        };
    }
    
    return { suspicious: false };
}
```

---

## 6. 히스토리 관리

### 6.1 LocalStorage 저장

```javascript
/**
 * 스캔 히스토리 관리
 */
const ScanHistory = {
    KEY: 'qr-scan-history',
    MAX_ITEMS: 50,
    
    getAll() {
        const data = localStorage.getItem(this.KEY);
        return data ? JSON.parse(data) : [];
    },
    
    add(item) {
        const history = this.getAll();
        
        const newItem = {
            id: Date.now(),
            content: item.content,
            type: item.type,
            timestamp: new Date().toISOString(),
            metadata: item.metadata || {}
        };
        
        history.unshift(newItem);
        
        // 최대 개수 제한
        if (history.length > this.MAX_ITEMS) {
            history.pop();
        }
        
        localStorage.setItem(this.KEY, JSON.stringify(history));
        return newItem;
    },
    
    remove(id) {
        const history = this.getAll().filter(item => item.id !== id);
        localStorage.setItem(this.KEY, JSON.stringify(history));
    },
    
    clear() {
        localStorage.removeItem(this.KEY);
    },
    
    search(query) {
        const history = this.getAll();
        const lowerQuery = query.toLowerCase();
        return history.filter(item => 
            item.content.toLowerCase().includes(lowerQuery)
        );
    }
};
```

### 6.2 CSV 내보내기

```javascript
/**
 * 히스토리를 CSV로 내보내기
 */
function exportToCSV(history) {
    // BOM (Byte Order Mark) for Excel UTF-8 compatibility
    const BOM = '\ufeff';
    
    // 헤더
    const headers = ['ID', 'Type', 'Content', 'Timestamp'];
    
    // 데이터 행
    const rows = history.map(item => [
        item.id,
        item.type,
        `"${item.content.replace(/"/g, '""')}"`,  // 따옴표 이스케이프
        item.timestamp
    ]);
    
    // CSV 생성
    const csv = [
        headers.join(','),
        ...rows.map(row => row.join(','))
    ].join('\n');
    
    // 다운로드
    const blob = new Blob([BOM + csv], { type: 'text/csv;charset=utf-8' });
    const url = URL.createObjectURL(blob);
    const filename = `qr-history-${new Date().toISOString().slice(0,10)}.csv`;
    
    downloadFile(url, filename);
    URL.revokeObjectURL(url);
}

/**
 * 히스토리를 JSON으로 내보내기
 */
function exportToJSON(history) {
    const json = JSON.stringify(history, null, 2);
    const blob = new Blob([json], { type: 'application/json' });
    const url = URL.createObjectURL(blob);
    const filename = `qr-history-${new Date().toISOString().slice(0,10)}.json`;
    
    downloadFile(url, filename);
    URL.revokeObjectURL(url);
}
```

---

## 7. 성능 최적화

### 7.1 스캔 성능

```javascript
/**
 * 스캔 최적화 설정
 */
const scannerConfig = {
    fps: 10,                        // 높을수록 빠르지만 CPU 사용량 증가
    qrbox: { width: 250, height: 250 },  // 스캔 영역 제한
    aspectRatio: 1.0,
    
    // 네이티브 바코드 API 사용 (지원 시)
    experimentalFeatures: {
        useBarCodeDetectorIfSupported: true
    },
    
    // 비디오 제약 조건
    videoConstraints: {
        facingMode: 'environment',
        width: { ideal: 1280 },
        height: { ideal: 720 }
    }
};
```

### 7.2 메모리 관리

```javascript
/**
 * 스캐너 정리
 */
async function cleanupScanner(html5QrCode) {
    if (html5QrCode) {
        try {
            await html5QrCode.stop();
        } catch (error) {
            console.log('스캐너가 이미 중지됨');
        }
        
        html5QrCode.clear();
    }
}

// 페이지 언로드 시 정리
window.addEventListener('beforeunload', () => {
    cleanupScanner(html5QrCode);
});
```

### 7.3 QR 생성 캐싱

```javascript
/**
 * QR 코드 캐싱 (동일 데이터 재생성 방지)
 */
const QRCache = {
    cache: new Map(),
    maxSize: 20,
    
    getKey(data, options) {
        return JSON.stringify({ data, options });
    },
    
    get(data, options) {
        const key = this.getKey(data, options);
        return this.cache.get(key);
    },
    
    set(data, options, canvas) {
        const key = this.getKey(data, options);
        
        if (this.cache.size >= this.maxSize) {
            // LRU: 가장 오래된 항목 삭제
            const firstKey = this.cache.keys().next().value;
            this.cache.delete(firstKey);
        }
        
        this.cache.set(key, canvas);
    }
};
```

---

## 8. 보안 고려사항

### 8.1 XSS 방지

```javascript
/**
 * HTML 이스케이프
 */
function escapeHTML(str) {
    const div = document.createElement('div');
    div.textContent = str;
    return div.innerHTML;
}

/**
 * 안전한 결과 표시
 */
function displayScanResult(content) {
    const resultElement = document.getElementById('result');
    
    // textContent 사용으로 XSS 방지
    resultElement.textContent = content;
    
    // 또는 이스케이프 후 innerHTML
    // resultElement.innerHTML = escapeHTML(content);
}
```

### 8.2 URL 열기 전 확인

```javascript
/**
 * 안전한 URL 열기
 */
function openUrl(url) {
    const safety = checkUrlSafety(url);
    
    if (safety.level === 'danger') {
        const confirmed = confirm(
            '⚠️ 이 URL은 위험할 수 있습니다.\n\n' +
            safety.issues.map(i => i.message).join('\n') +
            '\n\n정말 열시겠습니까?'
        );
        
        if (!confirmed) return;
    }
    
    // noopener, noreferrer로 보안 강화
    window.open(url, '_blank', 'noopener,noreferrer');
}
```

### 8.3 데이터 검증

```javascript
/**
 * QR 데이터 검증
 */
function validateQRData(data, type) {
    const maxLength = 4296;  // QR 코드 최대 용량
    
    if (data.length > maxLength) {
        throw new Error('데이터가 너무 깁니다');
    }
    
    switch (type) {
        case 'url':
            try {
                new URL(data);
            } catch {
                throw new Error('유효하지 않은 URL');
            }
            break;
            
        case 'email':
            if (!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(data)) {
                throw new Error('유효하지 않은 이메일');
            }
            break;
            
        case 'phone':
            if (!/^[\d\s\-+()]+$/.test(data)) {
                throw new Error('유효하지 않은 전화번호');
            }
            break;
    }
    
    return true;
}
```

---

## 부록: QR 코드 사양

### A. 문자 용량 표

| 버전 | 오류 정정 | 숫자 | 영숫자 | 바이트 | 한자 |
|------|----------|------|--------|--------|------|
| 1 | L | 41 | 25 | 17 | 10 |
| 5 | L | 154 | 93 | 64 | 39 |
| 10 | L | 395 | 240 | 165 | 101 |
| 20 | L | 1,249 | 758 | 520 | 320 |
| 40 | L | 4,296 | 2,606 | 1,789 | 1,100 |

### B. 모듈 크기 권장

```
인쇄물:
- 스캔 거리 10cm: 최소 2mm 모듈
- 스캔 거리 30cm: 최소 5mm 모듈

화면:
- 일반 스마트폰: 최소 3px 모듈
- 고해상도 디스플레이: 최소 6px 모듈
```

---

## 참고 자료

1. **ISO/IEC 18004** - QR Code 국제 표준
2. **qrcode.js** - https://github.com/soldair/node-qrcode
3. **html5-qrcode** - https://github.com/mebjas/html5-qrcode
4. **RFC 6350** - vCard 형식
5. **RFC 6068** - mailto URI 스키마

---

*이 문서는 QR Tool 프로젝트의 기술적 구현 세부사항을 설명합니다.*

