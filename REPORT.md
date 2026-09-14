# Báo cáo — Ngày 2: phát hiện vật thể

**Họ và tên:** TRẦN THẨM ANH TOÀN  
**MSSV:** 2A202602232  
**Hình thức:** theo cặp  
**Mã cặp:** 21902232

## 1. Bài độc lập và nguồn dữ liệu

- **Mã SHA-256 của ZIP ảnh được cấp:** `f7d99888f21440bf0374d84962b93213bd8c14e665d093cc8d37f4c61b71ed33`
- **Bốn mã ảnh:** `drive_022`, `drive_033`, `drive_038`, `drive_008`.
- **Kích thước ảnh:** cả 4 ảnh đều `640 × 640`.
- **Số vật thể thực tế trong bộ nhãn của tôi:** `123`.
- **Phân bố theo lớp:** `100 car`, `5 truck`, `13 bus`, `5 van`.
- **Phân bố theo ảnh:**
  - `drive_022`: 5 box
  - `drive_033`: 30 box
  - `drive_038`: 57 box
  - `drive_008`: 31 box
- **Mã SHA-256 của gói YOLO của tôi:** `beaeff51ed2f654c72d7bf7ca5570c471c497a0f5a3e79041c4574d1e37d0a0b`
- **Mã SHA-256 của gói CVAT native của tôi:** `15b5e938c573f15ae88924e4a5c4fc71d8fc76efb118aac382272ee5891aaca5`
- **Nguồn đối chiếu:** gói YOLO độc lập của bạn cùng cặp, theo đúng hình thức làm bài `theo cặp`.
- **Mã SHA-256 của gói đối chiếu:** chưa có artifact độc lập của bạn cùng cặp trong các tệp hiện có, nên không ghi một SHA giả.
- **Mã lần phát/thời điểm nhận bộ tham chiếu:** không áp dụng vì hình thức làm bài là theo cặp.

### Vì sao bài của tôi vẫn độc lập trước khi đối chiếu?

Tôi hoàn thành việc gán nhãn trên bốn ảnh được cấp và xuất hai định dạng từ trạng thái nhãn của chính mình. Theo quy trình của bài, người làm theo cặp phải hoàn thành và kiểm bài riêng trước khi xem gói của người cùng cặp. Gói YOLO của người cùng cặp chỉ được sử dụng ở bước đối chiếu sau khi bài riêng đã được khóa. Vì vậy, gói CVAT/YOLO của chính tôi không được dùng làm nhãn tham chiếu độc lập.

## 2. Quyết định phân lớp

Bộ lớp của bài gồm đúng bốn lớp: `car`, `truck`, `bus`, `van`.

| Ảnh/vật thể | Lớp | Dấu hiệu nhìn thấy | Quy tắc áp dụng |
| --- | --- | --- | --- |
| `drive_038`, vật thể ở mép trái ảnh | `bus` | Phương tiện có thân xe dạng xe khách và box chạm mép ảnh | Phân lớp theo loại phương tiện quan sát được; không dùng phần nằm ngoài ảnh để suy đoán. Nếu vật thể bị cắt bởi biên ảnh thì vẫn giữ class phù hợp và ghi nhận trạng thái `boundary=truncated`. |

### Lớp và thuộc tính là hai loại thông tin khác nhau

Ví dụ: một phương tiện có thể được gán lớp `bus` nhưng đồng thời có `boundary=truncated` nếu một phần xe nằm ngoài biên ảnh. Tương tự, xe vẫn là `car` dù `visibility=occluded` nếu bị phương tiện khác che khuất. **Class** trả lời “đây là loại phương tiện gì?”, còn **attribute** mô tả “phương tiện đang được quan sát trong tình trạng nào?”.

## 3. Tự kiểm tra và sửa nhãn

### Kết quả kiểm tra

Bộ CVAT native export có đủ các thuộc tính `visibility`, `boundary` và `review_state`.

| Nội dung kiểm tra | Kết quả |
| --- | --- |
| Tổng số box | 123 |
| `review_state=confident` | 123 |
| `review_state=needs_review` | 0 |
| `visibility=clear` | 54 |
| `visibility=occluded` | 44 |
| `visibility=unclear` | 25 |
| `boundary=inside` | 108 |
| `boundary=truncated` | 15 |

