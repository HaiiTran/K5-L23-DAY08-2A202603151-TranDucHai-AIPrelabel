# Báo cáo Day 08 — 2A202603151 - Trần Đức Hải

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person / bicycle / motorbike / car / bus / truck | YOLO26m | 	motorbike =0.25, các lớp khác =0.3 | `r1` - Theo threshold_sweep thì motorbike đạt recall 92% và precision 100% ở mức 0.25. Person và car không đạt recall 90% ở mức <= 0.5; lớp bicycle có 4 mẫu, bus và truck cùng có 1 mẫu nên giữ mặc định.|
| three_wheeler | YOLOE-26s, prompt: tuk-tuk, cyclo, cargo tricycle | 0.15 | `r3` - Lớp có 7 mẫu gold, mức 0.15 đạt recall 100% và precision 78%. |

- Lệnh `merge_prelabels.py` đã dùng (dán nguyên): python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "motorbike=0.25,three_wheeler=0.15" --conf 0.3 --iou 0.6 --split batch,gold --out work/prelabel/merged_review/pre.coco.json 
- **Phiên bản SAM bạn chọn** (SAM 1 / 2.1 / 3) và lý do: SAM 2.1 base. Có sẵn trong core chạy trên Colab. Tạo mask cho 16 box trên 2 ảnh, khoảng 1,75 giây/ảnh.
- Lỗi nào **không sửa được bằng đổi ngưỡng** (ví dụ tuk-tuk bị gọi `car`)? Sửa thế nào? YOLO26 `car` trong img_001.jpg thành `truck` với score 0.8758. Cần phải sửa bằng bằng cách đổi label thủ công khi review vì đổi ngưỡng chỉ tác dụng giữ hoặc loại box chứ không sửa được label.
- Box **điểm cao mà sai** hoặc **điểm thấp mà đúng** (ảnh nào): box truck score 0.8758 `img_001` trùng `car` trong gold. Box truck `img_020.jpg` score 0.07 khớp `truck` trong gold.
- Giới hạn: chọn ngưỡng trên chính gold (4 ảnh) nên số đo máy trên gold là lạc quan: Số mẫu tổng thể các lớp ở mức ít (bicycle 4, bus 1, truck 1, three_wheeler 7). 

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 466 | 466/532 = 87.6% | 532 box máy |
| edited | 20 | 20/532 = 3.8% | 532 box máy |
| deleted | 46 | 46/532 = 8.6% | 532 box máy |
| added | 33 | 33/519 = 6.4% | 519 đối tượng sau duyệt |

- Lớp nào máy tệ nhất? Vì sao (dẫn ảnh)? three_wheeler - chỉ 48.8% box được giữ nguyên, 4 box phải sửa và 17 bị xóa. Máy gán three_wheeler cho tủ điện (img_005), bus (img_018, img_022) và truck (img_034).
- So với gold: precision / recall của **máy**: precision 48/63 = 76.2% và recall 48/60 = 80.0% (mẫu số: 60 đối tượng gold)

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy: 10/12; loại nào lọt: img_006.jpg (loose), img_026.jpg (loose); vì sao (bạn đã bỏ bước nào của checklist)? bước soi đủ bốn cạnh và chỉnh box quá rộng chưa được làm kỹ ở mọi vật.
- Accepted cao mà bắt bẫy thấp (hoặc ngược lại) nói gì về cách bạn duyệt? không kiểm tra kỹ độ khít của từng box.

## 4. Máy có đáng dùng không?

