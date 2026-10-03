# YOLOv7 CVAT Serverless 배포

## 1. 목적

Custom YOLO26을 만들기 전에 CVAT Serverless Automatic Annotation이 정상적으로 작동하는지 YOLOv7 Function으로 확인한다.

---

# 2. CVAT Repository

```bash
cd /mnt/c/Users/82107/cvat
```

---

# 3. Nuclio 확인

```bash
nuctl version
```

Project:

```bash
nuctl get projects
```

정상:

```text
cvat
```

---

# 4. YOLOv7 Function

본 환경에서 사용한 Function:

```text
onnx-wongkinyiu-yolov7
```

---

# 5. Function 확인

```bash
nuctl get functions
```

정상 상태:

```text
onnx-wongkinyiu-yolov7
cvat
ready
1/1
4842
```

---

# 6. 상태 의미

```text
ready
```

Function이 실행 가능한 상태이다.

```text
1/1
```

필요한 Replica 1개가 정상 실행되고 있다는 의미이다.

---

# 7. CVAT 확인

CVAT 상단:

```text
Models
```

에서 YOLOv7이 표시되는지 확인한다.

---

# 8. Automatic Annotation

Task에서:

```text
Actions
→ Automatic annotation
```

YOLOv7을 선택할 수 있다면 CVAT ↔ Nuclio 연결이 정상적으로 이루어진 것이다.

---

# 9. YOLO26과 관계

YOLOv7 Function이 정상 동작하면 다음 구조가 이미 준비된 것이다.

```text
CVAT
 ↓
Nuclio
 ↓
Serverless Function
 ↓
Detection
 ↓
CVAT Annotation
```

따라서 이후 Custom YOLO26도 같은 구조로 추가할 수 있다.

---

# 10. 최종 확인

```bash
nuctl get functions
```

최종 환경에서는:

```text
onnx-wongkinyiu-yolov7    ready    1/1
ultralytics-yolo26        ready    1/1
```

두 Function이 동시에 정상 실행되었다.