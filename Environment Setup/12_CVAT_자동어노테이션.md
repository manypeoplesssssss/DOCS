# CVAT YOLO26 Automatic Annotation

## 1. 사전 조건

Nuclio:

```bash
nuctl get functions
```

YOLO26:

```text
ready
1/1
```

CVAT Models:

```text
YOLO26

white_paint
scratch
```

가 확인되어야 한다.

---

# 2. Task 이동

CVAT:

```text
Tasks
→ 원하는 Task
```

---

# 3. Automatic Annotation

```text
Actions
→ Automatic annotation
```

---

# 4. Model

선택:

```text
YOLO26
```

---

# 5. Label Mapping

다음처럼 연결한다.

```text
Model               Task

white_paint   →      white_paint
scratch       →      scratch
```

---

# 6. Threshold

```text
0.25
```

초기 자동 Annotation 테스트 값으로 사용하였다.

---

# 7. ROI

전체 이미지를 대상으로 Detection하려면 Region of Interest는 비워둔다.

```text
ROI = Empty
```

---

# 8. 기존 Annotation 보호

기존에 사람이 만든 Annotation이 존재하면:

```text
Clean previous annotations
```

를:

```text
OFF
```

로 유지한다.

중요:

이 옵션을 켜면 기존 Annotation이 제거될 수 있다.

---

# 9. 최종 설정

```text
Model:
YOLO26

Mapping:
white_paint → white_paint
scratch → scratch

Threshold:
0.25

ROI:
Empty

Clean previous annotations:
OFF
```

---

# 10. 실행

```text
Annotate
```

클릭.

---

# 11. 성공 확인

본 프로젝트에서는 다음 메시지가 표시되었다.

```text
Automatic annotation accomplished for the task #3
```

즉 Automatic Annotation 작업이 정상적으로 완료되었다.

---

# 12. 결과 확인

Task:

```text
Job
→ Annotation
```

으로 들어간다.

확인할 것:

```text
white_paint Bounding Box
scratch Bounding Box
```

---

# 13. 권장 검수

자동 Annotation 결과를 그대로 학습 데이터로 확정하지 않고 사람이 검수한다.

확인:

```text
False Positive
False Negative
잘못된 Class
Bounding Box 위치
Bounding Box 크기
누락된 결함
```

---

# 14. 개선 Cycle

```text
YOLO26 자동 Annotation
        ↓
사람 검수
        ↓
잘못된 Box 수정
        ↓
누락된 Box 추가
        ↓
Dataset Export
        ↓
YOLO26 재학습
        ↓
새 best.pt
        ↓
Nuclio 재배포
        ↓
Automatic Annotation
```

이 과정을 반복하면 Annotation 작업량을 점차 줄일 수 있다.