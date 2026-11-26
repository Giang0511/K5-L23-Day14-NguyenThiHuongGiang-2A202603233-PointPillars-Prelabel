# Báo cáo Day 14 — Nguyễn Thị Hương Giang — 2A202603233

- Mã học viên dùng với `--student`: `233`
- `seed_tag` trong `frames_A.txt`: `AC-_MUFc`

## 1. Dự đoán trước sweep (bước 5b)

- Tham số tôi nghĩ giảm công sửa nhiều nhất: `min_points = 5`, `score_thr = 0.2`
- Vì sao:
  1. Số đo cấu hình gốc cho thấy việc xóa box chiếm nhiều chi phí nhất: 33 box
     cần xóa (trọng số 0.3) tương đương 9.9 điểm, chiếm khoảng 64.3% tổng chi phí
     15.4. Thêm box (4 box, 4.0 điểm) chiếm khoảng 26%, sửa box (3 box, 1.5 điểm)
     chiếm khoảng 9.7%. Vì vậy cần tìm cách giảm số box dự đoán thừa.
  2. Theo số đo từng frame, box thừa tập trung ở s01f14: frame này có 22/33 box
     cần xóa (khoảng 66.7%), chi phí 8.1/15.4. Frame đó có 28 box dự đoán (19 Car,
     2 Pedestrian, 7 Cyclist), trong khi s03f07 chỉ cần xóa 7 box và s04f17 chỉ cần
     xóa 4 box. Trên ảnh BEV, các box thừa hiện màu đỏ và không khớp vật thể nào
     trong nhãn chuẩn. Tôi dự đoán phần lớn là các dự đoán có confidence score thấp
     hoặc ít điểm LiDAR. Tổng cộng mô hình dự đoán 71 box trong khi nhãn chuẩn chỉ
     có 42 box.
  3. `min_points` lọc các box theo số điểm LiDAR nằm bên trong box, còn
     `score_thr` lọc các dự đoán theo confidence score. Kết hợp hai tham số này
     loại được các box thừa mà không cần xóa tay.
  4. Rủi ro: các vật thật như Pedestrian và Cyclist, đặc biệt ở xa cảm biến hoặc
     bị che khuất, thường có ít điểm LiDAR và confidence thấp hơn ô tô. Nếu đặt
     ngưỡng quá cao, các vật thật này có thể bị loại nhầm và phải thêm lại bằng
     tay. Vì vậy tôi chọn `min_points = 5` và `score_thr = 0.2` ở mức tương đối
     thấp chứ không chọn mức cao hơn.

## 2. Tập A: sửa tay và tune

Số đo cấu hình gốc theo từng frame:

| Frame | Xóa | Thêm | Sửa | Chi phí (gốc) |
|---|---:|---:|---:|---:|
| s01f14 | 22 | 1 | 1 | 8.1 |
| s03f07 | 7 | 2 | 2 | 5.1 |
| s04f17 | 4 | 1 | 0 | 2.2 |
| **Tổng** | **33** | **4** | **3** | **15.4** |

So sánh cấu hình gốc và cấu hình tốt nhất:

| Cấu hình | Xóa | Thêm | Sửa | Chi phí |
|---|---:|---:|---:|---:|
| Gốc | 33 | 4 | 3 | 15.4 |
| `best_config.json` | [..] | [..] | [..] | [..] |

- Tham số đã đổi trong `best_config.json`: [tên tham số: giá trị gốc → giá trị mới].
- Kết quả leave-one-out (các dòng `scripts/pp_sweep.py` in ra):

  ```
  [dán output pp_sweep.py ở đây]
  ```

- Cấu hình có ổn định trên từng frame không? Vì sao tôi vẫn chọn (hoặc không chọn) nó:
  [Nêu chi phí từng frame của cấu hình mới so với gốc (8.1 / 5.1 / 2.2), frame nào
  giảm, frame nào không. Nếu leave-one-out cho kết quả gần với kết quả khi tune
  trên cả 3 frame thì cấu hình ổn định; nếu lệch nhiều thì nói rõ là cấu hình có
  thể bị kéo theo s01f14 vì frame này chiếm phần lớn box thừa.]

## 3. Trước/sau (bước 8)

![so sánh](../../d14_<ID>/compare/<frame>.jpg)

- `scripts/pp_compare.py` báo giảm [..]% công sửa trên tập A (chi phí từ 15.4
  xuống [..]).
- Dự đoán ở mục 1 đúng hay sai? [Đúng/Sai/Đúng một phần]. Giải thích bằng số liệu:
  số box xóa giảm từ 33 xuống [..], số box thêm đổi từ 4 thành [..], số box sửa
  đổi từ 3 thành [..]. [Nếu số box thêm tăng thì nói rõ đó là cái giá của việc lọc
  mạnh: một số vật thật bị loại cùng box thừa, đúng như rủi ro đã nêu ở mục 1.]

## 4. Tập B

Số đo từ `scripts/pp_prelabel.py --frames d14_<ID>/frames_B.txt --config d14_<ID>/best_config.json --gt d14_<ID>/ref_B.json`:

| Xóa | Thêm | Sửa | Chi phí |
|---:|---:|---:|---:|
| [..] | [..] | [..] | [..] |

- Loại box tốn công sửa nhất ở tập B: [Xóa/Thêm/Sửa], chiếm [..]% chi phí, vì
  [lý do ngắn]. [So sánh với tập A: cấu hình tune trên tập A có còn giảm công sửa
  trên tập B không.]

## 5. Giới hạn

- Chỉ có 3 frame có nhãn để tune, trong đó s01f14 chiếm 22/33 box cần xóa của
  cấu hình gốc, nên cấu hình tốt nhất có thể bị lệch theo một frame và chưa chắc
  tổng quát hóa sang dữ liệu khác.
- Mô hình KITTI không có lớp Truck/Bus, nên các vật thể loại này không thể được
  dự đoán đúng bằng cách chỉnh tham số lọc.
- Chi phí chỉ là công thức trọng số (xóa 0.3, thêm 1.0, sửa 0.5), chưa phải thời
  gian sửa thật của người gán nhãn.
- Việc lọc theo `min_points` và `score_thr` có thể loại nhầm vật thật ít điểm
  (Pedestrian, Cyclist ở xa), và chi phí thêm lại box (trọng số 1.0) cao hơn nhiều
  so với chi phí xóa (0.3).