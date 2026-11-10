# Báo cáo cá nhân — Ngày 2 Object Detection

## 1. Thông tin học viên

- Họ tên: Phan Hải Đăng
- MSSV: 2A202603229
- Email: 26ai.dangph4@vinuni.edu.vn
- Lớp: K5-L2A
- Link repo bài làm: https://github.com/phdang95/K5-L2-DAY02A-PhanHaiDang-2A202603229-ObjectDetection
- Link Colab đã chạy: https://colab.research.google.com/github/phdang95/K5-L2-DAY02A-PhanHaiDang-2A202603229-ObjectDetection/blob/main/notebooks/day2_annotation_to_yolo.ipynb?hl=en#scrollTo=ECSo20ByXoPZ

## 2. Schema và mini guideline

Dẫn tới [GUIDELINE_MINI.md](../../GUIDELINE_MINI.md).

### Tóm tắt ba ca mơ hồ đã quyết định:
1. **`drive_006` - Vật thể #88 (`van 88`)**: Xe minivan màu bạc có giá nóc ở góc dưới bên phải làn đường. Ban đầu phân vân giữa xe 7 chỗ gầm cao (`car`) và `van`. Quyết định gán **`van 88` (label 3)** vì thân xe dạng khối hộp vuông vức (boxy), trần cao và có giá nóc chở đồ thương mại.
2. **`drive_006` - Vật thể #86 (`car 86`)**: Xe con màu trắng (hatchback) và sedan đỏ ở ngã tư. Quyết định gán **`car 86` (label 0)** vì là xe cá nhân 4–5 chỗ gầm thấp, thân vỏ unibody liền khối.
3. **`drive_001` - Xe nhỏ li ti hậu cảnh**: Hàng loạt xe ở tít xa cuối đường kích thước $< 15 \times 15\text{ px}$, nhòe mờ. Quyết định **Bỏ qua (không gán nhãn)** để tránh làm nhiễu mô hình và giảm lỗi False Positive.

- **Áp dụng schema:** Phân chia rõ ràng giữa 4 lớp: `0: car`, `1: truck`, `2: bus`, `3: van`. Tuân thủ chặt chẽ quy tắc box ôm khít vật thể, cắt viền ảnh (truncated) và che khuất (occluded).

## 3. Annotation và self-QC

- Số file nhãn train: 50 file YOLO tương ứng với 50 ảnh (`dataset/labels/train/drive_001.txt` đến `drive_050.txt`).
- Kết quả checker và lỗi đã xử lý: Chạy `tools/check_yolo_labels.py` **ĐẠT 50/50 file hợp lệ**. Đã xử lý lỗi sai lệch thứ tự class do CVAT (remap về chuẩn `0: car`, `1: truck`, `2: bus`, `3: van`).
- Danh sách 10 ảnh đã rà bằng mắt: `drive_001`, `drive_002`, `drive_003`, `drive_005`, `drive_006`, `drive_014`, `drive_027`, `drive_030`, `drive_038`, `drive_048`.
- Lỗi phát hiện khi rà và cách sửa: Ban đầu vẽ thừa nhiều xe ở hậu cảnh xa và gán nhầm xe van thành bus/car. Đã tiến hành xóa bớt các box xe quá nhỏ và điều chỉnh lại nhãn đúng theo schema.
- Commit khóa nhãn ban đầu: `786fe1a`

## 4. Gold QC và rework

| Chỉ số | Trước sửa | Sau sửa nếu có |
|---|---:|---:|
| TP | 456 | 556 |
| FP | 315 | 166 |
| FN | 197 | 97 |
| Precision | 0.5914 | 0.7701 |
| Recall | 0.6983 | 0.8515 |
| F1 | 0.6404 | **0.8087** |
| mean_iou_of_matches | 0.8388 | **0.9180** |
| passed | false | **true** |

Dẫn tới file kết quả: [outputs/annotation_evaluation.json](../../outputs/annotation_evaluation.json).

- **Giải trình rework:**
  - *Trước sửa (F1 = 0.6404):* Bị phạt nhiều False Positive (315 box) do vẽ quá nhiều xe nhỏ mờ ở xa và nhầm lẫn class giữa `bus` và `van` (như ở `drive_048`).
  - *Quá trình rework:* Tiến hành đối chiếu với bộ Gold labels, rà soát lại các ảnh lệch nhiều (`drive_001`, `drive_006`, `drive_030`, `drive_048`), xóa bớt box nhỏ li ti, chuẩn hóa ranh giới class.
  - *Sau sửa (F1 = 0.8087, IoU = 0.9180):* Đạt vượt ngưỡng yêu cầu của lab (`F1 >= 0.80` và `mean IoU >= 0.70`).

## 5. Kết quả YOLO

- Runtime, GPU/CPU: Google Colab, GPU Tesla T4 (CUDA), Ultralytics v8.3, PyTorch 2.x
- Tham số thay đổi so với notebook: Giữ nguyên mặc định theo notebook (`epochs=50`, `imgsz=640`, `seed=20260909`).
- Đường dẫn best checkpoint: `/content/Day2-Lab/outputs/yolo26_runs/vehicle_annotation/weights/best.pt`

| Chỉ số trên 10 ảnh eval | Giá trị |
|---|---:|
| Precision | 29.40% (0.294) |
| Recall | 52.40% (0.524) |
| mAP50 | 37.61% (0.376) |
| mAP50–95 | 24.00% (0.240) |