- Thời gian (từ `time_log.csv`): gán tay 5 phút/ảnh vs duyệt 2.22 phút/ảnh. Thiên lệch (bạn gán gold trước nên đã quen cảnh): đã quen cảnh sau khi gán 4 ảnh gold
- Kết luận: có, dùng làm pre-label có người duyệt, dựa trên pipeline giữ 87.6% box máy. Vì `three_wheeler` chỉ giữ 20/41 và xóa 17 nên cần review kỹ lớp này.
- **Ba quy tắc giảm thiên kiến của bạn** (mỗi quy tắc kiểm tra được bằng số hoặc việc làm):
  1. Trước khi pre-label, quét toàn ảnh trái sang phải và ghi số người/xe mình đếm được; sau đó đối chiếu từng vật với box để phát hiện bỏ sót hoặc box thừa.
  2. Kiểm tra 100% box máy về lớp và biên.
  3. Kiểm thử bẫy, sau đó kiểm tra lại biên box trên toàn bộ ảnh đang review.

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| img_013 | SAM 2.1 | 4 | Mask lấn vào người lái | Dính người lái vào mask xe | tuktuk_img013.jpg |
| img_013 | SAM 1 | 3 | Một phần người lái vẫn có mask | Rìa mask còn thô, có vùng thủng/lấn ở khung và bánh xe | tuktuk_img013_sam1.jpg |
| img_004 | SAM 2.1 | 3 | Không gồm người lái hay hàng hóa | Mask phủ thân xe nhưng còn thiếu ở khung và lem ra nền ở vùng bánh | tuktuk_img004_sam2.jpg |
| img_017 | SAM 2.1 | 3 | Không gồm người lái hay hàng hóa | Mask còn lem ở khung và lỗ ở mái che| tuktuk_img017_sam2.jpg |

- Đối chiếu guideline mục 4.2: quy tắc nào giúp, quy tắc nào thiếu? Quy tắc 4.2.1 bám phần nhìn thấy, quy tắc 4.1.4 nói người ngồi trên xe là nhãn riêng với xe. Còn thiếu quy tắc hướng dẫn mask xử lý các khoảng hở lớn trong khung xe tuk-tuk
- *(Tuỳ chọn)* Interactor SAM trong CVAT: làm được / không (lỗi gì, đã thử gì); khác gì offline: 

## 6. Calibration (Part 6)

- Số quyết định: 3 (`calibration_decisions.csv`); quan trọng nhất: CAL-02 — nhãn `three_wheeler` bị dùng cho vật không phải xe ba bánh và cho bus/truck.
- Quy tắc mới đã vào `guideline_v1.1_calibrated.md`: xe có động cơ điện được xếp `motorbike`; nếu ảnh không đủ căn cứ thì đánh dấu `can_review` (CAL-01).
- Nguyên nhân gốc: chủ yếu là máy (nhầm lớp `three_wheeler`); bẫy `loose` bị lọt còn cho thấy cần làm đủ bước soi biên box.

## 7. QC và quyết định (Part 6)

- Mẫu: 6 ảnh (1 ảnh `can_review` + 5 ngẫu nhiên, seed 42); người kiểm: TranDucHai.
- Lỗi theo **đối tượng**: 0/90 = 0.0% · theo **ảnh**: 0/6 = 0.0%. (ngưỡng Release ≤ 2%)
- Top lỗi (trong: bỏ sót, thừa, sai lớp, sai vị trí/kích thước, sai thuộc tính, sai mask) + một ảnh ví dụ mỗi loại:
- **Giới hạn mẫu nhỏ** (cận trên 95% trong `qc_report_auto.md`): tỷ lệ ảnh lỗi là **39%**, mẫu 6 ảnh quá nhỏ để chứng minh tỷ lệ lỗi thật dưới 2%.
- Script kiểm được / không kiểm được (ví dụ ôm sát, cùng bỏ sót với máy, theo checklist thật): kiểm được số lỗi và tỷ lệ theo QC, không kiểm được box có ôm sát vật không hay người và máy có cùng bỏ sót vật không.

**Quyết định: Release**, ngày 2026-11-19. Lý do (đi theo Hold → Rework → Release): Có 0/6 ảnh lỗi, dưới ngưỡng 2%
Nếu Rework / Hold, việc cần làm tiếp:


## 8. Ghi chú cá nhân

- **Dùng AI để làm bài** (công cụ nào, việc gì, kiểm tra thế nào; "không dùng" nếu không):  Có, dùng ChatGPT/Codex để gỡ lỗi Colab, đọc log và số liệu, gợi ý điền phiếu
- Gold có bị nhiễm không (bạn có nhìn pre-label hay hỏi AI trước khi gán xong)? không
- Máy / GPU / CVAT dùng; vướng mắc cài đặt: Colab GPU CUDA (T4), CVAT v2.81.0
- Dùng cache `data/cache/` thay vì tự chạy? không
- Lỗi / bug script đã gặp (nếu có): không
- Một điều bạn làm khác lần sau: Ghi thời gian gán gold riêng trong `time_log.csv`
