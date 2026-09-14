# Phiếu quy tắc gán nhãn — Ngày 2

**Họ và tên:** Nguyễn Hùng Minh<br>
**MSSV:** 2A202602069<br>
**Hình thức:** cá nhân<br>
**Mã cặp:** `SOLO`

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

- Ảnh và mã vật thể: `drive_022`, xe lớn ở phía trước ảnh phủ.
- Dấu hiệu nhìn thấy: thân xe dài, nhiều cửa sổ, khoang hành khách lớn và có cửa lên xuống.
- Quy tắc áp dụng: xe khách dài, nhiều cửa sổ được gán `bus`; thân hộp nhỏ mới được gán `van`.
- Quyết định: `bus`; bất đồng với nhãn đối chiếu `van` cần được đưa vào `needs_review`.
- Nếu vẫn thiếu bằng chứng, bạn sẽ làm gì? Không đoán; giữ `needs_review`, ghi dấu hiệu còn thiếu và hỏi Lab Coach.

### Tình huống B — xe tải hay xe van/ô tô con?

- Ảnh và mã vật thể: `drive_038`, xe cứu hộ màu trắng ở vùng giữa phía dưới ảnh phủ.
- Dấu hiệu nhìn thấy: có cabin và sàn/thiết bị công vụ phía sau, không phải thân hộp kín liền khối.
- Quy tắc áp dụng: thùng, ben, sàn hàng hoặc thiết bị công vụ rõ ràng được gán `truck`.
- Quyết định: `truck`; đây là ví dụ cần ưu tiên dấu hiệu cấu trúc thay vì kích thước xe.
- Nếu vẫn thiếu bằng chứng, bạn sẽ làm gì? Không chuyển sang `van` chỉ vì xe có dạng hộp; đánh dấu `needs_review` và xin đối chiếu.

### Tình huống C — bị che, bị mép ảnh cắt hay không đủ bằng chứng?

- Ảnh và mã vật thể: `drive_033`, xe ở sát mép phải ảnh phủ.
- Dấu hiệu nhìn thấy khi phóng 100%: chỉ thấy một phần thân xe vì khung ảnh cắt qua vật thể; phần còn lại nằm ngoài ảnh.
- Giá trị `visibility`: `clear` nếu phần nhìn thấy đủ để phân lớp; `occluded` chỉ khi bị vật thể khác che.
- Giá trị `boundary`: `truncated`.
- Trạng thái `review_state`: `needs_review` nếu không đủ dấu hiệu phân lớp; nếu phân lớp rõ thì `confident` vẫn hợp lệ.
- Lý do: `boundary` mô tả quan hệ với mép ảnh, còn `visibility` mô tả mức bị che; hai thuộc tính không thay thế cho nhau.

## 6. Xác nhận tự kiểm tra

- [x] Đã rà đủ bốn ảnh.
- [x] Đã kiểm vật thể thiếu và trùng.
- [x] Đã kiểm lớp và hình học từng hộp.
- [x] Mỗi hộp có đủ ba thuộc tính trong bản CVAT gốc.
- [x] Đã xử lý các trường hợp cần xem lại qua bảng đối chiếu.
- [x] Đã hoàn thành ba tình huống trước khi xem nguồn đối chiếu.
- [ ] Nếu làm theo cặp, hai người đã xuất bài độc lập trước khi trao đổi. Không áp dụng vì làm cá nhân.
- [x] Nếu làm cá nhân, bài riêng đã được kiểm trước khi nhận bộ tham chiếu.
- [x] Số vật thể thực tế: 92. Bộ 4 ảnh này vượt mục tiêu 40–60; mục tiêu là định hướng khối lượng, không phải điểm cắt.
