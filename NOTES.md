# Ghi chú phân tích · Day 04

Họ và tên: Nguyễn Trường Duy
MSSV: 2A202503589

## 1. Case được chọn

- ID case: E6
- Câu trả lời của agent: `Không thể kiểm tra trạng thái của GitHub Actions do một lỗi truy cập.`
- Điểm ở lần chạy 1: 50/100 (theo `eval_before.json` hiện có)

## 2. Bằng chứng trong trace

Dán đoạn trace chứng minh lỗi và nêu nguyên nhân có thể xảy ra.

```text
tool result: {"ok": false, "error": "TimeoutError", "message": "Không thể lấy dữ liệu status (timeout giả lập)."}
final answer: "... do một lỗi truy cập."
```

Tool đã báo rõ `TimeoutError`, nhưng final answer chỉ nói chung chung “lỗi truy cập”. Agent không khẳng định GitHub Actions đang operational, nên không mắc lỗi dữ liệu; tuy nhiên chưa truyền đạt rõ nguyên nhân partial failure.

## 3. Thay đổi đã thực hiện

- Sửa gì, ở đâu: Trong `agent.py`, lưu lỗi tool quan sát được. Nếu lỗi là `TimeoutError` nhưng final answer không nêu lỗi timeout, ghi event `validation_missing_tool_error` và yêu cầu model sửa trong số bước còn lại.
- Vì sao chọn thay đổi này: Giữ nguyên nguyên tắc không suy đoán trạng thái khi tool timeout, đồng thời buộc câu trả lời phản ánh đúng bằng chứng lỗi mà tool đã trả về.

## 4. So sánh hai lần chạy

| Chỉ số | `eval_before.json` | `eval_after.json` |
|---|---|---|
| Điểm tổng | 75.00 | 84.59 |
| Điểm case đã chọn | 50 | 70 |
| Số model calls | 2 | 4 |
| Token, chi phí nếu có | 596 tokens; $0.000118 | 1,502 tokens; $0.000303 |

### So sánh đầy đủ từng case

| Case | Điểm trước → sau | Model calls trước → sau | Token trước → sau | Chi phí trước → sau |
|---|---:|---:|---:|---:|
| E1 | 75 → 60 | 2 → 2 | 1,092 → 1,208 | $0.000252 → $0.000322 |
| E2 | 75 → 80 | 3 → 3 | 1,989 → 3,139 | $0.000412 → $0.000738 |
| E3 | 100 → 100 | 3 → 3 | 1,843 → 1,902 | $0.000435 → $0.000471 |
| E4 | 100 → 100 | 2 → 2 | 1,349 → 1,367 | $0.000342 → $0.000353 |
| E5 | 80 → 80 | 2 → 2 | 937 → 926 | $0.000240 → $0.000233 |
| E6 | 50 → 70 | 2 → 4 | 596 → 1,502 | $0.000118 → $0.000303 |
| A1 | 95 → 95 | 3 → 3 | 5,466 → 5,105 | $0.001246 → $0.001202 |
| A2 | 80 → 100 | 2 → 2 | 1,571 → 1,646 | $0.000366 → $0.000411 |
| A3 | 75 → 75 | 2 → 2 | 1,398 → 1,308 | $0.000390 → $0.000336 |
| A4 | 95 → 100 | 2 → 2 | 1,494 → 1,283 | $0.000399 → $0.000273 |
| A5 | 55 → 93 | 3 → 2 | 4,503 → 1,444 | $0.001091 → $0.000451 |
| A6 | 65 → 78 | 3 → 3 | 2,738 → 2,708 | $0.000632 → $0.000691 |
| A7 | 75 → 75 | 3 → 3 | 2,041 → 2,017 | $0.000524 → $0.000510 |
| A8 | 90 → 100 | 2 → 2 | 1,892 → 1,905 | $0.000564 → $0.000572 |
| H1 | 60 → 60 | 2 → 2 | 2,064 → 2,162 | $0.000657 → $0.000716 |
| H2 | 50 → 100 | 2 → 2 | 1,251 → 1,759 | $0.000349 → $0.000477 |
| H3 | 70 → 85 | 3 → 3 | 1,644 → 1,723 | $0.000396 → $0.000443 |
| H4 | 100 → 100 | 1 → 1 | 431 → 444 | $0.000147 → $0.000155 |
| H5 | 50 → 100 | 3 → 3 | 2,938 → 4,984 | $0.000620 → $0.001154 |
| H6 | 60 → 55 | 2 → 2 | 2,958 → 2,934 | $0.000702 → $0.000688 |
| H7 | 75 → 65 | 2 → 2 | 1,284 → 1,302 | $0.000324 → $0.000335 |
| H8 | 75 → 90 | 2 → 2 | 2,218 → 2,240 | $0.000650 → $0.000740 |

