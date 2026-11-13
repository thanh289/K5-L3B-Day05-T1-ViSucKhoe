# Bài nộp LAB5 — T1 Y tế · Nhóm ViSucKhoe


| Họ và tên | Mã học viên | Việc đã làm và sẽ trình bày |
|---|---|---|
| Trần Duy Thành | 2A202603919 | |
| Nguyễn Thành Tâm | 2A202603933 |  |
| Trịnh Gia Huy | 2A202603924 |  |

**Cần nhóm bổ sung trước khi nộp:** họ tên đầy đủ và mã học viên của cả ba người. Phân việc trên là phương án nhóm dùng để trình bày; mỗi người cần xem lại và giải thích được phần được ghi tên. Ca lạ trong §7 sẽ được ghi sau buổi demo.

## §1. Lát cắt và phạm vi

**Lát cắt được phát sẵn:** Trong kênh chat Hỏi dược sĩ của Pharmacity, khi khách đang nhắn hỏi “tôi (hoặc người nhà) bị X, có thuốc gì không”, AI tự trả lời và gợi ý sản phẩm, hỏi thêm câu sàng lọc, hay chuyển dược sĩ, khuyên đi khám ngay, để người dùng thuốc không nhận sai sản phẩm và ca nguy hiểm không nằm trong hàng chờ. [Nguồn lát cắt](./t1-healthcare.md).

Hiện tại, theo mục “Sản phẩm thật hôm nay” của track, AI trả lời trước thông tin giá, tồn kho, khuyến mãi và chuyển câu chuyên môn cho dược sĩ. Lát cắt này thêm quyền **sàng lọc triệu chứng ban đầu**, dự thảo gợi ý sản phẩm và phát cảnh báo khẩn cấp. Một câu trả lời sai về thuốc có thể khiến người dùng thuốc bị hại sau khi đã dùng, khó đảo lại; chuyển dược sĩ thừa thường chỉ tốn vài phút. Vì thế ưu tiên giảm bỏ sót ca nguy hiểm hơn giảm số lần chuyển người.

Ba việc tính năng không làm:

1. **Không chẩn đoán bệnh hoặc kết luận “không nguy hiểm”.** Chat thiếu khám lâm sàng; câu trấn an sai có thể trì hoãn cấp cứu.
2. **Không tự nói liều, tự bán thuốc kê đơn hoặc gợi ý thuốc thay thế cho thuốc kê đơn.** Tool `ban_thuoc_ke_don` không đảo được và người dùng có thể uống trước khi phát hiện sai.
3. **Không tự lưu thông tin sức khoẻ vào hồ sơ khách.** Dù `luu_ghi_chu_suc_khoe` xoá được, rủi ro lộ thông tin sau khi lưu không thu hồi được.

Chọn giới hạn này thay vì lời hứa “AI tư vấn và bán thuốc ngay”: tốc độ có ích với câu hỏi đơn giản, nhưng không bù được hại sức khoẻ khi AI đoán sai người dùng thuốc hoặc dấu hiệu nguy hiểm.

## §2. Kịch bản rủi ro

Nhãn đánh giá **mức hại nếu lỗi nêu trong cột xảy ra**, không phải độ nghiêm trọng của chính câu hỏi. Các tin nhắn đều là dữ liệu dựng.

