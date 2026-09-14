# Phiếu quy tắc gán nhãn — Ngày 2

**Họ và tên:** TRẦN THẨM ANH TOÀN<br>
**MSSV:** 2A202602232<br>
**Hình thức:** theo cặp<br>
**Mã cặp:** 21902232

## 1. Phạm vi

- Chỉ gán phương tiện thuộc bốn lớp bên dưới.
- Mỗi phương tiện là một hộp; không gộp nhiều xe.
- Không gán người, xe máy, xe đạp, biển báo hoặc phần phản chiếu.
- Vật thể quá nhỏ hoặc mờ đến mức không thể phân lớp có căn cứ: không đoán; ghi lý do vào nhật ký quyết định.

## 2. Bốn lớp cố định

| Mã | Lớp | Gán khi nhìn thấy | Không gán vào lớp này |
| ---: | --- | --- | --- |
| 0 | `car` (ô tô con) | sedan, hatchback, SUV, taxi, xe bán tải dùng như xe con | xe có thùng/ben rõ; thân xe buýt; xe van thân hộp |
| 1 | `truck` (xe tải) | thùng, ben, sàn hàng hoặc thiết bị công vụ rõ ràng | ô tô con; thân xe buýt; xe van kín một khối |
| 2 | `bus` (xe buýt) | thân xe khách dài, nhiều cửa sổ hoặc hàng ghế | xe van nhỏ; xe tải; ô tô con |
| 3 | `van` (xe van) | thân hộp nhỏ, kín, dùng chở người hoặc hàng | thân xe buýt; khoang hàng tách biệt như xe tải |

Thứ tự lớp là cố định: `0 car, 1 truck, 2 bus, 3 van`.

## 3. Hộp giới hạn

- Vẽ sát phần vật thể nhìn thấy.
- Không ước lượng phần bị xe khác che.
- Vật thể chạm mép ảnh vẫn được gán nếu đủ bằng chứng phân lớp.
- Không để hộp chứa nhiều nền hoặc nhiều phương tiện.

## 4. Ba thuộc tính

| Thuộc tính | Giá trị | Ý nghĩa |
| --- | --- | --- |
| `visibility` (mức nhìn thấy) | `clear` (rõ), `occluded` (bị che), `unclear` (không rõ) | mức bằng chứng nhìn thấy |
| `boundary` (quan hệ mép ảnh) | `inside` (trong ảnh), `truncated` (bị cắt) | vật thể có bị mép ảnh cắt hay không |
| `review_state` (trạng thái xem lại) | `confident` (tự tin), `needs_review` (cần xem lại) | đánh dấu quyết định cần quay lại |

YOLO không lưu ba thuộc tính này. Vì vậy phải xuất thêm `CVAT for images 1.1` từ cùng công việc.

## 5. Ba tình huống mơ hồ

Hoàn thành trước khi xem bài của người khác hoặc bộ nhãn tham chiếu.

### Tình huống A — xe buýt hay xe van?

- **Ảnh và mã vật thể:** `drive_022`, vật thể `bus` tại khoảng `xyxy=(104.08, 351.66, 384.41, 576.38)`.
- **Dấu hiệu nhìn thấy:** thân xe dài, kích thước lớn và có nhiều cửa sổ dọc thân; hình dạng phù hợp xe buýt.
- **Quy tắc áp dụng:** `bus` khi nhìn thấy thân xe khách dài, nhiều cửa sổ hoặc hàng ghế; không xếp xe van nhỏ vào `bus`.
- **Quyết định:** `bus`.
- **Nếu vẫn thiếu bằng chứng, bạn sẽ làm gì?** Không đoán; phóng ảnh 100%, ghi lý do và đặt `review_state=needs_review`, sau đó xin hỗ trợ từ Lab Coach/người cùng cặp.

### Tình huống B — xe tải hay xe van/ô tô con?

- **Ảnh và mã vật thể:** `drive_022`, vật thể `truck` tại khoảng `xyxy=(575.90, 259.06, 628.18, 328.02)`.
- **Dấu hiệu nhìn thấy:** phương tiện có dạng xe tải với cabin phía trước và phần thân hàng phía sau; không phải thân hộp kín kiểu van hoặc thân ô tô con.
- **Quy tắc áp dụng:** `truck` khi có thùng, ben, sàn hàng hoặc thiết bị công vụ rõ ràng; `van` chỉ dùng cho thân hộp nhỏ, kín; `car` dùng cho ô tô con.
- **Quyết định:** `truck`.
- **Nếu vẫn thiếu bằng chứng, bạn sẽ làm gì?** Không dùng màu hoặc kích thước hộp để đoán. Phóng 100%, kiểm tra phần thân xe/cabin và đặt `needs_review` nếu vẫn không đủ bằng chứng.

### Tình huống C — bị che, bị mép ảnh cắt hay không đủ bằng chứng?

- **Ảnh và mã vật thể:** `drive_038`, vật thể `bus` đầu tiên tại khoảng `xyxy=(0.00, 95.92, 43.48, 190.32)`.
- **Dấu hiệu nhìn thấy khi phóng 100%:** phần thân xe và nhiều cửa sổ vẫn đủ để nhận diện là xe buýt; hộp chạm mép trái ảnh nên phần còn lại của xe nằm ngoài khung hình.
- **Giá trị `visibility`:** `occluded`.
- **Giá trị `boundary`:** `truncated`.
- **Trạng thái `review_state`:** `confident`.
- **Lý do:** đủ bằng chứng để xác định `bus`; đồng thời ghi nhận vật thể bị cắt bởi mép ảnh bằng `boundary=truncated`. Không mở rộng hộp ra ngoài ảnh và không suy đoán phần không nhìn thấy.

## 6. Xác nhận tự kiểm tra

- [x] Đã rà đủ bốn ảnh.
- [x] Đã kiểm vật thể thiếu và trùng.
- [x] Đã kiểm lớp và hình học từng hộp.
- [x] Mỗi hộp có đủ ba thuộc tính.
- [x] Đã xử lý mọi hộp `needs_review`.
- [x] Đã hoàn thành ba tình huống trước khi xem nguồn đối chiếu.
- [x] Nếu làm theo cặp, hai người đã xuất bài độc lập trước khi trao đổi.
- [ ] Nếu làm cá nhân, bài riêng đã được kiểm trước khi nhận bộ tham chiếu. Không áp dụng vì hình thức là theo cặp.
- **Số vật thể thực tế:** `123` — gồm `100 car`, `5 truck`, `13 bus`, `5 van`. Mục tiêu 40–60 chỉ là mục tiêu khối lượng, không phải điểm cắt.

## 7. Nguyên tắc khi có bất đồng

Khi hai người trong cặp có khác biệt về class, phạm vi hoặc hình học, không chọn nhãn chỉ vì IoU cao hơn. Quay lại ảnh gốc, đối chiếu dấu hiệu nhìn thấy với quy tắc ở mục 2–4, sau đó thống nhất cách xử lý. Nếu bằng chứng vẫn không đủ, giữ `needs_review` và xin Lab Coach hỗ trợ.

## 8. Ghi chú về export

- YOLO Detection dùng để lưu class và bounding box.
- CVAT for Images 1.1 dùng để giữ `visibility`, `boundary` và `review_state`.
- Không sửa file TXT thủ công chỉ để làm chỉ số đối chiếu đẹp hơn; nếu cần sửa phải sửa trong CVAT rồi export lại.
