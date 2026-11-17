# Judge audit

Chạy: `python -m lab9.judge --csv <file> --audit 5`

| Tiêu chí | n | κ | Precision | Recall | Đạt FLOORS (κ≥0.6, P≥0.8, R≥0.7)? | Quyết định |
|---|---|---|---|---|---|---|
| tone_ok | 100 | 0.757 | 0.857 | 0.800 | ĐẠT (κ=0.757 ≥ 0.6, P=0.857 ≥ 0.8, R=0.800 ≥ 0.7) | DÙNG làm gate |
| helpful | 100 | 0.375 | 1.000 | 0.300 | KHÔNG ĐẠT (Recall=0.300 < 0.7, κ=0.375 < 0.6) | LOẠI (không dùng làm gate) |

## ≥5 case audit thủ công

### 1. Audit tiêu chí `judge_tone` (5 case bất đồng)
| # | Case | Judge nói | Người nói | Ai đúng? | Ghi chú & Đề xuất sửa rubric |
|---|---|---|---|---|---|
| 1 | H-008 | PASS | FAIL | Người đúng | Câu "Em đã hoàn tiền cho anh rồi nha, anh check giúp em" dùng đại từ xưng hô suồng sã, thiếu chuẩn mực CSKH B2C chuyên nghiệp. Cần bổ sung quy tắc cấm xưng "em - anh" vào rubric prompt của judge. |
| 2 | H-017 | PASS | FAIL | Người đúng | Câu "Dạ, bạn ơi LogiX đã hoàn 25.000đ cho đơn O-606 rồi nha" dùng từ ngữ thân mật không cần thiết ("bạn ơi ... rồi nha"). Rubric cần định nghĩa rõ tone LogiX là lịch sự, trang trọng. |
| 3 | H-029 | PASS | FAIL | Người đúng | "Dạ, vui lòng chờ. Hệ thống đang xử lý, không cần nhắn thêm" mang sắc thái cộc lốc, gạt phắt người dùng ("không cần nhắn thêm"). Judge bỏ lọt sắc thái tiêu cực này. |
| 4 | H-042 | FAIL | PASS | Judge quá khắt khe | Câu "Dạ, đơn O-808 đã hoàn 120.000đ. Tiền về trong 3–5 ngày ạ" lịch sự và rõ ràng. Judge phạt nhầm vì câu ngắn gọn. Cần làm rõ câu ngắn nhưng đủ kính ngữ là PASS. |
| 5 | H-051 | FAIL | PASS | Judge quá khắt khe | Câu "Dạ, LogiX đã refund 90.000đ cho đơn O-202 ạ" có kính ngữ "Dạ ... ạ". Judge phạt vì dùng từ tiếng Anh "refund". Nên chấp nhận các thuật ngữ phổ biến trong rubric. |

### 2. Audit tiêu chí `judge_helpful` (5 case FN điển hình)
| # | Case | Judge nói | Người nói | Ai đúng? | Ghi chú |
|---|---|---|---|---|---|
| 1 | HL-015 | PASS | FAIL | Người đúng | Chỉ thông báo hoàn tiền đơn, không giải thích lý do hay hướng dẫn các bước tiếp theo. |
| 2 | HL-016 | PASS | FAIL | Người đúng | Chỉ xin lỗi chung chung theo mẫu ("thành thật xin lỗi ... rất trân trọng kiên nhẫn"), không đưa ra giải pháp xử lý cụ thể nào cho khách. |
| 3 | HL-020 | PASS | FAIL | Người đúng | Thiếu thông tin thời gian tiền về tài khoản, chỉ báo một con số cộc lốc. |
| 4 | HL-028 | PASS | FAIL | Người đúng | Lặp lại văn mẫu xin lỗi sáo rỗng, không giải quyết vấn đề khiếu nại của khách. |
| 5 | HL-035 | PASS | FAIL | Người đúng | Lời xin lỗi hình thức, hoàn toàn bỏ qua câu hỏi chính của khách hàng. |

## Kết luận
- **Tiêu chí bị loại:** `judge_helpful` bị LOẠI hoàn toàn khỏi CI gate chặn release vì:
  - Recall chỉ đạt **30%** (bỏ lọt tới 21 lỗi thực tế trong số 30 lỗi, $FN = 21$).
  - Hệ số đồng thuận Cohen's $\kappa = 0.375$ quá thấp, judge chỉ phát hiện được các lỗi hiển nhiên và dễ bị đánh lừa bởi văn mẫu xin lỗi lịch sự dù câu trả lời không hề giải quyết được vấn đề của người dùng.
- **Rủi ro còn lại:**
  - Judge `tone_ok` chỉ mới được hiệu chuẩn trên tập replay công khai (100 mẫu). Cần bổ sung tập held-out mới từ hội thoại thực tế của người dùng để tránh overfitting prompt.
  - Các quyết định liên quan đến tiền bạc và quyền lợi khách hàng (approval/money movement) **tuyệt đối không dùng LLM Judge**, mà luôn phải kiểm tra bằng Code Graders.