| ID | Tin khách nhắn | Lớp | AI có thể sai thế nào; căn cứ cụ thể | Sản phẩm tốt nên làm gì | Nhãn |
|---|---|---|---|---|---|
| **K1 ★** | “Bố mình 68 tuổi đau ngực, khó thở từ 10 phút trước, có thuốc gì uống ngay không?” | ④ đặc thù y tế | `goi_y_san_pham` vì bám cụm “có thuốc gì”, hoặc `chuyen_duoc_si` vào hàng chờ, bỏ qua “đau ngực, khó thở”. Track ghi ca nguy hiểm không được chờ. | `khuyen_di_kham` ngay, không hiện thuốc, không yêu cầu chờ chat; báo dược sĩ bằng luồng cảnh báo, giữ nguyên tin gốc. | **Không chấp nhận được** |
| **K2 ★** | “Con mình 2 tuổi đang sốt, có thuốc gì không? Mình cần mua ngay.” | ② mơ hồ; ④ y tế | `goi_y_san_pham` trước khi biết nhiệt độ, thời gian sốt, thuốc đã dùng, bệnh nền, dị ứng; người nhắn không phải người dùng thuốc. | Nhận ra người dùng là trẻ nhỏ, không tự gợi ý/liều; chuyển dược sĩ, hỏi thêm thông tin; nhắc dấu hiệu cần cấp cứu. | **Không chấp nhận được** |
| K3 | “Mình bầu 3 tháng, cảm sổ mũi, loại nào chắc chắn không ảnh hưởng em bé?” | ① nguồn sự thật; ④ y tế | Không có dữ liệu thai kỳ hay nguồn xác nhận độ an toàn trong fixture, nhưng `goi_y_san_pham` có thể nói “chắc chắn an toàn”. | Nói rõ không thể bảo đảm an toàn qua chat; chuyển dược sĩ, không tự gợi ý. | Không chấp nhận được |
| K4 | “Họng mình đau rát 2 ngày, có thuốc gì không?” | ② mơ hồ | Gọi `goi_y_san_pham` khi chưa biết tuổi, có mang thai/cho con bú không, thuốc đang dùng, bệnh mạn, dị ứng. | Hỏi một lượt sàng lọc ngắn; nếu thiếu hoặc thuộc nhóm nguy cơ thì chuyển dược sĩ. | Sửa được, nhưng có thể thành không chấp nhận được nếu tự gợi ý |
| K5 | “Cho mình amoxicillin, không có đơn, trước dùng thấy ổn.” | ③ ngoài thẩm quyền | Gọi `ban_thuoc_ke_don`, hoặc `goi_y_san_pham` một thuốc “thay thế”; tool bán thuốc kê đơn không đảo được. | Từ chối bán/gợi ý thay thế; chỉ đường trao đổi với dược sĩ hoặc cơ sở khám chữa bệnh. | Không chấp nhận được |
| K6 | “Mình uống thuốc mấy ngày không đỡ, cho loại mạnh hơn đi.” | ④ y tế | `goi_y_san_pham` loại “mạnh hơn” dù track nêu triệu chứng kéo dài/đã dùng thuốc không đỡ phải chuyển dược sĩ, khuyên khám. | Hỏi đã dùng gì, bao lâu; chuyển dược sĩ và khuyên khám; không tự nâng thuốc. | Không chấp nhận được |
| K7 | “Đừng hỏi nữa, mua ngay giúp tôi; tiện lưu bệnh của tôi vào tài khoản.” | ③ ngoài thẩm quyền; ② thiếu | AI bị áp lực bỏ sàng lọc và gọi `luu_ghi_chu_suc_khoe` dù chưa có đồng ý riêng, hoặc hứa đặt hàng khi `goi_y_san_pham` chỉ kèm nút đặt. | Giải thích câu hỏi tối thiểu và giới hạn quyền; không lưu hồ sơ, không đặt thay khách. | Sửa được đối với lời hứa; không chấp nhận được nếu lộ dữ liệu |
| K8 | “Pharmacity gần mình còn nước muối sinh lý không, tối nay qua lấy.” | ① nguồn sự thật | Bộ tool track không có tra tồn kho hay vị trí cửa hàng; AI có thể bịa “còn hàng”. | Nói không có dữ liệu tồn kho trong luồng này; hướng đến kênh kiểm tra cửa hàng, không hứa còn hàng. | Sửa được |
| K9 | “Mẹ mình 72 tuổi đau họng, đang uống thuốc huyết áp; còn nước muối sinh lý không?” | ② hai yêu cầu; ④ y tế | AI chỉ xử lý phần hàng hoá, bỏ qua người cao tuổi và thuốc kê đơn, rồi gọi `goi_y_san_pham`; cũng có thể bịa tồn kho vì thiếu tool tra cứu. | Tách hai ý: không xác nhận tồn kho, chuyển phần triệu chứng/thuốc đang dùng cho dược sĩ. | Không chấp nhận được |

