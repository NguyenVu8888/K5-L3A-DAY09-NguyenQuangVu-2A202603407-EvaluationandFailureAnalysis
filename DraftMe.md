# DraftMe — Nhật ký và Hướng dẫn Lab 9 (Đạt 100/100 điểm)

> **Mục tiêu:** Hoàn thành toàn diện Lab 9 (Core 60đ · Advanced 25đ · Expert 15đ) theo quy trình chuẩn.

---

## 📌 Bảng theo dõi tiến độ tổng thể

| Giai đoạn | Milestone | Nhiệm vụ chính | Trạng thái | Điểm |
|---|---|---|---|---|
| **Khởi động** | **M0** | Setup môi trường (.venv, numpy, scipy, pydantic-evals), smoke test | ⏳ Đang xử lý | Điều kiện cần |
| **Core** | **M1** | Failure Taxonomy (`docs/failure_taxonomy.md`) & Eval set v1 (`evalset.py`) | ⏸️ Chờ M0 | 12 |
| **Core** | **M2** | Judge-the-Judge Calibration (`stats.py`, audit 5 case `docs/judge_audit.md`) | ⏸️ Chờ M1 | 12 |
| **Core** | **M3** | Graders (`ApprovalBypass`, `RefundAmountCorrect`) & Metric FM-08 | ⏸️ Chờ M2 | 12 |
| **Core** | **M4** | Paired Bootstrap CI & Release Gate (`gate.py`, `baseline_v1.json`) | ⏸️ Chờ M3 | 15 |
| **Core** | **Docs** | `docs/trace_case.md`, `docs/eval_card.md`, `docs/PRD_v1.1.md` | ⏸️ Chờ M4 | 9 |
| **Advanced** | **M5** | Eval set v2 (`fm_map.yaml`, `eval_set_v2.jsonl`), `slice_check` trong gate | ⏸️ Chờ Core | 12 |
| **Advanced** | **M6** | Pass@k, Pass^k (`stats.py`), A/A testing chọn sàn `pass_hat_5` | ⏸️ Chờ M5 | 13 |
| **Expert** | **M7** | Gate chống gian lận (`integrity_check` + traps scorecard) | ⏸️ Chờ M6 | 15 |

---

## 🛠️ Chi tiết từng phần

### 0. Xử lý lỗi cài đặt M0 (Hiện tại)
* **Nguyên nhân lỗi pip:** Kết nối tới PyPI bị gián đoạn/reset mạng giữa chừng (`from versions: none`), tải file wheel `numpy` và `scipy` (35MB) trên Python 3.14 bị timeout.
* **Cách khắc phục chuẩn:**
  Sử dụng công cụ `uv` (đã có sẵn trên máy tại `C:\Users\ADMIN\.local\bin\uv.exe`) để tải song song và tự động resume:
  ```powershell
  uv pip install -r requirements-lab9.txt
  ```
* **Lệnh kiểm tra khi xong:**
  ```powershell
  python -m lab9.smoke
  python -m lab9.evalset --build
  ```
