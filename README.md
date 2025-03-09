# duukrchive
99duuk Archive

---
## 목표

#### 시스템 개요
```
[사용자] --> [로컬 폴더: ~/Desktop/archive]
                     |
                     v
[Spring Boot] <--> [Python (MobileNetV2 or YOLO)] <--> [Kafka (Optional)]
         |                       |
         v                       v
[Elasticsearch]       [대분류 폴더: ~/Desktop/archive/jeans, boots 등]
         |
         v
[검색 결과] --> [Finder]
```

#### 컴포넌트 설명
1. **로컬 폴더 (`~/Desktop/archive`)**
   - 네가 이미지를 넣는 시작점.
   - 예: `random.jpg`, `screenshot.png` 등.

2. **Spring Boot**
   - 역할: 시스템의 중심 허브.
   - 기능:
     - `WatchService`로 폴더 감지.
     - Python 모델 호출 (또는 Kafka로 통신).
     - 태그 받아서 파일 이동 + ES 저장.
   - 연결:
     ```
     Spring Boot --> Python (이미지 분석 요청)
     Spring Boot <-- Python (태그 반환)
     Spring Boot --> Elasticsearch (태그/경로 저장)
     Spring Boot --> 대분류 폴더 (파일 이동)
     ```

3. **Python (MobileNetV2 or YOLO)**
   - 역할: 이미지 분석으로 태그 생성.
   - MobileNetV2:
     - 다중 레이블 튜닝: `[jeans, boots, man]`.
   - YOLO (대안):
     - 객체 탐지: "jeans", "boots" 등 위치 탐지 후 태그화.
     - 약간 무거움, 나중에 확장 가능.
   - 연결:
     ```
     Python <-- Spring Boot (이미지 경로 받기)
     Python --> Spring Boot (태그 전달)
     Python --> Kafka (옵션, 태그 전달)
     ```

4. **Kafka (Optional, 공부용)**
   - 역할: Spring Boot와 Python 간 비동기 통신.
   - 흐름:
     ```
     Python --> Kafka (태그 발행)
     Spring Boot <-- Kafka (태그 구독)
     ```
   - 특징: 조잡해도 학습 목적이면 OK.

5. **Elasticsearch**
   - 역할: 태그와 파일 경로 저장, 검색 제공.
   - 데이터 예:
     ```
     {
       "path": "~/Desktop/archive/jeans/random.jpg",
       "tags": ["jeans", "boots", "man"]
     }
     ```
   - 연결:
     ```
     Elasticsearch <-- Spring Boot (데이터 저장)
     Elasticsearch --> Spring Boot (검색 결과 반환)
     ```

6. **대분류 폴더**
   - 역할: 태그 기반으로 파일 정리.
   - 예: `~/Desktop/archive/jeans`, `~/Desktop/archive/webtoon`.
   - 흐름:
     ```
     대분류 폴더 <-- Spring Boot (파일 이동)
     ```

7. **Finder**
   - 역할: 검색 결과로 파일 열기.
   - 연결:
     ```
     Finder <-- Spring Boot (open 명령 실행)
     ```

8. 사용자
    - `Electron/Tauri + vue` Desktop App 

#### 데이터 흐름 (시나리오 예시)
```
1. 사용자가 ~/Desktop/archive에 "random.jpg" 넣음
   |
2. Spring Boot가 감지
   |
3. Spring Boot --> Python: "random.jpg" 전달
   |
4. Python (MobileNetV2): 이미지 분석 --> [jeans, boots] 반환
   |        (Kafka 옵션: Python --> Kafka --> Spring Boot)
   v
5. Spring Boot:
   - [jeans]로 ~/Desktop/archive/jeans 폴더 생성 후 파일 이동
   - [jeans, boots]와 경로를 Elasticsearch에 저장
   |
6. 사용자가 ES에서 "boots" 검색
   |
7. Spring Boot --> ES: 쿼리 날림 --> 경로 반환
   |
8. Spring Boot --> Finder: 파일 열기
```

---

### 아키텍처 특징
- **단순함**: 이미지 넣으면 "알잘딱"으로 대분류 + 검색 가능.
- **조잡함 허용**: 모델이 70~80%만 맞춰도 OK, Kafka 없어도 동작.
- **학습 포인트**:
  - MobileNetV2 튜닝 (or YOLO).
  - Kafka 통신 (공부용).
  - ES 토크나이징/검색.
- **확장성**: 나중에 Vue.js 웹 추가, YOLO로 업그레이드 가능.

---

