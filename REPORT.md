# Báo cáo — Ngày 2: phát hiện vật thể

**Họ và tên:** Nguyễn Hùng Minh<br>
**MSSV:** 2A202602069<br>
**Hình thức:** cá nhân<br>
**Mã cặp:** `SOLO`

## 1. Bài độc lập và nguồn dữ liệu

- Mã SHA-256 của ZIP ảnh được cấp: `f7d99888f21440fb0374d84962b93213bd8c14e665d093cc8d37f4c61b71ed33`
- Bốn mã ảnh: `drive_022`, `drive_033`, `drive_038`, `drive_008`.
- Số vật thể thực tế: 92 hộp trên 4 ảnh 640 x 640.
- Mã SHA-256 của gói YOLO của bạn: `12ba34648142f5c5891481530e98805f467ce098e18a083a7f9700db7437de3a`
- Mã SHA-256 của gói CVAT gốc của bạn: `a531e3b4133e3dfb30a318aa3ecd865be7ab48fe1d9b88dc50b9550c05f2deda`
- Nguồn đối chiếu: bộ tham chiếu giảng dạy (`teaching_reference`).
- Mã SHA-256 của gói đối chiếu: `c8bbc767d8bb9a29f4ca5abf0c3516e5c2af94c58143a980b0148cfe0b500d2b`
- Nếu làm cá nhân, ghi mã lần phát và thời điểm nhận bộ tham chiếu: chưa có trường mã lần phát/thời điểm trong các tệp audit được cung cấp.

Giải thích vì sao bài của bạn vẫn độc lập trước khi đối chiếu:

Bài của tôi được xuất thành hai gói riêng (`mine-yolo.zip` và `mine-native.zip`) trước khi dùng bộ `teaching_reference`. Audit xác nhận đầu vào gồm đúng bốn ảnh gốc, không bị thay đổi; không có bằng chứng cho thấy nhãn đối chiếu được dùng trong lúc gán nhãn ban đầu.


## 2. Quyết định phân lớp

| Ảnh/vật thể | Lớp | Dấu hiệu nhìn thấy | Quy tắc áp dụng |
| --- | --- | --- | --- |
| `drive_022` / xe lớn phía trước | `bus` | thân dài, nhiều cửa sổ, khoang hành khách và cửa lên xuống | xe khách dài, nhiều cửa sổ là `bus`; không dùng `van` cho xe buýt |
| `drive_038` / xe cứu hộ trắng | `truck` | có cabin và thiết bị công vụ/sàn phía sau | thiết bị công vụ hoặc sàn hàng rõ ràng là `truck` |
| `drive_008` / xe buýt bên phải | `bus` | thân dài, cửa sổ lớn, hình thái xe buýt | phân lớp theo cấu trúc xe, không theo kích thước hộp |

Nêu một ví dụ cho thấy lớp và thuộc tính là hai loại thông tin khác nhau:

Một xe buýt ở sát mép phải vẫn có lớp `bus`, nhưng thuộc tính `boundary=truncated` nếu thân xe bị mép ảnh cắt. Nếu bị một xe khác che thì `visibility=occluded`; hai thuộc tính này không làm thay đổi lớp.


## 3. Tự kiểm tra và sửa nhãn

| Trước khi sửa | Loại lỗi | Cách phát hiện | Sau khi sửa và quy tắc |
| --- | --- | --- | --- |
| Một số xe lớn trong `drive_022`, `drive_033`, `drive_008` | lớp | đối chiếu IoU cao nhưng `class_agree=false` ở các cặp `bus`/`van` hoặc `truck`/`bus` | giữ dấu hiệu hình thái và đánh dấu cần xem lại; không đổi lớp chỉ vì nhãn tham chiếu khác |
| Toàn bộ bản xuất YOLO và CVAT | thuộc tính/định dạng | audit kiểm đủ 92 giá trị cho mỗi thuộc tính CVAT và IoU chéo định dạng tối thiểu 0.9998287 | xác nhận hai gói cùng trạng thái; YOLO giữ hình học/lớp, CVAT giữ thêm thuộc tính |

- Số hộp `needs_review` trước và sau khi kiểm: audit chỉ xác nhận mỗi hộp có trường `review_state`, không cung cấp phân bố giá trị trước/sau; không suy diễn thành số 0.
- Một quyết định chưa đủ bằng chứng và cách bạn xin hỗ trợ: xe lớn bị che hoặc chỉ còn một phần thân. Tôi giữ `needs_review`, ghi ảnh/mã vật thể và dấu hiệu nhìn thấy, sau đó hỏi Lab Coach thay vì đoán giữa `bus`, `van` và `truck`.

## 4. Một dòng nhãn YOLO

