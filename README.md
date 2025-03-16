# duukrchive
99duuk Archive

---

---

### Python Tagger 기능 상세 정리
#### 1. `main.py`
- **목적**: 시스템 시작점, 컴포넌트 초기화 및 실행 흐름 제어.
- **기능**:
  - `DirectoryWatcher`로 입력 폴더 감시.
  - `process_image`로 새 이미지 처리 (분석 → 이동 → Kafka 발행).
  - 전역 객체 관리 (`tagger`, `file_manager`, `kafka_producer`).
- **동작**: Python이 `~/Desktop/duukrchive`를 감시하며 Kafka로 데이터 전달 시작.

#### 2. `TagProducer` (`interface/kafka_producer.py`)
- **목적**: Kafka로 태그 데이터 발행.
- **기능**:
  - JSON 메시지 생성 (예: `{"file_path": "...", "primary_tag": "vehicle", "tags": ["white", "bicycle"], "timestamp": "..."}`).
  - `image_tags` 토픽으로 발행.
- **동작**: Python에서 분석된 태그를 Kafka로 전송, Kafka Connect가 이를 받아 ES로 전달.

#### 3. `FileManager` (`core/file_manager.py`)
- **목적**: 태그 기반 파일 이동.
- **기능**:
  - `primary_tag`로 디렉터리 생성 (예: `~/Desktop/duukrchive/vehicle`).
  - 파일 이동 및 새 경로 반환.
- **동작**: Python이 대분류 폴더로 파일을 정리.

#### 4. `ModelLoader` (`models/model_loader.py`)
- **목적**: YOLO 모델 로드 및 디바이스 관리.
- **기능**:
  - MPS(Apple Silicon), CPU 등 적합한 디바이스 선택.
  - YOLOv8 모델 로드.
- **동작**: Python에서 모델을 준비해 `ImageTagger`에 제공.

#### 5. `ImageTagger` (`core/tagger.py`)
- **목적**: 이미지 분석 및 태그 생성.
- **기능**:
  - YOLO로 객체 탐지 (예: "human", "bicycle").
  - 신뢰도(confidence) 기반 대분류(`primary_tag`) 선택.
  - 색상 분석으로 추가 태그 생성 (예: "white").
  - `primary_tag`와 `tags` 반환.
- **동작**: Python이 이미지를 분석해 태그를 생성, 이후 파일 이동과 Kafka 발행에 사용.

---

### 전체 흐름에서 동작 방식
1. **Python (YOLO)**:
   - 사용자가 `~/Desktop/duukrchive`에 이미지를 넣으면 `DirectoryWatcher`가 감지.
   - `ImageTagger`가 YOLO로 분석 → `primary_tag`와 `tags` 생성.
   - `FileManager`가 `primary_tag` 폴더로 파일 이동.
   - `TagProducer`가 Kafka `image_tags` 토픽에 발행.
2. **Kafka**:
   - Python에서 발행된 메시지를 `image_tags` 토픽에 저장.
3. **Kafka Connect**:
   - `es-sink` 커넥터가 `image_tags` 토픽을 소비.
   - ES의 `image_tags` 인덱스에 데이터 저장.
4. **Elasticsearch**:
   - 태그와 경로를 저장, 검색 쿼리로 결과 제공.

---

이렇게 정리하면 네 시스템의 현재 모습과 동작이 명확해질 거야! 추가로 보완하고 싶거나 다른 질문 있으면 언제든 말해줘. 잘했어! 😊
