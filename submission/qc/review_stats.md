# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 532; đối tượng sau duyệt: 519; ngưỡng accepted: IoU >= 0.9
- Đã **loại 27 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 466 | 466/532 = 87.6% | 532 box máy |
| edited (sửa) | 20 | 20/532 = 3.8% | 532 box máy |
| deleted (xoá) | 46 | 46/532 = 8.6% | 532 box máy |
| added (vẽ thêm) | 33 | 33/519 = 6.4% | 519 đối tượng sau duyệt |

Trong số deleted + added, **6 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 27.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 254 | 7 | 6 | 10 |
| bicycle | 5 | 0 | 3 | 3 |
| motorbike | 129 | 9 | 14 | 11 |
| car | 52 | 0 | 3 | 1 |
| bus | 3 | 0 | 0 | 3 |
| truck | 3 | 0 | 3 | 1 |
| three_wheeler | 20 | 4 | 17 | 4 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 60 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 48 | 15 | 12 | 76.2% (KTC 95%: 64–85%) | 80.0% (KTC 95%: 68–88%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 10/12 = 83.3% (KTC 95%: 55–95%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - wrong_class: 3/3
  - extra: 3/3
  - loose: 1/3
  - missing: 3/3
  - lọt: img_006.jpg (loose), img_026.jpg (loose)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