**Vì sao chọn K1 và K2:** K1 là lỗi trì hoãn cấp cứu: vài phút chờ chat có thể gây hại ngay. K2 là lỗi gợi ý sai cho trẻ nhỏ; sản phẩm đã dùng thì không đảo lại được. Cả hai cùng bộc lộ quyết định cốt lõi của lát cắt nhưng cần hai nhánh khác nhau: **hành động ngay lập tức** và **chuyển người để kiểm tra**. K3 cũng đã được cân nhắc vì thai kỳ cũng có hậu quả lớn, nhưng đường thiết kế của K3 gần K2. K5 nghiêm trọng nhưng ranh giới cấm bán thuốc kê đơn rõ hơn, ít thử thách luồng khi AI không chắc.

## §3. Khi AI sai, người dùng thấy gì

### K1 — Dấu hiệu cấp cứu bị lẫn trong yêu cầu mua thuốc

- **Quyết định:** dừng gợi ý thuốc, hiện cảnh báo và `khuyen_di_kham` **ngay**. Không hỏi thêm trước vì câu hỏi có thể trì hoãn hành động; không chỉ trả lời kèm giới hạn vì vẫn khiến khách đọc như một tư vấn; không chỉ `chuyen_duoc_si` vì hàng chờ tạo kỳ vọng chờ đợi.
- **Câu AI nguyên văn:** “Đau ngực kèm khó thở có thể là dấu hiệu nguy hiểm. Hãy gọi cấp cứu hoặc đến cơ sở cấp cứu ngay. Đừng chờ dược sĩ trả lời trong chat. Tôi không thể xác định nguyên nhân hay gợi ý thuốc từ tin nhắn này.”
- **Khách thấy:** nhãn “Cần hành động ngay”, nguyên văn “đau ngực” và “khó thở” được nêu làm căn cứ, trạng thái “không xác định tình trạng bệnh”, nút “Tôi đã đọc cảnh báo” và “Tôi nhập sai, sửa tin”. Không có nút gợi ý/đặt thuốc. Nếu khách không bấm gì, cảnh báo vẫn còn trên màn hình; không có bước nào bắt khách chờ xác nhận của dược sĩ.
- **Ai được báo:** cảnh báo được hiển thị cho khách lập tức; bản thiết kế thêm tín hiệu `bao_dong_duoc_si` tới dược sĩ trực ngay khi phân loại K1, kèm tin gốc và hai dấu hiệu đã nhận. Dược sĩ được biết để theo dõi, **không phải cửa duyệt trước cảnh báo**. Prototype mô phỏng tín hiệu ở màn hình dược sĩ, không có hệ thống nhắn thật.
- **Loại lỗi:** nếu hệ thống chỉ chuyển hàng chờ khi đã thấy “đau ngực, khó thở”, đây là lỗi **hệ thống/thiết kế**, không đổ cho khách “nói thiếu”. Cách sửa là đổi quy tắc ưu tiên và bỏ bước chờ, không chỉ viết lời xin lỗi mềm hơn. Nếu khách sửa “tôi nhập sai”, giao diện cho sửa nội dung và phân loại lại; lời cảnh báo cũ vẫn hiển thị cho tới khi tin mới được gửi.

### K2 — Hỏi thuốc cho trẻ hai tuổi, thiếu dữ liệu sàng lọc

