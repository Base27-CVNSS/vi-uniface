# Nguồn gốc dự án & ghi công

`vi-uniface` là **bản Việt hóa tài liệu và trải nghiệm dành cho cộng đồng Việt Nam** của dự án mã nguồn mở [UniFace](https://github.com/yakhyo/uniface).

## Dự án gốc

- **Tên:** UniFace
- **Tác giả / maintainer gốc:** Yakhyokhuja Valikhujaev (`yakhyo`)
- **Kho mã:** `yakhyo/uniface`
- **Mục tiêu:** cung cấp một API Python thống nhất cho detection, alignment, recognition, landmark, face mesh, tracking, parsing, matting, gaze, head pose, attributes, quality, anti-spoofing, privacy và vector search.
- **Giấy phép lõi:** MIT License.

## Bản Việt hóa

Kho `Base27-CVNSS/vi-uniface` tập trung vào:

1. Việt hóa giao diện và cấu trúc tài liệu MkDocs.
2. Giải thích thuật ngữ theo ngữ cảnh Computer Vision thay vì dịch máy từng từ.
3. Bổ sung phần “bản chất – kiến trúc – luồng dữ liệu – khi nào nên dùng”.
4. Giữ API Python và tên class/function nguyên bản để code vẫn đối chiếu được với upstream.
5. Duy trì liên kết về tài liệu, giấy phép và nguồn mô hình gốc.

## Nguyên tắc Việt hóa

Các định danh kỹ thuật như `FaceAnalyzer`, `RetinaFace`, `SCRFD`, `ArcFace`, `embedding`, `bbox`, `landmarks`, `execution provider` không bị đổi tên trong code. Phần tiếng Việt chỉ giải thích ý nghĩa và cách dùng.

Điều này giúp tránh tình trạng tài liệu tiếng Việt dễ đọc nhưng code lại không thể tìm kiếm/đối chiếu với hệ sinh thái Python quốc tế.

## Đồng bộ upstream

Vì đây là fork, các thay đổi mới có thể xuất hiện ở dự án gốc trước. Khi đồng bộ nên:

1. kiểm tra release/changelog upstream;
2. so sánh thay đổi API;
3. merge/rebase code kỹ thuật;
4. cập nhật lại phần mô tả tiếng Việt nếu hành vi đã thay đổi;
5. chạy test và build tài liệu trước khi phát hành.

## Giấy phép mô hình

MIT License của UniFace core **không có nghĩa mọi pretrained weight đều là MIT**. Một số model/weight có giấy phép riêng (ví dụ Apache-2.0, CC BY, GPL hoặc điều khoản nguồn gốc khác).

Trước khi dùng thương mại, hãy xem [Giấy phép & ghi công](license-attribution.md) và kiểm tra lại nguồn model tương ứng.

## Liên kết

- [UniFace upstream](https://github.com/yakhyo/uniface)
- [Tài liệu upstream](https://yakhyo.github.io/uniface/)
- [UniFace trên PyPI](https://pypi.org/project/uniface/)
- [Fork Việt hóa](https://github.com/Base27-CVNSS/vi-uniface)
