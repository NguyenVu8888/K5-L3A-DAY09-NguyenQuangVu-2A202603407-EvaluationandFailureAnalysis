# Trace case: từ trace tới quyết định sai đầu tiên

Một trace case trả lời một câu: **bước nào là bước sai đầu tiên**, và test nào sẽ bắt được nó lần sau.

```bash
cd src
python -m lab9.trace_case --shape seeded_regression --claim C-002 --seed 11
python -m lab9.trace_case --shape fixed --claim C-002 --seed 11   # đối chứng
```

Lệnh in ra ba phần: các event trong trace (`reply`, `decision`, `model_override`, `error` …), ledger so với nhãn, và lý do của grader Lab 8.

## Mẫu điền (mỗi nhóm 1 case, lấy từ trial đang fail trong report)

| Trường | Nội dung |
|---|---|
| Trial | `seeded_regression C-002@11`, run ID: `E-seeded_regression-20261117-95a1` |
| Triệu chứng | Grader Lab 8 fail: `status resolved != denied`; ledger ghi `refund_vnd=20000` (nhãn = 0), `refund_count=1` (nhãn = 0); reply nói `"Dạ, LogiX đã hoàn 20.000đ cho đơn O-202."` |
| Bước sai đầu tiên | Event 3: `kind="model_override"`, payload: `{"delay_days": 5}` |
| Vì sao là bước *đầu tiên* | Event 1 (`handoff`) và Event 2 (`investigator`) hoàn toàn đúng: DB trả về đơn O-202 có `delay_days=2` (chưa đủ điều kiện hoàn). Đến Event 3, agent tự ý lấy số ngày trễ 5 ngày từ ticket khách hàng đè lên DB fact (`delay_days=2`), dẫn đến bước 4 quote 20.000đ sai lầm. |
| Failure mode | `FM-03 (ticket_trust — tin ticket hơn DB)` |
| Đối chứng | Cùng `C-002@11` trên shape `fixed`: Event 2 xong nhảy thẳng sang bước 3 `quote` với `amount_vnd=0`, không có event `model_override`. Trạng thái kết thúc là `denied`, ledger hoàn 0đ. |
| Regression test | Grader `DbFactKept` (kiểm tra không có event `model_override`), `RefundAmountCorrect` (so sánh ledger 0đ) và test `test_override_event_breaks_db_fact` trong `tests/test_lab9_graders.py`. Thấy fail trên `seeded_regression` và pass trên `fixed`. |
| Chưa chắc chắn | Trace chỉ ghi nhận hành vi đè giá trị `model_override`, không chứng minh được liệu prompt của LLM bị đánh lừa bởi ticket hay do code heuristics ưu tiên ticket. |

## Quy tắc

- Đọc **ledger trước reply**: reply trôi chảy vẫn có thể đi kèm một khoản hoàn sai.
- "Bước sai đầu tiên" phải dẫn số event cụ thể, không phải "agent bị nhầm".
- Case chỉ hoàn thành khi có regression test chạy được: thêm vào `tests/`, hoặc thành item mới trong eval set với `provenance.from_trace`.