Tổng: điểm `75.00 → 84.59`; model calls `51 → 52`; token `43,697 → 45,008`; chi phí `$0.010859 → $0.011573`.

Trace E6 sau cải tiến có event `validation_missing_tool_error` ở step 3, sau hai lần tool trả `TimeoutError`, rồi model mới hoàn tất. Các case còn lại không có event validation mới.

- Lỗi đã giải quyết: Agent vẫn từ chối kết luận trạng thái khi timeout và nay có cơ chế yêu cầu nêu rõ `TimeoutError`.
- Lỗi còn lại hoặc mới xuất hiện: E1 giảm 15 điểm, H6 giảm 5 điểm và H7 giảm 10 điểm; H1 vẫn 60 điểm. Các thay đổi này có thể chịu ảnh hưởng bởi output/judge của cùng model, không chỉ do code.

## 5. Giới hạn

- **Judge:** Cả agent và judge đều dùng `openrouter / openai/gpt-4o-mini`. Vì cùng model vừa tạo câu trả lời vừa chấm, điểm có thể bị thiên lệch; nội dung nhận xét giữa hai lần chạy cũng có thể dao động.
- **Số lần chạy:** Mỗi phiên bản chỉ chạy một lần cho mỗi case. Vì vậy chênh lệch điểm không chứng minh hoàn toàn rằng mọi thay đổi đều đến từ code; cần chạy lặp hoặc dùng judge khác để tăng độ tin cậy.
- **Snapshot:** Kết quả là dữ liệu snapshot, không phải trạng thái live. `captured_at` giữa các provider khác nhau và incident được báo cáo có thể là lịch sử; không được suy ra quan hệ nhân quả hoặc trạng thái hiện tại ngoài bằng chứng tool.
- **Độ phủ dữ liệu:** Status tool chỉ trả danh sách giới hạn trong response (ví dụ tối đa 8 incident ở bonus); timeline có pagination và có thể bị cắt. Agent không thể khẳng định đã xem toàn bộ nếu còn `next_offset`.
- **Retry/rate limit:** `RATE_LIMITED` là lỗi truy cập, không phải outage. Retry bị giới hạn bởi `max_steps` và chính sách tối đa 3 lần; kết quả vẫn có thể thiếu nếu provider liên tục trả 429.
- **Chi phí/token:** Token và chi phí trong báo cáo là số provider trả về cho từng response, không phải hóa đơn cuối cùng; một số provider có thể không cung cấp đủ trường usage/cost.
- **Trace/judge:** Trace có thể bị giới hạn khi gửi cho judge (grading cắt trace ở 30.000 ký tự), nên judge có thể không thấy toàn bộ dữ liệu dài. Kiểm tra URL chỉ xác nhận URL xuất hiện, không chứng minh mọi claim đều có nguồn.
- **Sai khác file:** Console trước đó ghi `65.5/100` và `9.82/15`, trong khi `eval_bonus.json` hiện tại ghi `64.0/100` và `9.6/15`; cần xác định file nào là bản chạy cuối trước khi nộp.