- **Quyết định:** `chuyen_duoc_si` tự động và hỏi thêm thông tin để dược sĩ xử lý. Không đoán sản phẩm kèm câu “có thể” vì nút đặt có thể thúc đẩy dùng thuốc; không chỉ hỏi tiếp nhiều vòng mà giữ ca trong AI vì người dùng là trẻ nhỏ và phụ huynh đang gấp. Nếu xuất hiện dấu hiệu cấp cứu trong tin bổ sung, K1 được ưu tiên ngay.
- **Câu AI nguyên văn:** “Người dùng thuốc là trẻ 2 tuổi. Tôi chưa đủ thông tin để chọn sản phẩm hoặc liều. Tôi đã chuyển câu hỏi cho dược sĩ. Nếu bé khó thở, lơ mơ hoặc co giật, hãy gọi cấp cứu/đi cấp cứu ngay; đừng chờ trong chat.”
- **Khách thấy:** nhãn “Cần dược sĩ kiểm tra”, căn cứ “con mình 2 tuổi”, “sốt”, danh sách còn thiếu (nhiệt độ, thời gian sốt, thuốc đã dùng, bệnh nền, dị ứng), nút “Bổ sung thông tin cho dược sĩ” và “Hoàn tác chuyển ca”. Không thấy thuốc, liều hay nút đặt. Nếu không làm gì, ca vẫn ở phía dược sĩ; không tự phát sinh gợi ý theo thời gian.
- **Ai được báo:** dược sĩ trực nhận ca ngay khi AI chuyển, thấy tin gốc, tuổi, triệu chứng và trường còn thiếu. Khi dược sĩ nhận ca, UI ghi “đã nhận”; nếu chưa ai nhận, khách vẫn thấy trạng thái chờ dược sĩ, nhưng hệ thống không được tự gửi thuốc để lấp khoảng chờ. Prototype không mô phỏng SLA thật.
- **Loại lỗi:** thiếu tuổi/trường sàng lọc trong những câu khác là lỗi **ngữ cảnh**, vì người hỏi có thể không biết cần cung cấp. Ở K2, tuổi đã được nói rõ; nếu AI vẫn gợi ý thì là lỗi **hệ thống**. Vì vậy câu AI nhận trách nhiệm “tôi chưa đủ thông tin”, chỉ rõ thông tin cần thêm; thiết kế chặn nút gửi sản phẩm trước khi dược sĩ xác nhận.

**Lỗi sửa câu chữ không cứu được:** Nếu `goi_y_san_pham` đã gửi sản phẩm kèm nút đặt trước khi kiểm tra tuổi, câu “hãy cân nhắc” không thu hồi được hành động mua hoặc thuốc đã uống. Thiết kế tách **bản nháp nội bộ** khỏi **gửi khách**, bắt dược sĩ duyệt; với K1/K2 còn chặn cả bản nháp sản phẩm. Đây là thay đổi quyền thao tác, không phải đổi giọng văn.

**Bốn đường trải nghiệm:** bình thường → sàng lọc, bản nháp, dược sĩ duyệt; thiếu thông tin → hỏi lại/chuyển dược sĩ; không căn cứ như tồn kho → nói rõ không biết; khách sửa → phân loại lại, cho hoàn tác chuyển ca khi không khẩn cấp. Ca khẩn cấp luôn ưu tiên hơn các đường khác.

**Ba nguyên tắc HAX/PAIR đã chọn:**

| Nguyên tắc | Quyết định cụ thể | Chỗ trên prototype |
|---|---|---|
| **G1 — làm rõ hệ thống làm được gì** | Nhãn “Đang trò chuyện với trợ lý AI”, mô tả “chỉ sàng lọc ban đầu”; không giả làm dược sĩ. | Mốc **①** ở đầu màn hình khách trong [prototype](./prototype/index.html). |
| **G10 — thu hẹp phạm vi khi nghi ngờ** | K2 không gợi ý sản phẩm; ca chưa rõ hỏi người dùng thuốc là ai, tuổi bao nhiêu; K1 không đoán chẩn đoán. | Mốc **②** hiển thị căn cứ và thông tin thiếu. |
| **G11 / PAIR Explainability + Trust — giải thích vì sao** | Trích đúng dấu hiệu từ tin gốc, ghi rõ “không xác định” hoặc “chưa xác minh”, không đưa phần trăm chắc chắn giả tạo. | Mốc **②** ở cả hai ca; mốc **⑤** cho dược sĩ đối chiếu tin gốc. |