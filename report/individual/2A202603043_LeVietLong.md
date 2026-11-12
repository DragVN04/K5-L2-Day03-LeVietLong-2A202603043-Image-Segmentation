# Báo cáo thực hành Day 3 — Image Segmentation

- Mã học viên theo lớp: 2A202603043
- Ngày / CVAT local: 12/11/2026 / CVAT local
- Công cụ đã dùng: CVAT (gán nhãn thủ công bằng Polygon); GitHub Actions (kiểm tra cấu trúc ZIP).

## 1. Bài đã nộp

Em đã thực hiện và xuất dữ liệu của cả 9 task. Sau khi đưa các file ZIP lên GitHub, workflow *Day 3 self-check* trả về `OK` cho tất cả các task. Đây là kết quả kiểm tra cấu trúc dữ liệu, chưa phải điểm đánh giá độ chính xác của nhãn.

| Task | File ZIP đúng tên | Hoàn thành mấy ảnh | Điểm tối đa (coach chấm sau) |
| --- | --- | ---: | ---: |
| easy_semantic | `easy_semantic.zip` | 3 / 3 | 20 |
| medium_instance | `medium_instance.zip` | 3 / 3 | 32 |
| hard_panoptic | `hard_panoptic.zip` | 2 / 2 | 30 |
| cp1_holes | `cp1_holes.zip` | 1 / 1 | 3 |
| cp2_slice | `cp2_slice.zip` | 1 / 1 | 3 |
| cp5_occlusion | `cp5_occlusion.zip` | 1 / 1 | 3 |
| cp3_thin | `cp3_thin.zip` | 1 / 1 | 3 |
| cp4_curb | `cp4_curb.zip` | 1 / 1 | 3 |
| cp6_coverage | `cp6_coverage.zip` | 1 / 1 | 3 |
| **Tổng tối đa** | | **14 / 14** | **100** |

## 2. Một quyết định trước khi dùng gợi ý

Em gán nhãn thủ công toàn bộ bài bằng Polygon trên CVAT, không sử dụng gợi ý tự động. Ở phần `medium_instance`, Em chú ý việc mỗi vật thể phải có một mask riêng ngay cả khi nhiều vật thể cùng class. Khi vẽ, Em dựa vào ranh giới thực tế nhìn thấy, hạn chế lấy cả bóng hoặc nền xung quanh vào mask; phần bị che khuất không được tự ước lượng thêm.

- Ảnh, vị trí và object Medium đầu tiên tự vẽ: **[VẼ TAY 100%]**.
- Class đã chọn: **[VEGETATION, CAR, BUILDING, SIDEWALK, ROAD,....]**.
- Quy tắc chọn biên: Vẽ sát phần nhìn thấy; tách các vật thể khác nhau thành instance riêng, không vẽ thêm phần bị che.
- Gợi ý tự động: **Không sử dụng**. Em tự chọn class và chỉnh Polygon trên CVAT.

## 3. Một lỗi Em tìm thấy và sửa

Trong quá trình xem lại nhãn ở `cp1_holes`, Em chú ý khu vực bánh sau chiếc Honda, nơi có một giá nâng màu đỏ đặt sát xe. Đây là vị trí dễ bị gán thừa vì đường viền giá nâng nằm gần đường viền bánh xe, nhưng giá nâng không phải bộ phận của chiếc Honda. Đồng thời, kính chắn gió và các mảng chi tiết trên xe không phải cứ khác màu là phải khoét khỏi mask.

- Task/ảnh/vùng: `cp1_holes` — bánh sau xe Honda và giá nâng màu đỏ.
- Loại lỗi cần kiểm soát: Gán thừa vùng không thuộc class `motorcycle`.
- Dấu hiệu: Giá nâng là vật thể riêng, nằm sát xe nhưng không phải chi tiết của xe.
- Cách xử lý được xác định khi rà soát: Giữ đường bao xe sát phần thực sự thuộc xe, loại phần giá nâng ra khỏi mask; không khoét kính hoặc chi tiết xe chỉ vì khác màu.
- Tình trạng sửa, Save và export lại: **[XÁC NHẬN THEO THAO TÁC THỰC TẾ TRÊN CVAT]**.

Tại lần chạy GitHub Actions đã xem, `cp1_holes.zip` có 7 annotations và tất cả 9 ZIP đều đạt kiểm tra cấu trúc. Hệ thống chưa có gói ground truth chính thức nên chưa thể kết luận độ chính xác mask hoặc điểm trước/sau chỉnh sửa.

## 4. Ba ca chưa chắc hoặc đã cân nhắc

| Ảnh/vị trí | Hai cách hiểu có thể | Quy tắc/chứng cứ | Quyết định hoặc câu hỏi cho coach |
| --- | --- | --- | --- |
| `cp1_holes` — kính chắn gió và các khoảng hở ở chiếc Honda | Khoét bỏ những vùng có màu/chất liệu khác, hoặc giữ chúng trong mask của xe | Guideline nêu kính và chi tiết xe không tự động là lỗ. Kính chắn gió vẫn là bộ phận của xe, còn khoảng trống nhìn xuyên thấy nền phải xem xét theo hình dạng thực tế | Em giữ phần kính thuộc mask xe, không khoét theo màu sắc; với khoảng trống thật thì kiểm tra kỹ trước khi tô |
| `cp4_curb` — bó vỉa cạnh bồn đất trồng cây bên phải ảnh | Chừa trống dải bó vỉa, hoặc tính bó vỉa về phía `sidewalk` | Hai class là `road` và `sidewalk`; guideline yêu cầu phân biệt bằng chức năng và bó vỉa, không chỉ màu. Khu vực bồn đất không phải toàn bộ là bề mặt vỉa hè | Em dùng chân bó vỉa làm mốc phân cách phần đường nhựa. Cách gán mặt đứng bó vỉa vào `sidewalk` là lựa chọn thực hành cần coach xác nhận nếu bộ đáp án quy định khác |
| `cp5_occlusion` — vật thể bị che thành các phần nhìn thấy rời nhau | Tách thành nhiều object vì có nhiều mảng mask, hoặc giữ một instance | Guideline yêu cầu một vật thể bị che vẫn giữ cùng định danh; chỉ gán phần nhìn thấy, không suy đoán vùng khuất | Em áp dụng nguyên tắc một vật thể là một instance, dù có thể cần nhiều vùng mask rời nhau; khi xuất cần kiểm tra chúng còn giữ cùng instance |

