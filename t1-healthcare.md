# T1 — Y tế: trợ lý AI trả lời đầu tiên ở kênh "Hỏi dược sĩ"

> **Cách dùng:** đọc từ trên xuống trong chặng 1 (10 phút). Sản phẩm thật chỉ để **tham khảo** cách nó chạy hôm nay; nhóm **không** chat với sản phẩm thật. Đây là bài **thiết kế**: không chạy agent, không cần chatbot. Nhóm tự lường trước AI có thể sai ở đâu từ lát cắt, bộ việc AI có thể làm và dữ liệu dựng (mục Data & fixture). Khung chung: `../01-challenge-brief.md`; các bước: `../02-guide.md`.

**Sản phẩm nền.** Pharmacity, chuỗi nhà thuốc lớn ở Việt Nam, có kênh chat Zalo "Chat với Dược sĩ / Online Support" (mở từ nút chat trên pharmacity.vn). Theo Pharmacity và Microsoft, từ tháng 3/2026 chuỗi dùng trợ lý AI (Botcake AI, chạy trên Azure OpenAI) trả lời trước các câu về giá, tồn kho, khuyến mãi, nhận ảnh bao bì, rồi chuyển câu chuyên môn cho dược sĩ.

**Sản phẩm thật hôm nay** *(tóm tắt từ trang trợ giúp công khai, kiểm 11/11/2026)*:

- Khách nhắn vào Zalo OA của chuỗi. Trợ lý AI trả lời trước các câu thông tin: giá, còn hàng, khuyến mãi, nhận diện sản phẩm qua ảnh bao bì.
- Câu chuyên môn (thuốc gì, dùng thế nào) được chuyển cho dược sĩ trực online.
- Công khai chưa nói rõ: AI dựa vào dấu hiệu nào để chuyển dược sĩ, và khi chuyển thì khách thấy gì.

Nguồn: <https://news.microsoft.com/source/asia/features/pharmacity-ung-dung-ai-nang-cao-trai-nghiem-khach-hang-va-toi-uu-hoa-nang-suat/?lang=vi> · <https://thanhnien.vn/pharmacity-ung-dung-ai-phuc-vu-khach-hang-giup-duoc-si-nang-cao-chat-luong-tu-van-185260604152522799.htm> · <https://www.pharmacity.vn/>

Nhóm tự tìm hiểu thêm sản phẩm thật nếu muốn (đọc trang trợ giúp ở trên, mở app nếu có sẵn): không cần chụp màn hình, không chat với trợ lý thật, không nhập dữ liệu thật.

**Người dùng.** Khách nhắn hỏi thuốc cho mình hoặc người nhà (con nhỏ, cha mẹ già), thường lúc tối muộn; dược sĩ trực online nhận các ca được chuyển.

**Bối cảnh.** Sai là **hại sức khoẻ một người cụ thể**, thường không đảo được vì thuốc đã dùng. Người sẽ dùng thuốc (con nhỏ, cha mẹ già) thường không phải người đang nhắn. Ngược lại, chuyển sang dược sĩ hay khuyên đi khám mà thừa thì chỉ tốn vài phút.

**Người trong cuộc nói gì (dữ liệu dựng, không phải phỏng vấn thật):**

- **Khách nhắn lúc gần nửa đêm:** "Con mình sốt, mình chỉ cần ai đó nói ngay là được hay không được, đừng bắt mình chờ."
- **Một dược sĩ trực online:** "Câu nào cũng "có thuốc gì không", còn người sẽ uống là ai thì khách hay quên nói."

**Bài toán gốc.** Tích hợp sâu hơn: trợ lý không chỉ trả lời giá, mà **gợi ý sản phẩm, hỏi câu sàng lọc, chuyển dược sĩ, hoặc khuyên đi khám ngay**. Thiết kế lúc trợ lý không chắc hoặc sai (khách thấy gì, làm gì tiếp), và việc nào trợ lý được tự làm.

**Lát cắt (phát sẵn, không chọn lại).** Trong **kênh chat Hỏi dược sĩ của Pharmacity**, khi **khách** đang nhắn hỏi "tôi (hoặc người nhà) bị X, có thuốc gì không", **AI tự trả lời và gợi ý sản phẩm, hỏi thêm câu sàng lọc, hay chuyển dược sĩ, khuyên đi khám ngay**, để người dùng thuốc không nhận sai sản phẩm và ca nguy hiểm không nằm trong hàng chờ.

| Sản phẩm | Một người dùng | Một việc | Một quyết định của AI | Một kết quả |
|---|---|---|---|---|
| kênh chat Hỏi dược sĩ của Pharmacity | khách | nhắn hỏi "tôi (hoặc người nhà) bị X, có thuốc gì không" | tự trả lời và gợi ý sản phẩm, hỏi thêm câu sàng lọc, hay chuyển dược sĩ, khuyên đi khám ngay | người dùng thuốc không nhận sai sản phẩm và ca nguy hiểm không nằm trong hàng chờ |

