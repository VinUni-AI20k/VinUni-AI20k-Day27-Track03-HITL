# BÁO CÁO BÀI TẬP LAB 27 — TRACK 3: HITL PR REVIEW AGENT

**Chương trình:** VinUni AI20k Cohort 4 (Batch 03)  
**Học viên:** Lâm Hải Dương  
**Mã số sinh viên (MSSV):** 2A202602676  
**GitHub Repository:** [ElysiaTheElysier/K4-DAY27-LamHaiDuong-2A202602676](https://github.com/ElysiaTheElysier/K4-DAY27-LamHaiDuong-2A202602676)  
**Upstream Repository:** [VinUni-AI20k/VinUni-AI20k-Day27-Track03-HITL](https://github.com/VinUni-AI20k/VinUni-AI20k-Day27-Track03-HITL)

---

## 1. Giới thiệu & Mục tiêu
Bài lab xây dựng một hệ thống **Agent tự động Review Pull Request có sự can thiệp của con người (Human-in-the-Loop - HITL)** bằng **LangGraph**, **OpenAI-compatible LLM (OpenRouter)**, và giao diện **Streamlit**.

Hệ thống giải quyết nút thắt cổ chai trong quy trình review mã nguồn:
- **Tự động hoá** các PR an toàn / đơn giản (> 72% confidence): Agent tự động post comment phê duyệt mà không làm phiền reviewer.
- **Can thiệp tối thiểu** với các PR có rủi ro trung bình (58% – 72% confidence): Agent tạm dừng luồng (`interrupt()`), hiển thị phân tích, lý do và 3 nút bấm nhanh **Approve / Reject / Edit**.
- **Leo thang thông minh** với các PR tiềm ẩn rủi ro bảo mật hoặc mơ hồ về thiết kế (< 58% confidence): Agent đặt các câu hỏi trọng tâm cần làm rõ cho lập trình viên/reviewer, sau đó tổng hợp lại review dựa trên câu trả lời.
- **Audit Trail toàn diện**: Mỗi bước quyết định và tương tác người dùng đều được ghi nhận vào SQLite database (`audit_events`) với các trường dữ liệu có thể truy vấn và replay.

---

## 2. Trạng thái hoàn thành 100% (5/5 Bài tập)

| Bài tập | File mã nguồn | Mô tả nội dung hoàn thiện | Trạng thái |
|---|---|---|:---:|
| **Exercise 1** | `exercises/exercise_1_confidence.py` | Tích hợp LLM phân tích diff PR, tự động tính độ tin cậy và phân luồng điều kiện (Conditional Edges). | ✅ Hoàn thành |
| **Exercise 2** | `exercises/exercise_2_hitl.py` | Sử dụng `interrupt()` với `MemorySaver` checkpointer và resume bằng `Command(resume=...)`. | ✅ Hoàn thành |
| **Exercise 3** | `exercises/exercise_3_escalation.py` | Sinh 2–4 câu hỏi làm rõ khi gặp PR rủi ro cao, tổng hợp lại review sau khi nhận phản hồi từ reviewer. | ✅ Hoàn thành |
| **Exercise 4** | `exercises/exercise_4_audit.py` | Sử dụng `AsyncSqliteSaver`, ghi nhận đầy đủ 10 sự kiện vào bảng `audit_events`, hỗ trợ `audit.replay`. | ✅ Hoàn thành |
| **Exercise 5** | `app.py` | Xây dựng giao diện web tương tác Streamlit đầy đủ tính năng: card phê duyệt, form trả lời câu hỏi, lịch sử audit. | ✅ Hoàn thành |

---

## 3. Chi tiết kiểm thử và bằng chứng thực thi

### 3.1. Exercise 1 — Confidence Routing
Kiểm thử trên 2 PR mẫu của kho lưu trữ `VinUni-AI20k/PR-Demo`:
1. **PR #1** (`https://github.com/VinUni-AI20k/PR-Demo/pull/1`): Thêm trường `priority` vào task tracking (~40 dòng code).
   - **Confidence:** `65%`
   - **Nhánh kích hoạt:** `human_approval`
   - **Hành động cuối:** `pending_human_approval`
2. **PR #2** (`https://github.com/VinUni-AI20k/PR-Demo/pull/2`): Thêm xác thực đăng nhập và đồng bộ cloud (có nhiều điểm đỏ: MD5, plaintext token, SQL injection).
   - **Confidence:** `45%`
   - **Nhánh kích hoạt:** `escalate`
   - **Hành động cuối:** `pending_escalation`

### 3.2. Exercise 2 — HITL Interrupt & Resume
- Agent phân tích PR #1, nhận diện confidence 65%, kích hoạt `interrupt()`.
- Giao diện terminal hiển thị khung Approval Request màu xanh:
  - Tóm tắt PR, phân tích của LLM, preview diff.
  - Reviewer nhập lựa chọn (`approve` / `reject` / `edit`) và phản hồi.
- Graph tiếp tục chạy và commit comment lên GitHub PR khi được chấp thuận.

### 3.3. Exercise 3 — Escalation & Synthesis
- Agent phân tích PR #2, nhận diện confidence 45%, sinh ra 4 câu hỏi trọng tâm về bảo mật token, thuật toán băm mật khẩu MD5, cơ chế session và bộ test case.
- Reviewer trả lời câu hỏi làm rõ ngữ cảnh.
- Node `synthesize` gọi lại LLM kèm câu trả lời của reviewer, tinh chỉnh review nâng confidence lên `70%` và đăng review hoàn chỉnh lên PR.

### 3.4. Exercise 4 — Structured SQLite Audit Trail
Chạy kiểm thử phiên hoàn chỉnh và xem lại bằng công cụ `audit.replay`:
```bash
$ uv run python -m audit.replay --list
                            Recent review sessions                             
┌─────────────┬─────────────┬─────────────┬─────────────┬────────────┬────────┐
│ thread_id   │ pr_url      │ started     │ last_event  │ worst_risk │ events │
├─────────────┼─────────────┼─────────────┼─────────────┼────────────┼────────┤
│ 739bddca-7… │ https://gi… │ 2026-10-15… │ 2026-10-15… │ med        │ 7      │
└─────────────┴─────────────┴─────────────┴─────────────┴────────────┴────────┘

$ uv run python -m audit.replay --thread 739bddca-741b-4fd8-b85b-b794b706bd96
───────────────── Replay 739bddca-741b-4fd8-b85b-b794b706bd96 ─────────────────
2026-10-15T05:28:13  fetch_pr        conf=0.00 risk=med  decision=pending    reviewer=-               1703ms  Fetched 3 files, head=1646156
2026-10-15T05:28:19  analyze         conf=0.65 risk=med  decision=pending    reviewer=-               5703ms  The changes introduce a new feature...
2026-10-15T05:28:19  route           conf=0.65 risk=med  decision=human_app  reviewer=-                  0ms  Routed to human_approval (65%)
2026-10-15T05:28:19  human_approval  conf=0.65 risk=med  decision=pending    reviewer=-                  0ms  Awaiting human approval
2026-10-15T05:28:19  human_approval  conf=0.65 risk=med  decision=approve    reviewer=human_reviewer     0ms  Good changes
2026-10-15T05:28:20  commit          conf=0.65 risk=med  decision=approve    reviewer=-                953ms  Action: committed
```

### 3.5. Exercise 5 — Streamlit Web UI
Ứng dụng web chạy qua lệnh:
```bash
uv run streamlit run app.py
```
- **Sidebar:** Hiển thị 25 phiên review gần nhất từ bảng `audit_events`. Khi click vào bất kỳ phiên nào, hệ thống nạp và vẽ lại toàn bộ timeline sự kiện.
- **Form URL:** Cho phép nhập URL PR bất kỳ trên GitHub để tiến hành review.
- **Card tương tác:** Tự động hiển thị Approval Card (cho độ tin cậy 58–72%) hoặc Escalation Card (cho độ tin cậy < 58%).

---

## 4. Hướng dẫn chạy

```bash
# 1. Cài đặt dependencies
uv sync

# 2. Khởi tạo môi trường (sao chép .env.example thành .env và điền khóa)
# OPENROUTER_API_KEY=...
# GITHUB_TOKEN=...

# 3. Chạy từng Exercise
uv run python exercises/exercise_1_confidence.py --pr https://github.com/VinUni-AI20k/PR-Demo/pull/1
uv run python exercises/exercise_2_hitl.py --pr https://github.com/VinUni-AI20k/PR-Demo/pull/1
uv run python exercises/exercise_3_escalation.py --pr https://github.com/VinUni-AI20k/PR-Demo/pull/2
uv run python exercises/exercise_4_audit.py --pr https://github.com/VinUni-AI20k/PR-Demo/pull/1

# 4. Xem lại lịch sử phiên
uv run python -m audit.replay --list

# 5. Khởi chạy Streamlit Approval UI
uv run streamlit run app.py
```
