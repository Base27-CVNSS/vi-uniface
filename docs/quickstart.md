# Khởi động nhanh

Mục tiêu của trang này là giúp bạn chạy được những pipeline UniFace phổ biến nhất trong vài phút, đồng thời hiểu **đầu vào → model → đầu ra**.

---

## 1. Pipeline hợp nhất với `FaceAnalyzer`

`FaceAnalyzer` là điểm bắt đầu phù hợp nếu bạn muốn detection + alignment + recognition mà không phải tự nối từng module.

```python
import cv2
from uniface import FaceAnalyzer

image = cv2.imread("photo.jpg")
analyzer = FaceAnalyzer()
faces = analyzer.analyze(image)

for face in faces:
    print("BBox:", face.bbox)
    print("Confidence:", face.confidence)
    print("Embedding:", face.embedding.shape if face.embedding is not None else None)
```

!!! info "Trường nào luôn có?"
    Sau pipeline mặc định, các trường cơ bản như `bbox`, `confidence`, `landmarks` và embedding nhận dạng được tạo theo cấu hình analyzer. Các thuộc tính như tuổi, nhóm tuổi, chủng loại nhân khẩu học, cảm xúc, quality hoặc face states chỉ có khi predictor tương ứng được chạy.

Thêm predictor:

```python
from uniface import FaceAnalyzer, FairFace

analyzer = FaceAnalyzer(predictors=[FairFace()])
faces = analyzer.analyze(image)

for face in faces:
    print(face.sex, face.age_group, face.race)
```

---

## 2. Chỉ phát hiện khuôn mặt

```python
import cv2
from uniface.detection import RetinaFace

image = cv2.imread("photo.jpg")
detector = RetinaFace()
faces = detector.detect(image)

for i, face in enumerate(faces, start=1):
    print(f"Khuôn mặt {i}")
    print("  confidence:", face.confidence)
    print("  bbox:", face.bbox)
    print("  landmarks:", face.landmarks.shape)
```

**Bản chất:** detector trả về *vị trí* khuôn mặt. Nó chưa trả lời “đây là ai”.

---

## 3. Vẽ kết quả detection

```python
import cv2
from uniface.detection import RetinaFace
from uniface.draw import draw_detections

image = cv2.imread("photo.jpg")
faces = RetinaFace().detect(image)

draw_detections(image=image, faces=faces, vis_threshold=0.6)
cv2.imwrite("output.jpg", image)
```

---

## 4. So khớp hai khuôn mặt

```python
import cv2
from uniface import compute_similarity
from uniface.detection import RetinaFace
from uniface.recognition import ArcFace

detector = RetinaFace()
recognizer = ArcFace()

image1 = cv2.imread("person1.jpg")
image2 = cv2.imread("person2.jpg")

faces1 = detector.detect(image1)
faces2 = detector.detect(image2)

if faces1 and faces2:
    emb1 = recognizer.get_normalized_embedding(image1, faces1[0].landmarks)
    emb2 = recognizer.get_normalized_embedding(image2, faces2[0].landmarks)

    similarity = compute_similarity(emb1, emb2, normalized=True)
    print("Cosine similarity:", similarity)
```

!!! warning "Không hard-code threshold cho production"
    Ngưỡng cosine phù hợp phụ thuộc model, dataset, camera, chất lượng ảnh và mức FAR/FRR bạn chấp nhận. Con số trong demo chỉ nên dùng để thử nghiệm ban đầu; hệ thống thật cần hiệu chuẩn.

---

## 5. Landmark 106 điểm

```python
import cv2
from uniface.detection import RetinaFace
from uniface.landmark import Landmark106

image = cv2.imread("photo.jpg")
faces = RetinaFace().detect(image)

if faces:
    landmarker = Landmark106()
    landmarks = landmarker.get_landmarks(image, faces[0].bbox)
    print(landmarks.shape)  # (106, 2)
```

Landmark dày phù hợp cho căn chỉnh tinh, đo hình học, biểu cảm và các pipeline chỉnh sửa khuôn mặt.

---

## 6. Face Mesh 468 / 478 điểm

```python
from uniface.landmark import FaceMesh

mesher = FaceMesh()
results = mesher.predict(image, faces)

if results:
    print(results[0].landmarks.shape)  # (468, 3)
    print(results[0].points_2d.shape)  # (468, 2)
    print(results[0].score)
```

Biến thể 478 điểm bổ sung iris. Tọa độ z là độ sâu tương đối của mesh, không nên mặc định xem là khoảng cách metric tuyệt đối theo mm.

---

## 7. Ước lượng hướng nhìn

```python
import cv2
import numpy as np
from uniface.detection import RetinaFace
from uniface.gaze import MobileGaze

image = cv2.imread("photo.jpg")
faces = RetinaFace().detect(image)
gaze = MobileGaze()

for face in faces:
    x1, y1, x2, y2 = map(int, face.bbox[:4])
    crop = image[y1:y2, x1:x2]

    if crop.size:
        result = gaze.estimate(crop)
        print("pitch:", np.degrees(result.pitch))
        print("yaw:", np.degrees(result.yaw))
```

---

## 8. Tư thế đầu 3D