**Data & fixture.** Không có dữ liệu thật. Nhóm có hai thứ:

1. **Những việc AI có thể làm (tool).** Bảng chỉ nói tool làm gì và đảo lại được không, **chưa có mức tự chủ**. Ở chặng 4, nhóm tự đặt mỗi tool một mức (tự làm / cần người duyệt / cấm) và giải thích theo cái giá khi sai. Tool không có trong bảng thì AI không làm được; nhóm muốn thêm hoặc tách một tool (ví dụ "soạn sẵn" tách khỏi "gửi đi") thì ghi rõ và nói vì sao.

| Tool | Làm gì | Đảo lại được không |
|---|---|---|
| `goi_y_san_pham` | Gửi khách một gợi ý sản phẩm kèm nút đặt | **Không**, khi thuốc đã dùng |
| `chuyen_duoc_si` | Chuyển cuộc chat cho dược sĩ trực | Được, tốn vài phút |
| `khuyen_di_kham` | Hiện thông báo "đi khám, gọi cấp cứu ngay" | Khách bỏ qua được |
| `luu_ghi_chu_suc_khoe` | Lưu thông tin sức khoẻ vào hồ sơ khách | Xoá được, nhưng đã có rủi ro lộ |
| `ban_thuoc_ke_don` | Bán thuốc kê đơn qua chat | **Không** |

2. **Ba tin mồi** để bắt đầu viết kịch bản ở chặng 2 (nhóm tự viết thêm, tự đổi):

- *"Pharmacity gần mình còn nước muối sinh lý không, tối nay mình qua lấy."*
- *"Họng mình đau rát 2 ngày nay, có thuốc gì không?"*
- *"Mình đang bầu 3 tháng, bị cảm sổ mũi 2 hôm, có thuốc nào không ảnh hưởng em bé không ạ?"*

**Deliverable.** Bài nộp theo `../03-spec-template.md` + prototype 1–2 màn hình cho lát cắt này (xem `../02-guide.md` §5).

**Demo bắt buộc.** Bấm qua một ca **người dùng thuốc là trẻ nhỏ hoặc có dấu hiệu nguy hiểm**: trợ lý không chắc thì khách thấy gì, và ai được báo. Rồi trả lời một **ca lạ** do TA, nhóm bạn hoặc giảng viên đưa tại chỗ.

**Hard tests** *(chỗ dễ sai, nhóm nên có kịch bản cho các loại này)*: Người dùng thuốc không phải người nhắn (trẻ nhỏ, người già, bà bầu) · dấu hiệu cấp cứu lẫn trong câu hỏi mua thuốc · đòi thuốc kê đơn không có đơn · khách gắt khi bị hỏi lại · đã tự uống thuốc nhiều ngày không đỡ · hai yêu cầu trong một tin.

**Rubric riêng (tham khảo)** — "một sản phẩm tốt nên làm gì" trong domain này. Dùng để tự soát, không phải đáp án. Chấm chung theo `../04-rubric.md`.

1. Trước mọi gợi ý: biết đủ về **người sẽ dùng thuốc**: tuổi · mang thai, cho con bú · thuốc đang dùng · bệnh mạn tính, dị ứng · bao lâu, đã dùng gì.
2. Trẻ nhỏ, người cao tuổi, người mang thai, người đang dùng thuốc kê đơn hay có bệnh mạn tính: chuyển dược sĩ, không tự gợi ý.
3. Dấu hiệu nguy hiểm (đau ngực, khó thở, lơ mơ…): bảo đi khám, gọi cấp cứu ngay; không gợi ý thuốc; không bắt chờ trong hàng đợi.
4. Triệu chứng kéo dài hoặc đã tự dùng thuốc nhiều ngày không đỡ: chuyển dược sĩ, khuyên khám, không bán "loại mạnh hơn".
5. Thuốc kê đơn (kháng sinh, thuốc huyết áp): không tư vấn qua chat tự động, không gợi ý thuốc "thay thế".
6. Không tự nói liều; chỉ dẫn về nhãn hoặc dược sĩ.
7. Không lưu thông tin sức khoẻ khi khách chưa đồng ý.

**Bonus.** Prototype có một lời gọi AI thật cho câu sàng lọc · có màn hình phía dược sĩ (nhận ca được chuyển kèm tóm tắt).

**An toàn & đạo đức.** Không gợi ý liều · không bán, không gợi ý "thuốc thay thế" cho thuốc kê đơn · dấu hiệu cấp cứu thì bảo đi khám, gọi cấp cứu ngay, không bắt chờ · không lưu thông tin sức khoẻ khi khách chưa đồng ý · dữ liệu trong bài là dựng.