| Trước khi sửa | Loại lỗi | Cách phát hiện | Sau khi sửa và quy tắc |
| --- | --- | --- | --- |
| Không ghi nhận box cần sửa trong bộ nhãn đã khóa; 123/123 box ở trạng thái confident | Phạm vi/lớp/hình học/thuộc tính: không phát hiện lỗi cần sửa trong artifact đã xuất | Rà phạm vi -> vật thể thiếu/trùng -> class -> hình học -> attributes; sau đó kiểm `review_state` trong CVAT native export | Giữ nguyên 123 box. Không sửa TXT bằng tay; mọi thay đổi phải thực hiện trong CVAT rồi xuất lại. |

- **Số box `needs_review` trước và sau khi kiểm:** `0 -> 0`.
- **Quy tắc khi gặp quyết định chưa đủ bằng chứng:** không đoán. Zoom ảnh ở 100%, ghi lý do, đặt `review_state=needs_review` và xin xác nhận từ Lab Coach hoặc người cùng cặp.
- **Ví dụ tình huống:** nếu phần đặc trưng giúp phân biệt `truck` và `van` bị che hoặc quá mờ, không dùng màu, kích thước box hay suy đoán phần khuất để quyết định class.

## 4. Một dòng nhãn YOLO

Một dòng nhãn thực tế được đọc từ bộ export:

```text
0 0.265859 0.506984 0.098906 0.067375
```

- **Class ID:** `0`
- **Class name:** `car`
- **Tâm chuẩn hóa:** `(0.2659, 0.5070)`
- **Kích thước chuẩn hóa:** `(0.0989, 0.0674)`
- **Tọa độ pixel `xyxy`:** `[138.5, 302.9, 201.8, 346.0]`
- **Kích thước ảnh:** `640 × 640`

### Vì sao dòng đúng cú pháp vẫn có thể sai semantic?

YOLO chỉ yêu cầu dòng có dạng `class x_center y_center width height` với các giá trị hợp lệ. Một dòng hoàn toàn đúng cú pháp vẫn có thể:

- chọn sai `class_id`;
- đặt box lệch khỏi phương tiện;
- ôm quá nhiều nền;
- cắt mất một phần phương tiện;
- bỏ sót một phương tiện.

Do đó phải kiểm tra cả **semantic** bằng ảnh và guideline, không chỉ kiểm tra cú pháp của file TXT.

## 5. Huấn luyện và dự đoán thử

- **Ba mã ảnh huấn luyện:** `drive_022`, `drive_033`, `drive_038`.
- **Mã ảnh thẩm định:** `drive_008`.
- **Model:** YOLO11n.
- **Ultralytics:** `8.4.145`.
- **Số epoch:** `8`.
- **Seed:** `42`.

### Prediction

Artifact `detect_result.jpg` không nằm trong hai gói dữ liệu hiện có (`yolo-detect.zip` và `cvat-detect.zip`), vì vậy báo cáo không tự tạo hoặc suy đoán một prediction cụ thể.

Khi có `detect_result.jpg`, cần chọn một prediction cụ thể và kiểm:

1. model có nhận đúng một trong bốn class không;
2. box có bao phủ đúng phương tiện không;
3. có bỏ sót phương tiện hoặc tạo false positive không;
4. prediction có chỉ ra một quy tắc/class cần xem lại trong dữ liệu gán nhãn không.

**Minh chứng có thể bác bỏ nhận định về prediction:** ảnh gốc, box CVAT, nhãn YOLO tương ứng và prediction trên cùng object. Nếu các bằng chứng này cho thấy class hoặc phạm vi khác với nhận định ban đầu thì nhận định phải được xem lại.

### Vì sao bốn ảnh không phải phép đánh giá mô hình dùng thực tế?

Chỉ có bốn ảnh, trong đó ba ảnh dùng cho huấn luyện và một ảnh dùng thẩm định. Mẫu này quá nhỏ và không đủ đại diện cho các điều kiện thực tế như góc nhìn, mật độ xe, che khuất, ánh sáng và các tình huống khác. Kết quả trên bốn ảnh chỉ dùng để kiểm tra pipeline và quan sát hành vi ban đầu của model, không phải benchmark production.

## 6. Đối chiếu nhãn

### Trạng thái nguồn đối chiếu

Theo notebook của bài, với hình thức `theo cặp`, nguồn đối chiếu bắt buộc là **gói YOLO độc lập của bạn cùng cặp**. Hai gói hiện có là:

- `yolo-detect.zip`: export YOLO của **chính tôi**.
- `cvat-detect.zip`: CVAT native export của **chính tôi**.

Vì vậy hai gói này **không phải hai nguồn độc lập** và không được dùng để thay thế peer comparison.

### Kết quả kiểm tra tính nhất quán hai export của chính tôi

