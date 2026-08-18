<h1 align="center">🇻🇳 UniFace Việt Nam</h1>

<p align="center"><b>Thư viện Python hợp nhất cho phân tích khuôn mặt</b><br>Phát hiện · Nhận dạng · Tracking · Landmark · Face Mesh · Parsing · Matting · Gaze · Head Pose · Thuộc tính · Chất lượng · Anti-Spoofing · Ẩn danh · Vector Search</p>

<div align="center">

[![PyPI](https://img.shields.io/pypi/v/uniface.svg?label=PyPI)](https://pypi.org/project/uniface/)
[![Python](https://img.shields.io/badge/Python-3.10%2B-3776AB?logo=python&logoColor=white)](https://www.python.org/)
[![License](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Upstream](https://img.shields.io/badge/Upstream-yakhyo%2Funiface-181717?logo=github)](https://github.com/yakhyo/uniface)
[![Docs](https://img.shields.io/badge/Tài_liệu-Tiếng_Việt-0A66C2)](https://base27-cvnss.github.io/vi-uniface/)

</div>

<div align="center">
  <img src="https://raw.githubusercontent.com/yakhyo/uniface/main/.github/logos/uniface_rounded_q80.webp" width="88%" alt="UniFace - thư viện phân tích khuôn mặt hợp nhất cho Python">
</div>

> **`vi-uniface` là bản Việt hóa và biên soạn tài liệu của UniFace.** Lõi kỹ thuật, API Python và cơ chế mô hình được giữ tương thích với dự án gốc [`yakhyo/uniface`](https://github.com/yakhyo/uniface). Bản này tập trung giải thích rõ bản chất, kiến trúc, pipeline, cách dùng và lưu ý triển khai cho người dùng Việt Nam.

## 🚀 Cài nhanh

```bash
pip install "uniface[cpu]"          # CPU / Apple Silicon
pip install "uniface[gpu]"          # NVIDIA CUDA
```

Cài trực tiếp từ fork Việt hóa để đọc tài liệu và phát triển:

```bash
git clone https://github.com/Base27-CVNSS/vi-uniface.git
cd vi-uniface
pip install -e ".[cpu]"             # hoặc .[gpu]
```

> ⚠️ Không nên cài đồng thời `onnxruntime` và `onnxruntime-gpu`, vì chúng dùng chung namespace Python. Hãy chọn đúng một runtime.

## 🧠 UniFace thực chất là gì?

UniFace **không phải một mô hình AI duy nhất**. Đây là một lớp thư viện hợp nhất nhiều mô hình chuyên biệt dưới cùng quy ước dữ liệu và API. Một pipeline điển hình có dạng:

```text
Ảnh / Video
    ↓
Phát hiện khuôn mặt
    ↓
Căn chỉnh ──────────────┐
    ↓                    │
Embedding / Recognition │
    ↓                    │
FAISS / So khớp         │
                         ├─ Landmark / Face Mesh
                         ├─ Thuộc tính
                         ├─ Gaze / Head Pose
                         ├─ Parsing / Matting
                         ├─ Quality / Anti-Spoofing
                         └─ BYTETracker / Privacy
```

Các mô hình chủ yếu chạy qua **ONNX Runtime**, cho phép cùng một codebase hoạt động trên Windows, Linux, macOS, CPU, Apple Silicon và NVIDIA CUDA.

## 🧩 15 nhóm tác vụ chính

| Tác vụ | Mô hình / thành phần | Công dụng |
|---|---|---|
| 👤 Phát hiện khuôn mặt | RetinaFace, SCRFD, CenterFace, YOLOv5/8-Face, BlazeFace | Tìm vị trí khuôn mặt và landmark cơ bản |
| 🪪 Nhận dạng khuôn mặt | AdaFace, ArcFace, EdgeFace, MobileFace, SphereFace | Tạo embedding để xác minh / tìm kiếm |
| 🎯 Theo dõi | BYTETracker | Duy trì ID khuôn mặt qua video |
| 📍 Landmark | 2d106det, PIPNet | 68 / 98 / 106 điểm đặc trưng |
| 🕸️ Face Mesh | MediaPipe Face Mesh | 468 / 478 điểm 3D, có biến thể iris |
| 🧬 Thuộc tính | AgeGender, FairFace, AffectNet, FaceAttribNet | Tuổi, nhóm tuổi, cảm xúc, trạng thái khuôn mặt |
| 🧩 Face Parsing | BiSeNet, XSeg | Phân vùng các thành phần trên khuôn mặt |
| 🖼️ Portrait Matting | MODNet | Tách nền và alpha matte |
| 👁️ Gaze | MobileGaze | Ước lượng hướng nhìn |
| 🧭 Head Pose | 6D rotation | Pitch / yaw / roll của đầu |
| 🛡️ Anti-Spoofing | MiniFASNet | Ước lượng liveness |
| ⭐ Face Quality | eDifFIQA | Chấm điểm chất lượng trước nhận dạng |
| 🫥 Ẩn danh | BlurFace | Che / làm mờ khuôn mặt |
| 🔎 Vector Search | FAISS | Tìm embedding gần nhất |
| 🧠 Pipeline hợp nhất | `FaceAnalyzer` | Ghép detection + recognition + predictor |

## ⚡ Ví dụ đầu tiên

```python
import cv2
from uniface import FaceAnalyzer

analyzer = FaceAnalyzer()
image = cv2.imread("photo.jpg")
faces = analyzer.analyze(image)

for face in faces:
    print("bbox:", face.bbox)
    print("confidence:", face.confidence)
    print("embedding:", face.embedding.shape if face.embedding is not None else None)
```

`FaceAnalyzer()` mặc định chạy pipeline cơ bản. Các mô hình thuộc tính là **opt-in**; chỉ chạy khi bạn truyền predictor tương ứng:

```python
from uniface import FaceAnalyzer, FairFace

analyzer = FaceAnalyzer(predictors=[FairFace()])
faces = analyzer.analyze(image)

for face in faces:
    print(face.sex, face.age_group, face.race)
```

## 🖼️ Ví dụ trực quan

**Phát hiện khuôn mặt**

<img src="https://raw.githubusercontent.com/yakhyo/uniface/main/assets/demo/detection.jpg" width="100%" alt="Face detection demo">

**Face Mesh**

<img src="https://raw.githubusercontent.com/yakhyo/uniface/main/assets/demo/face_mesh.jpg" width="100%" alt="Face mesh demo">

**Head Pose**

<img src="https://raw.githubusercontent.com/yakhyo/uniface/main/assets/demo/headpose.jpg" width="100%" alt="Head pose demo">

**Gaze Estimation**

<img src="https://raw.githubusercontent.com/yakhyo/uniface/main/assets/demo/gaze.jpg" width="100%" alt="Gaze estimation demo">

**Face Parsing**

<img src="https://raw.githubusercontent.com/yakhyo/uniface/main/assets/demo/parsing.jpg" width="100%" alt="Face parsing demo">

**Anti-Spoofing**

<img src="https://raw.githubusercontent.com/yakhyo/uniface/main/assets/demo/spoofing.jpg" width="100%" alt="Anti-spoofing demo">

## 🏗️ Triết lý kiến trúc

- **Module hóa:** mỗi tác vụ là một thành phần chuyên biệt, có thể dùng độc lập.
- **API thống nhất:** detector, recognizer, predictor và `Face` object dùng quy ước chung.
- **Model-on-demand:** trọng số tải khi dùng lần đầu và lưu vào cache.
- **Xác minh trọng số:** model được kiểm tra checksum SHA-256 trước khi dùng.
- **Đa nền tảng:** tối ưu provider phần cứng thông qua ONNX Runtime.
- **Từ thấp đến cao:** có thể dùng class trực tiếp hoặc ghép pipeline bằng `FaceAnalyzer`.

## 📚 Tài liệu tiếng Việt

Tài liệu MkDocs được tổ chức theo cùng tư duy chuyên nghiệp của upstream:

- **Bắt đầu:** cài đặt, quickstart, notebook, model zoo, dataset.
- **Hướng dẫn thực hành:** ảnh, video/webcam, face search, batch, anonymization.
- **Tham chiếu API:** detection, recognition, tracking, landmarks, attributes, parsing, gaze, head pose, spoofing, quality, privacy, stores.
- **Kiến trúc & nguyên lý:** input/output, hệ tọa độ, provider phần cứng, model cache, threshold và calibration.

👉 **Website:** https://base27-cvnss.github.io/vi-uniface/

## ⚖️ Giấy phép và ghi công

- **UniFace core:** MIT License.
- **Tác giả dự án gốc:** Yakhyokhuja Valikhujaev — [`yakhyo/uniface`](https://github.com/yakhyo/uniface).
- **Bản Việt hóa:** cộng đồng Base27-CVNSS.
- Một số **pretrained weights** có giấy phép khác với MIT. Hãy kiểm tra `docs/license-attribution.md` trước khi phân phối hoặc dùng thương mại.

## 🔐 Sử dụng có trách nhiệm

Công nghệ phân tích khuôn mặt có thể liên quan đến quyền riêng tư, thiên lệch mô hình và quyết định nhạy cảm. Không nên xem các dự đoán tuổi, giới tính, nhóm nhân khẩu học, cảm xúc hay liveness là sự thật tuyệt đối. Với hệ thống định danh, kiểm soát truy cập, y tế hoặc quyết định có ảnh hưởng lớn, cần kiểm định dữ liệu, hiệu chuẩn ngưỡng, đo FAR/FRR và duy trì cơ chế giám sát của con người.

---

<p align="center">
  <a href="https://base27-cvnss.github.io/vi-uniface/"><b>📘 Tài liệu Việt hóa</b></a> ·
  <a href="https://github.com/yakhyo/uniface"><b>🌐 Dự án gốc</b></a> ·
  <a href="https://pypi.org/project/uniface/"><b>🐍 PyPI</b></a> ·
  <a href="https://github.com/Base27-CVNSS/vi-uniface"><b>💻 Fork Việt hóa</b></a>
</p>