```python
import cv2
from uniface.detection import RetinaFace
from uniface.headpose import HeadPose

image = cv2.imread("photo.jpg")
faces = RetinaFace().detect(image)
head_pose = HeadPose()

for face in faces:
    x1, y1, x2, y2 = map(int, face.bbox[:4])
    crop = image[y1:y2, x1:x2]

    if crop.size:
        result = head_pose.estimate(crop)
        print(result.pitch, result.yaw, result.roll)
```

---

## 9. Face Parsing

```python
import cv2
import numpy as np
from uniface.parsing import BiSeNet

face_image = cv2.imread("face.jpg")
parser = BiSeNet()
mask = parser.parse(face_image)

print("Số nhãn xuất hiện:", len(np.unique(mask)))
```

BiSeNet tạo mask ngữ nghĩa cho các thành phần khuôn mặt, hữu ích cho makeup AR, chỉnh sửa ảnh, compositing và nghiên cứu thị giác máy tính.

---

## 10. Tách nền chân dung

```python
import cv2
import numpy as np
from uniface.matting import MODNet

image = cv2.imread("portrait.jpg")
matte = MODNet().predict(image)

rgba = cv2.cvtColor(image, cv2.COLOR_BGR2BGRA)
rgba[:, :, 3] = (matte * 255).astype(np.uint8)
cv2.imwrite("transparent.png", rgba)
```

---

## 11. Ẩn danh khuôn mặt

```python
import cv2
from uniface.detection import RetinaFace
from uniface.privacy import BlurFace

image = cv2.imread("group_photo.jpg")
faces = RetinaFace().detect(image)

blurrer = BlurFace(method="pixelate")
anonymized = blurrer.anonymize(image, faces)
cv2.imwrite("anonymized.jpg", anonymized)
```

Các kiểu làm mờ có thể gồm pixelate, gaussian, blackout, elliptical hoặc median tùy phiên bản/API.

---

## 12. Chống giả mạo / liveness

```python
import cv2
from uniface.detection import RetinaFace
from uniface.spoofing import MiniFASNet

image = cv2.imread("photo.jpg")
faces = RetinaFace().detect(image)
spoofer = MiniFASNet()

for face in faces:
    result = spoofer.predict(image, face.bbox)
    print("is_real:", result.is_real)
    print("confidence:", result.confidence)
```

!!! danger "Liveness không phải lá chắn tuyệt đối"
    Một anti-spoofing model đơn lẻ không bảo đảm chống mọi replay, print, mask hoặc attack mới. Hệ thống bảo mật nghiêm túc thường kết hợp challenge-response, nhiều tín hiệu và kiểm thử presentation attack.

---

## 13. Chấm điểm chất lượng khuôn mặt

```python
import cv2
from uniface.detection import SCRFD
from uniface.quality import EDifFIQA

image = cv2.imread("photo.jpg")
faces = SCRFD(confidence_threshold=0.3).detect(image)
quality = EDifFIQA()

for face in faces:
    result = quality.predict(image, face.landmarks)
    print("quality:", result.score)
```

Nên dùng quality score như **cổng lọc trước recognition** thay vì nhận dạng mọi ảnh bất kể độ mờ/pose/kích thước.

---

## 14. Webcam thời gian thực

```python
import cv2
from uniface.detection import RetinaFace
from uniface.draw import draw_detections

detector = RetinaFace()
cap = cv2.VideoCapture(0)

while True:
    ok, frame = cap.read()
    if not ok:
        break

    faces = detector.detect(frame)
    draw_detections(image=frame, faces=faces)
    cv2.imshow("UniFace", frame)

    if cv2.waitKey(1) & 0xFF == ord("q"):
        break

cap.release()
cv2.destroyAllWindows()
```

Để đạt FPS tốt hơn, cần cân nhắc detector nhẹ, input resolution, provider phần cứng và tần suất chạy predictor phụ trợ.

---

## 15. Tracking qua video

Luồng cơ bản:

```text
Frame → Detector → [bbox + confidence] → BYTETracker → track_id
```

Detection trả lời **“khuôn mặt ở đâu trong frame này?”**; tracking trả lời **“detection này có phải cùng đối tượng với frame trước hay không?”**.

Xem chi tiết tại [Theo dõi khuôn mặt](modules/tracking.md).

---

## Chọn pipeline theo bài toán

| Bài toán | Pipeline tối thiểu |
|---|---|
| Đếm/tìm mặt | Detection |
| So khớp danh tính | Detection → Alignment → Recognition |
| Tìm người trong kho ảnh | Detection → Recognition → FAISS |
| Video analytics | Detection → Tracking → Predictor cần thiết |
| AR/Avatar | Detection → Landmark/Face Mesh → Pose/Gaze |
| Ảnh chân dung | Matting / Parsing |
| Bảo vệ dữ liệu | Detection → Privacy |
| Kiểm soát truy cập | Detection → Quality → Liveness → Recognition → calibrated threshold |

---

## Bước tiếp theo

- [Tổng quan kiến trúc](concepts/overview.md)
- [Kho mô hình](models.md)
- [Pipeline ảnh](recipes/image-pipeline.md)
- [Video & Webcam](recipes/video-webcam.md)
- [Ngưỡng & hiệu chuẩn](concepts/thresholds-calibration.md)
