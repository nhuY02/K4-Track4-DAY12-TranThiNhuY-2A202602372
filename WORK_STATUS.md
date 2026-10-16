# Work Status

## Mục tiêu hiện tại
Hoàn thiện Lab Day 12, đối chiếu evidence thật và điền REPORT.md theo README.md, GUIDE.md và RUBRIC.md.

## Việc đã hoàn thành
- Đọc README.md, GUIDE.md, RUBRIC.md và kiểm tra mã nguồn, tests cùng 10 vị trí TODO.
- Xác nhận notebook Colab là tùy chọn; lab chỉ cần Python/NumPy, pandas, matplotlib và không cần GPU.
- Hoàn thiện 10 hàm TODO trong lab12/.
- Chạy toàn bộ unit tests có sẵn bằng Python runtime đi kèm Codex: 25/25 PASS.
- Phát hiện evidence sẵn có cho MSSV 2A202602759 với 5 seed; xác nhận tất cả file có cùng MSSV/seed và các bước pipeline đã chạy.
- Điền bảng kết quả và cả 8 câu trong REPORT.md dựa trên evidence; lấy mẫu khung TTC thấp để giải thích AEB.
- Tạo 5 commit cục bộ theo nhóm bước, tổng số commit của học viên đạt yêu cầu rubric.
- Chạy `scripts/check_submission.py --save-verify` trong Python UTF-8 mode: 56 PASS, 0 cảnh báo, 0 FAIL; evidence/verify.txt đã được tạo lại với kết quả đạt.
- Chạy checker lần cuối sau khi commit: 56 PASS, 0 cảnh báo, 0 FAIL; cây Git sạch.

## Việc đang làm
- Không còn việc cần làm trong phạm vi hoàn thiện code, evidence và report ở máy local.

## File đã sửa/tạo
- lab12/replay.py
- lab12/scenarios.py
- lab12/models.py
- lab12/stats.py
- lab12/gate.py
- lab12/augment.py
- WORK_STATUS.md (tạo mới để theo dõi task theo yêu cầu).
- REPORT.md (điền kết quả và câu trả lời từ evidence).
- evidence/*.json, evidence/*.png và evidence/verify.txt (bằng chứng chạy thật, đã được checker xác nhận).

## Quyết định quan trọng
- Phạm vi code: chỉ 10 hàm TODO trong lab12/; chưa chỉnh REPORT.md.
- Trước mắt xác minh bằng unit tests cục bộ; chỉ dùng Colab/Kaggle nếu pipeline thực tế cần tài nguyên hoặc môi trường không khả dụng.
- Evidence hiện có dùng MSSV thật 2A202602759, cùng seed chia 5199 và 5 seed huấn luyện; không cần chạy lại pipeline nặng hoặc dùng Colab/Kaggle.
- Checker chạy mặc định vấp encoding cp1252 khi đọc báo cáo UTF-8; chạy lại với Python UTF-8 mode, không sửa script lớp.

## Lỗi/vấn đề còn tồn tại
- Không còn lỗi checker đã biết. Thay đổi đang nằm trên nhánh local `main`, chưa được đẩy lên GitHub.

## Bước tiếp theo cần làm
- Nếu muốn cập nhật repo GitHub, đẩy nhánh local `main` lên `origin`; thao tác này chưa được thực hiện.