- **Dẫn link assets kết quả Colab:**
  - [Thư mục ảnh eval predictions](assets/eval_predictions/visuals/)
  - [Thư mục log và biểu đồ train YOLO](assets/yolo26_runs/)

### Trả lời bốn câu hỏi cuối notebook:

1. **Lớp nào recall thấp nhất? Ảnh nào minh họa?**
   - Lớp `bus` và `truck` có recall thấp nhất do sự mất cân bằng dữ liệu nghiêm trọng trong tập train (train chỉ có 34 xe truck, 46 xe bus so với 577 xe car) và tập eval chỉ có vỏn vẹn 2 xe bus, 5 xe truck. Minh họa ở ảnh `drive_058.jpg` các xe tải và xe buýt ở cự ly trung bình/xa bị model bỏ sót.
2. **Nếu mAP50 cao nhưng mAP50–95 thấp, cần kiểm điều gì trên box?**
   - Cần kiểm tra **độ bám sát và hình học của Bounding Box (tightness)**. Chỉ số mAP50–95 đo lường độ chính xác ở các ngưỡng IoU khắt khe từ 0.50 đến 0.95. Nếu mAP50–95 thấp, nghĩa là box dự đoán còn lỏng biên (thừa mặt đường, thừa nền trời), box bị lệch tâm hoặc bao trùm sang xe bên cạnh.
3. **Lỗi class nào có thể liên quan guideline? Dữ kiện nào hỗ trợ nhận xét?**
   - Lỗi nhầm lẫn giữa **`car`** và **`van`**. Dữ kiện hỗ trợ: trên thực tế nhiều dòng xe 7–9 chỗ gầm cao (SUV/Crossover) có hình thể rất giống với xe minivan cỡ nhỏ. Nếu guideline không định lượng rõ tiêu chí về số chỗ ngồi và kết cấu thân xe/mục đích thương mại thì cả người gán nhãn lẫn model đều rất dễ bị nhầm.
4. **Vì sao không dùng 10 ảnh eval để train? Vai trò validation của chúng trong notebook là gì?**
   - Không được dùng 10 ảnh eval để train để tránh hiện tượng **Data Leakage (rò rỉ dữ liệu)** và **Overfitting**, đảm bảo đánh giá khách quan khả năng tổng quát hóa (generalization) của mô hình. Vai trò của 10 ảnh eval trong notebook là tập validation giám sát quá trình học qua từng epoch, cung cấp cơ sở để chọn checkpoint tối ưu nhất (`best.pt`).

## 6. Khó khăn và cách xử lý

| Bước/thời điểm | Lỗi quan sát | Cách đã thử | Kết quả/còn thiếu |
|---|---|---|---|
| Khởi tạo dữ liệu trên CVAT | Nhãn bị đảo thứ tự (bus=0, car=1, truck=2, van=3) | Viết script Python tự động remap lại class ID theo đúng `data.yaml` | Đạt chuẩn 50/50 file cấu trúc YOLO |
| Gold QC lần 1 | F1 chỉ đạt 0.6404 do False Positive cao (315 box) | Rà soát các ảnh lệch nhiều, lọc bỏ xe li ti $< 15\text{ px}$ ở hậu cảnh xa và sửa nhãn van | F1 tăng lên **0.8087**, Mean IoU đạt **0.9180** (Passed) |
| Colab Runtime (01:00 AM) | Máy chủ Colab lúc 1h sáng báo lỗi không gán được GPU ("Không có phần phụ trợ nào có sẵn GPU") do nghẽn server/hết quota | Dùng Antigravity tư vấn phương án chạy CPU và đổi tài khoản Google để tiếp tục cấp phát GPU | Train hoàn tất 50 epochs trên Colab, xuất đầy đủ `day2_colab_outputs.zip` |

## 7. Tự đánh giá theo rubric

Đối chiếu [RUBRIC.md](../../RUBRIC.md):
- Hoàn thành và định dạng YOLO (10đ): ĐẠT (50/50 file chuẩn cấu trúc).
- Đầy đủ vật thể (20đ): ĐẠT (Bao phủ đầy đủ các xe thuộc 4 lớp).
- Độ bám sát và hình học box (20đ): ĐẠT (Mean IoU đạt 0.9180 vượt chuẩn 0.70).
- Phân lớp nhất quán (15đ): ĐẠT (Đã chuẩn hóa ranh giới car/truck/bus/van).
- Mini guideline (10đ): ĐẠT (Đầy đủ định nghĩa và 3 ca mơ hồ cụ thể).
- Gold-set QC (15đ): ĐẠT (F1 = 0.8087 $\ge 0.80$).
- Pipeline YOLO và phân tích (10đ): ĐẠT (Đầy đủ kết quả Colab, chỉ số và phân tích 4 câu hỏi).

## 8. Công cụ AI và tài liệu tham khảo

- Công cụ AI: Sử dụng Antigravity Assistant để confirm các thuật ngữ kỹ thuật trong schema, check chéo và đối chiếu các ảnh ban đầu, kiểm tra cấu trúc định dạng YOLO và phân tích nguyên nhân lỗi đánh giá mô hình. Học viên tự rà soát, kiểm tra trực quan từng ảnh trên CVAT và chịu trách nhiệm toàn bộ về nhãn nộp.
- Tài liệu tham khảo: Khung tài liệu bài học Day 2 (RULES.md, GUIDE.md, RUBRIC.md, SUBMISSION.md), tài liệu Ultralytics YOLO26.
