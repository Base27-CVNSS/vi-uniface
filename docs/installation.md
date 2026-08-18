# Cài đặt

Trang này hướng dẫn cài UniFace trên **Windows, Linux, macOS, CPU, Apple Silicon và NVIDIA CUDA**.

---

## Yêu cầu

- **Python:** 3.10 – 3.14
- **Hệ điều hành:** Windows, Linux, macOS
- **RAM:** phụ thuộc số mô hình được nạp đồng thời
- **GPU:** không bắt buộc; NVIDIA CUDA chỉ cần khi muốn tăng tốc GPU

---

## Vì sao có hai gói `cpu` và `gpu`?

UniFace tách runtime thành hai extra:

- `uniface[cpu]` → cài `onnxruntime`
- `uniface[gpu]` → cài `onnxruntime-gpu`

Hai package ONNX Runtime này dùng cùng namespace Python. Nếu cài đồng thời, môi trường có thể bị xung đột file hoặc chọn sai execution provider. Vì vậy hãy chọn **một** biến thể phù hợp với máy của bạn.

---

## Cài nhanh

=== "CPU / Apple Silicon"

    ```bash
    pip install "uniface[cpu]"
    ```

=== "NVIDIA GPU (CUDA)"

    ```bash
    pip install "uniface[gpu]"
    ```

=== "Bản pre-release"

    ```bash
    pip install --pre "uniface[cpu]"
    ```

---

## Windows / Linux + NVIDIA GPU

```bash
pip install "uniface[gpu]"
```

Gói này cài `onnxruntime-gpu`, trong đó đã có cả `CUDAExecutionProvider` và `CPUExecutionProvider`.

Kiểm tra provider:

```python
import onnxruntime as ort
print(ort.get_available_providers())
```

Kết quả nên có:

```text
CUDAExecutionProvider
CPUExecutionProvider
```

!!! warning "Nếu không thấy CUDA"
    Kiểm tra driver bằng `nvidia-smi`, sau đó đối chiếu phiên bản CUDA/cuDNN với phiên bản ONNX Runtime đang dùng. Không nên đoán rằng cứ cài CUDA Toolkit là UniFace sẽ tự dùng GPU.

---

## macOS Apple Silicon

Dùng biến thể CPU:

```bash
pip install "uniface[cpu]"
```

Kiểm tra kiến trúc Python:

```bash
python -c "import platform; print(platform.machine())"
```

Máy Apple Silicon nên trả về:

```text
arm64
```

Nếu trả về `x86_64`, có thể Python đang chạy qua Rosetta và không tận dụng đúng kiến trúc ARM64.

---

## CPU thuần

```bash
pip install "uniface[cpu]"
```

Đây là lựa chọn ổn định nhất nếu bạn không cần CUDA hoặc muốn triển khai trên máy chủ/PC phổ thông.

---

## Cài từ mã nguồn Việt hóa

```bash
git clone https://github.com/Base27-CVNSS/vi-uniface.git
cd vi-uniface
pip install -e ".[cpu]"
```

Nếu dùng NVIDIA CUDA:

```bash
pip install -e ".[gpu]"
```

Cho môi trường phát triển:

```bash
pip install -e ".[cpu,dev]"
```

!!! info "Upstream"
    Fork Việt hóa giữ tương thích với dự án gốc [`yakhyo/uniface`](https://github.com/yakhyo/uniface). Khi cần đối chiếu thay đổi mới nhất, hãy kiểm tra upstream trước khi đồng bộ.

---

## FAISS cho tìm kiếm khuôn mặt

Nếu cần lưu và truy vấn embedding trên tập lớn:

```bash
pip install faiss-cpu
```

Hoặc bản GPU nếu môi trường FAISS/CUDA của bạn hỗ trợ:

```bash
pip install faiss-gpu
```

FAISS không bắt buộc cho detection/recognition cơ bản; nó chỉ cần cho lớp **vector store/search**.

---

## Các dependency chính

| Package | Vai trò |
|---|---|
| `numpy` | Mảng số và tensor đầu vào/đầu ra |
| `opencv-python` | Đọc ảnh, video, xử lý ảnh |
| `scikit-image` | Biến đổi hình học và căn chỉnh |
| `scipy` | Tính toán khoa học hỗ trợ |
| `requests` | Tải trọng số mô hình |
| `tqdm` | Hiển thị tiến trình |
| `onnxruntime` | Suy luận CPU / Apple Silicon |
| `onnxruntime-gpu` | Suy luận NVIDIA CUDA |

Một số mô hình tùy chọn có thể cần `torch` hoặc `torchvision`.

---

## Vòng đời trọng số mô hình

Khi bạn khởi tạo một model lần đầu:

1. UniFace kiểm tra cache cục bộ.
2. Nếu chưa có, model được tải về.
3. File được xác minh bằng checksum SHA-256.
4. Model được nạp vào execution provider phù hợp.
5. Các lần sau dùng lại file trong cache.

Vị trí mặc định:

```text
~/.uniface/models
```

Có thể đổi bằng Python:

```python
from uniface.model_store import set_cache_dir
set_cache_dir("D:/AI/uniface-models")
```

Hoặc biến môi trường:

```bash
set UNIFACE_CACHE_DIR=D:\AI\uniface-models
```

Trên Linux/macOS:

```bash
export UNIFACE_CACHE_DIR=/data/uniface-models
```

---

## Kiểm tra sau cài đặt

```python
import uniface
import onnxruntime as ort
from uniface.detection import RetinaFace

print("UniFace:", uniface.__version__)
print("Providers:", ort.get_available_providers())

detector = RetinaFace()
print("Khởi tạo detector thành công")
```

---

## Chuyển từ CPU sang GPU

Không cài chồng hai runtime. Hãy gỡ sạch trước:

```bash
pip uninstall onnxruntime onnxruntime-gpu -y
pip install "uniface[gpu]"
```

Chuyển ngược về CPU:

```bash
pip uninstall onnxruntime onnxruntime-gpu -y
pip install "uniface[cpu]"
```

---

## Lỗi thường gặp

### `onnxruntime is not installed`

Bạn có thể đã cài `uniface` mà không chọn extra. Cài lại:

```bash
pip install "uniface[cpu]"
```

hoặc:

```bash
pip install "uniface[gpu]"
```

### Đã cài cả `onnxruntime` và `onnxruntime-gpu`

```bash
pip uninstall onnxruntime onnxruntime-gpu -y
pip install "uniface[cpu]"   # hoặc [gpu], chỉ chọn một
```

### Model tải thất bại

Kiểm tra mạng, quyền ghi thư mục cache và dung lượng đĩa. Có thể kiểm tra/tải model qua model store:

```python
from uniface.constants import RetinaFaceWeights
from uniface.model_store import verify_model_weights

path = verify_model_weights(RetinaFaceWeights.MNET_V2)
print(path)
```

### CUDA không được nhận

```bash
nvidia-smi
```

Sau đó kiểm tra:

```python
import onnxruntime as ort
print(ort.get_available_providers())
```

Nếu chỉ có `CPUExecutionProvider`, pipeline đang chạy CPU dù máy có GPU.

---

## Bước tiếp theo

- [Khởi động nhanh](quickstart.md) — chạy pipeline đầu tiên.
- [Tổng quan kiến trúc](concepts/overview.md) — hiểu cách các module phối hợp.
- [Execution Providers](concepts/execution-providers.md) — đi sâu vào tăng tốc phần cứng.
- [Kho mô hình](models.md) — chọn model theo độ chính xác/tài nguyên.