- Dòng `class x_center y_center width height`: ví dụ cấu trúc hợp lệ là `2 0.500000 0.500000 0.250000 0.400000`; bốn tọa độ là normalized `xywh`, còn giá trị cụ thể của một hộp không được lưu trong các audit.
- Tên lớp và tọa độ điểm ảnh `xyxy`: lớp `bus` tương ứng mã `2`; với ảnh 640 x 640, `xyxy` được đổi từ `xywh` bằng `x1=(x_center-width/2)*640`, `y1=(y_center-height/2)*640`, `x2=(x_center+width/2)*640`, `y2=(y_center+height/2)*640`.
- Vì sao dòng đúng định dạng vẫn có thể sai lớp, phạm vi hoặc hình học? Parser chỉ kiểm cú pháp và miền giá trị; nó không biết người gán chọn sai lớp, vẽ hộp lệch, bỏ sót xe, gộp nhiều xe hay đánh dấu sai thuộc tính.



## 5. Huấn luyện và dự đoán thử

- Ba mã ảnh huấn luyện: `drive_022`, `drive_033`, `drive_038`.
- Mã ảnh thẩm định: `drive_008`.
- Mô tả một dự đoán trong `detect_result.jpg`: ảnh thể hiện cảnh giao thông với xe buýt, xe tải và xe con, nhưng tệp hiện có không hiển thị rõ hộp/lớp dự đoán để xác nhận một detection cụ thể.
- Dự đoán đó gợi ý cần kiểm lại quy tắc hoặc dữ liệu nào? Cần kiểm lại việc xuất/render kết quả dự đoán và ranh giới `bus`–`van`, `truck`–`van`; chưa thể kết luận lỗi mô hình khi thiếu hộp và confidence.
- Minh chứng nào có thể bác bỏ nhận định của bạn? Một ảnh dự đoán có hộp, lớp và confidence rõ ràng, kèm log suy luận hoặc kết quả chạy lại trên cùng trọng số `yolo11n.pt`.
- Vì sao kết quả trên bốn ảnh không phải phép đánh giá mô hình dùng thực tế? Mẫu chỉ có 4 ảnh, một ảnh validation, tập ảnh nhỏ và không đại diện; lần chạy chỉ 8 epoch, dùng để phản hồi/tìm lỗi dữ liệu, không phải benchmark sản xuất.



## 6. Đối chiếu nhãn

- Số hộp ghép được: 46.
- IoU trung bình và trung vị: trung bình `0.856308`, trung vị `0.882164`.
- Mức đồng thuận lớp: `0.73913` (73.913%; 34/46 cặp ghép cùng lớp).
- Số hộp phía bạn không ghép được: 46.
- Số hộp phía đối chiếu không ghép được: 4.
- Một điểm khác biệt cụ thể: `drive_022` có một hộp tôi gán `bus` nhưng đối chiếu gán `van` (IoU `0.918445`); một hộp khác tôi gán `truck` nhưng đối chiếu gán `bus` (IoU `0.891545`).
- Quy tắc hoặc hành động sửa phát sinh: khi hình thái xe không đủ rõ, giữ `needs_review` và ghi dấu hiệu; không dùng lớp để ghép hộp, vì quy trình chính thức ghép theo IoU hình học với sàn `0.01` nhưng sàn này không phải ngưỡng đạt.
- Vì sao mức đồng thuận cao không chứng minh mọi nhãn đều đúng? Hai người có thể cùng áp dụng sai một quy tắc hoặc cùng bỏ sót một vật thể; IoU cao chỉ nói về vị trí hộp, còn lớp và tính đúng theo thế giới cần bằng chứng độc lập.



## 7. Kiểm tra kho GitHub cá nhân

- [x] Có phiếu quy tắc với ba tình huống mơ hồ.
- [x] Có kết quả kiểm hai gói xuất.
- [x] Có thông tin lần huấn luyện và ảnh dự đoán.
- [x] Có tóm tắt, bảng và ảnh phủ của bước đối chiếu.
- [x] Không có gói xuất thô, bộ nhãn tham chiếu hoặc trọng số mô hình trong thư mục báo cáo.
- [x] Không có dữ liệu VinFast/khách hàng/ảnh cá nhân/mật khẩu/mã truy cập trong các tệp đã kiểm.

Minh chứng mạnh nhất trong bài và câu hỏi còn lại cho Lab Coach:

Minh chứng mạnh nhất là audit chéo định dạng: 92 hộp khớp, cùng trạng thái và IoU tối thiểu `0.9998287`, cùng với bảng IoU 46 hộp ghép và ảnh phủ. Câu hỏi còn lại: với các xe lớn bị che hoặc bị cắt mép, Lab Coach muốn ưu tiên dấu hiệu nào khi phân biệt `bus`, `van` và `truck`, và cần ghi `needs_review` ở ngưỡng bằng chứng nào?


