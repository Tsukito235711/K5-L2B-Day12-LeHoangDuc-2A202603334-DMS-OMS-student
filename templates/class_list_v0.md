# Danh sách lớp v0 — LeHoangDuc · Gxx

Viết tiếp từ slide 54 bài giảng. Mỗi lớp trả lời: **model cần nhận ra cái này để làm gì?**

## DMS (ít nhất 6 lớp)

| # | Lớp / thuộc tính | Giá trị | Để làm gì (chức năng DMS) | Nhìn thấy được trên IR? |
|---|---|---|---|---|
| 1 | gaze_zone | road / mirror_l / mirror_r / mirror_c / cluster / center / lap / other | Đo mức chú ý vào đường; phát hiện lơ đãng (nhìn đùi, màn hình quá lâu) | Một phần: hướng đầu thấy rõ, hướng mắt khó khi kính phản xạ hoặc mặt nhỏ |
| 2 | glasses | none / clear / sun / unknown | Quyết định có tin được trạng thái mắt hay không; chọn ngưỡng cho thuật toán nhìn mắt | Hạn chế: trên NIR kính râm có thể trông trong suốt, nên phải có `unknown` |
| 3 | face13 (13 điểm mặt, cờ occluded/outside) | tọa độ 13 điểm | Tính độ mở mắt, tư thế đầu, ngáp; làm đầu vào cho gaze và cảnh báo buồn ngủ | Có, nhưng điểm mí và khóe mắt hay bị che bởi đốm phản xạ trên kính |
| 4 | hand (whose, side, on_wheel) | driver / passenger; left / right; yes / no / unknown | Biết tay có trên vô lăng không; phát hiện thao tác khác (với xe lái tự động, `no` là bình thường) | Có (hình dạng bàn tay), kém khi tay bị che hoặc tối |
| 5 | phone (state) | held / mounted / lap | Phát hiện dùng điện thoại khi lái | Có nếu thấy cạnh, góc hoặc màn hình; không vẽ khi chỉ thấy vật tối gần tai |
| 6 | seatbelt của tài xế | worn_correct / worn_incorrect / not_worn / unknown | Nhắc thắt dây; phát hiện thắt sai | Hạn chế: dây chìm trong bóng tối thì phải ghi `unknown` |

## OMS (ít nhất 6 lớp)

| # | Lớp / thuộc tính | Giá trị | Để làm gì (chức năng OMS) | Nhìn thấy được trên IR? |
|---|---|---|---|---|
| 1 | person + seat | rear_left / rear_middle / rear_right (và hàng trước) | Biết ghế nào có người: nhắc thắt dây, điều khiển túi khí theo ghế | Có (hình dạng người), gọi ghế theo xe nên cần quy ước trái/phải đúng |
| 2 | seatbelt của hành khách | not_worn / unknown (SVIRO không có `worn_*`) | Nhắc thắt dây cho từng ghế | Hạn chế: dải dây mảnh, dễ chìm trong bóng |
| 3 | child_seat (occupied) | no / unknown | Phát hiện ghế trẻ em lắp trong xe; theo dõi ghế đang trống hay không rõ | Có (hình dạng ghế); không nhìn rõ bên trong thì `unknown` |
| 4 | infant_seat (facing) | rear / front | Phát hiện nôi xách và hướng ghế; tắt túi khí ghế trước nếu nôi quay lưng | Có (hình dạng, quai xách) |
| 5 | object (type) | bag / box / cup / other | Phát hiện đồ bỏ quên, vật có thể thành đạn khi va chạm | Có (hình dạng), nhưng mất màu nên phân loại dựa vào dạng |
| 6 | frame_meta.camera_blocked | yes / no | Biết camera bị che, khi đó mọi dự đoán khác không đáng tin | Có (cả khung tối hoặc bị che) |

## 2 câu hỏi leo thang

Câu hỏi bạn muốn hỏi người viết guideline (hoặc khách hàng) vì guideline v1.0 chưa trả lời. Mỗi câu kèm **tên khung** làm ví dụ.

1. Câu hỏi: Khi `child_seat` hoặc `infant_seat` **đang có trẻ ngồi/nằm trong**, gán thế nào? Guideline v1.0 chỉ định nghĩa hai lớp này cho ghế **trống** (`child_seat` chỉ có `occupied = no/unknown`, không có `yes`), và nói bộ ảnh không có trẻ em ngồi, nhưng ảnh thực tế có trẻ.
   Khung ví dụ: S05_rgb (nôi xách có em bé nằm), S01_rgb (bé ngồi trong ghế trẻ em).
   Bạn tạm gán thế nào và vì sao: Chưa vẽ box cho em bé hay bé (guideline bảo báo coach, không gán). Vẫn vẽ box cho ghế theo đúng `seat` (ví dụ nôi ở trái ảnh là `rear_right`); với `child_seat` ghi `occupied = unknown` vì không có giá trị `yes`; với `infant_seat` ghi `facing` theo ví dụ S00 hoặc `unknown` nếu không chắc, và ghi chú rõ trong ghi chú. Lý do: không đoán, và không dạy model sai rằng ghế có trẻ là ghế trống.

2. Câu hỏi: Khi người ngồi **lấn qua hai chỗ**, `seat` chọn theo chỗ nào: theo phần thân nhiều nhất, theo hông, hay theo chỗ dây an toàn? Guideline chỉ nói "ghế gọi theo xe", chưa nói cách chọn khi người không nằm gọn một ghế.
   Khung ví dụ: S08_rgb (người lớn ngồi bên phải ảnh, thân và tay lấn sang chỗ giữa).
   Bạn tạm gán thế nào và vì sao: Chọn ghế mà phần lớn thân người nằm trên đó (ở đây là `rear_left`, phải ảnh theo xe). Lý do: hộp `person` ôm phần thấy được, nên ghế theo phần thân chiếm nhiều nhất là cách dễ kiểm chứng giữa hai người gán nhất.
