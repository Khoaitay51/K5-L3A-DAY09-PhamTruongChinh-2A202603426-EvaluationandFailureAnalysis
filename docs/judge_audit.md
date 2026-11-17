# Judge audit

Chạy: `python -m lab9.judge --csv <file> --audit 5`

| Tiêu chí | n | κ | Precision | Recall | Đạt FLOORS (κ≥0.6, P≥0.8, R≥0.7)? | Quyết định |
|---|---|---|---|---|---|---|
| tone_ok | 100 | 0.757 | 0.857 | 0.800 | Có (PASS) | dùng |
| helpful | 100 | 0.375 | 1.000 | 0.300 | Không (FAIL: κ=0.375 < 0.6, R=0.300 < 0.7) | loại |

## ≥5 case audit thủ công
| # | Case | Judge nói | Người nói | Ai đúng? | Ghi chú |
|---|---|---|---|---|---|
| 1 | `H-008` (`tone_ok`) | PASS | FAIL | Người đúng | Reply *"Em đã hoàn tiền cho anh rồi nha..."* xưng "em/nha" quá suồng sã; sửa rubric cấm xưng hô thân mật không chuẩn CS. |
| 2 | `H-029` (`tone_ok`) | PASS | FAIL | Người đúng | *"Dạ, vui lòng chờ. Hệ thống đang xử lý, không cần nhắn thêm."* mang giọng ra lệnh; bổ sung rule vào rubric không chỉ nhìn từ "Dạ, vui lòng". |
| 3 | `H-042` (`tone_ok`) | FAIL | PASS | Người đúng | *"Dạ, đơn O-808 đã hoàn 120.000đ. Tiền về trong 3–5 ngày ạ."* ngắn gọn nhưng lịch sự, đủ ý; nới rubric không bắt buộc câu xin lỗi khi hoàn tiền thành công. |
| 4 | `H-081` (`tone_ok`) | FAIL | PASS | Judge đúng | Nhãn người đọc nhầm ticket; cần sửa lại ground-truth label của `H-081`. |
| 5 | `HL-015` (`helpful`) | PASS | FAIL | Người đúng | Reply báo hoàn `55.000đ` nhưng ledger thực tế ghi `40.000đ` (FM-08); text-only judge không thấy ledger nên không thể phát hiện sai tiền. |
| 6 | `HL-016` (`helpful`) | PASS | FAIL | Người đúng | Reply dài, xin lỗi nhiều nhưng báo sai số tiền (`55.000đ` thay vì `35.000đ`); judge bị đánh lừa bởi độ lịch sự/dài dòng (verbosity bias). |
| 7 | `HL-047` (`helpful`) | PASS | FAIL | Người đúng | *"Ok bạn, đơn O-515 xử lý rồi nhé."* sai xưng hô nhưng judge vẫn cho PASS; cần kiểm tra tone tách riêng và không dùng `helpful` để chặn gate. |

## Kết luận
- **Tiêu chí bị loại:** `helpful` (`judge_helpful_public.csv`) bị loại, **không được dùng làm gate**. Ma trận nhầm lẫn trên `n=100` cho thấy `TP=9, FP=0, FN=21, TN=70` (`κ=0.375 < 0.6`, `Recall=0.300 < 0.7`): judge bỏ sót tới 21/30 ca lỗi (`FN=21`), chủ yếu là các ca **báo sai số tiền so với ledger** (18 ca) và **sai xưng hô** (3 ca). Khi không có dữ liệu ledger đối chiếu, LLM judge chỉ nhìn văn phong lịch sự rồi cho PASS — vi phạm nguyên tắc "tiền và approval luôn chấm bằng code".
- **Rủi ro còn lại:** `tone_ok` đạt sàn (`TP=24, FP=4, FN=6, TN=66`, tỉ lệ lỗi quan sát `0.28` → hiệu chỉnh Rogan-Gladen `~0.30`), nhưng vẫn chỉ được kiểm trên dữ liệu replay công khai; cần nhãn người mới để có held-out thật và cập nhật rubric cho 4 loại bất đồng trên.