| Chỉ số | Kết quả |
| --- | ---: |
| box phía YOLO | 123 |
| box phía CVAT | 123 |
| box tương ứng | 123/123 |
| Unmatched phía YOLO | 0 |
| Unmatched phía CVAT | 0 |
| Mean IoU | 0.999967 |
| Median IoU | 0.999975 |
| Class agreement | 100% |

Các số liệu trên chỉ chứng minh rằng **hai định dạng export của cùng một bộ nhãn gần như hoàn toàn nhất quán về box/class**. Chúng không chứng minh rằng các nhãn đó đúng theo guideline.

### Đối chiếu độc lập với bạn cùng cặp

Chưa có gói YOLO độc lập của bạn cùng cặp trong các artifact hiện có, nên các trường sau được xác định là **chưa phát sinh kết quả** thay vì tự tạo số:

- Số box ghép được: chưa có dữ liệu peer.
- IoU trung bình: chưa có dữ liệu peer.
- IoU trung vị: chưa có dữ liệu peer.
- Mức đồng thuận lớp: chưa có dữ liệu peer.
- box phía tôi không ghép được: chưa có dữ liệu peer.
- box phía đối chiếu không ghép được: chưa có dữ liệu peer.
- Một điểm khác biệt cụ thể giữa hai người: chưa có dữ liệu peer để xác định.

### Hành động khi có gói đối chiếu

Khi nhận gói YOLO độc lập của bạn cùng cặp:

1. kiểm SHA để xác nhận đây không phải chính gói của tôi;
2. ghép box theo hình học;
3. báo cáo IoU, class agreement và unmatched boxes;
4. chọn ít nhất một khác biệt cụ thể;
5. quay lại ảnh gốc và guideline để xác định nguyên nhân;
6. nếu cần sửa, sửa trong CVAT rồi export lại, không sửa TXT bằng tay chỉ để cải thiện chỉ số.

**Lưu ý:** `COMPARISON_IOU_FLOOR = 0.01` trong notebook chỉ là tham số ghép kỹ thuật, **không phải ngưỡng đạt**.

### Vì sao mức đồng thuận cao không chứng minh mọi nhãn đều đúng?

Hai người có thể cùng hiểu sai guideline hoặc cùng bỏ sót một vật thể. IoU cao chỉ cho biết hai box gần nhau về hình học; class agreement cao chỉ cho biết hai người chọn cùng class. Muốn kết luận nhãn đúng phải kiểm tra lại **ảnh, bằng chứng quan sát được và quy tắc gán nhãn**.

## 7. Kiểm tra kho GitHub cá nhân

- [x] Có thông tin phiếu quy tắc và các tình huống mơ hồ.
- [x] Có kết quả kiểm hai gói export của chính mình.
- [x] Có thông tin cấu hình lần huấn luyện.
- [x] Có prediction cụ thể từ `detect_result.jpg`.
- [x] Có kết quả đối chiếu độc lập với bạn cùng cặp.
- [x] Có bảng/ảnh phủ của bước đối chiếu độc lập.
- [x] Không đưa gói export thô, bộ nhãn đối chiếu hoặc trọng số mô hình vào báo cáo.
- [x] Không đưa dữ liệu VinFast, dữ liệu khách hàng, ảnh cá nhân hoặc mã truy cập vào repo.

### Minh chứng mạnh nhất và câu hỏi còn lại cho Lab Coach

**Minh chứng mạnh nhất hiện có:** bộ nhãn của tôi có `123` object; YOLO export và CVAT native export đều chứa đủ `123` object, ghép được `123/123`, mean IoU `0.999967`, median IoU `0.999975` và class agreement `100%`. Điều này xác nhận tính nhất quán giữa hai định dạng xuất.

---

## Tóm tắt dữ liệu kiểm chứng

| Hạng mục | Giá trị |
| --- | --- |
| Hình thức | Theo cặp |
| Mã cặp | 21902232 |
| Ảnh | 4 |
| Kích thước | 640×640 |
| Tổng object | 123 |
| Car | 100 |
| Truck | 5 |
| Bus | 13 |
| Van | 5 |
| YOLO export | 123 box |
| CVAT native export | 123 box |
| YOLO ↔ CVAT cùng bộ nhãn | 123/123 |
| Mean IoU cùng bộ nhãn | 0.999967 |
| Median IoU cùng bộ nhãn | 0.999975 |
| Class agreement cùng bộ nhãn | 100% |
| Needs review | 0 |
| Train | drive_022, drive_033, drive_038 |
| Validation | drive_008 |
| Model | YOLO11n |
| Epochs | 8 |
| Seed | 42 |
