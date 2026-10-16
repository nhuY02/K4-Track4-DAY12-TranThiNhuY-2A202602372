# REPORT — Lab Day 12

> Điền **bằng lời và số liệu của chính bạn** (lấy từ màn hình và từ `evidence/*.json`). Mỗi câu 4–8 dòng là đủ. Số không có trong bằng chứng của bạn sẽ bị trừ điểm. Báo cáo công khai trên GitHub; giảng viên đối chiếu với `evidence/` và với bài của các bạn khác.

| | |
|---|---|
| Họ và tên | Nguyễn Minh Hiển |
| MSSV (đúng bằng `--student-id` bạn đã chạy) | 2A202602759 |
| GitHub | nmhien165-ui |
| Seed chia dữ liệu / seed mô hình (xem `evidence/closedloop.json`, mục `student`) | 5199 / 4453 |
| Số seed huấn luyện bạn dùng (`--seeds`) | 5 |

## Kết quả cuối

Điền từ `evidence/closedloop.json`, `openloop.json`, `gate.json` (cho mỗi mô hình học, lấy **dải qua các seed**).

| Bộ điều khiển | L2 vòng hở (m) | Va chạm vòng kín trên log (%) | Quyết định cổng |
|---|---|---|---|
| constant-velocity | 1.265 | 20.95 | BLOCK |
| IDM | 1.530 | 0.07 | baseline |
| EgoMLP | 0.768 | 13.53–28.13 | BLOCK |
| PercMLP | 0.625 | 0.55–2.50 | BLOCK |
| PercMLP+ (sau khi sửa) | 0.811 | 0.24–0.35 | RELEASE qua cổng |

## Câu 1 · Dữ liệu
Bước 0 bạn phát hiện những vấn đề gì ở dữ liệu? Vì sao không dùng cột `Acc_LV`, và bạn thay bằng gì? Bao nhiêu quỹ đạo bị loại vì trùng, và vì sao chia theo quỹ đạo?

`data_audit.json` có 992.679 dòng và 9.993 quỹ đạo thô; loại 383 quỹ đạo trùng chính xác, còn 9.610. Tương quan `Acc_LV` với đạo hàm tốc độ xe dẫn chỉ 0,045, trong khi cột gia tốc xe theo sau đạt 0,929, nên `Acc_LV` không đáng tin để tìm ca phanh.
Pipeline dùng gia tốc xe dẫn suy ra từ `Speed_LV` đã làm mượt; dữ liệu chia theo quỹ đạo để các khung liên tiếp và các bản sao không lọt đồng thời vào train và test. Nếu chia theo khung, những đoạn gần như giống hệt sẽ làm điểm test cao giả tạo.
Kiểm tra hình học cũng cho thấy `Spatial_Gap` khớp vị trí trừ chiều dài xe với sai lệch cực đại chỉ 3,64×10⁻¹¹ m. Dù vậy, các quỹ đạo trùng xa nhau theo ID chứng minh cần khử trùng trước khi chia tập.

## Câu 2 · Phát lại
Bản refactor float32 khác bit ở bao nhiêu đoạn, vượt dung sai ở bao nhiêu? Thay đổi AEB làm lệch bao nhiêu đoạn, và vì sao lệch ở đúng những khung đó (mở một đoạn ra xem)? Phát theo thời gian thật làm lộ ra điều gì mà phát nhanh nhất che đi?

Trong 600 đoạn, bản refactor float32 khác chính xác ở cả 600 đoạn nhưng không đoạn nào vượt dung sai; sai khác tuyệt đối lớn nhất là 2,13×10⁻⁵ m/s². AEB làm đổi 6/600 đoạn, tổng cộng 22 khung; ví dụ đoạn 129, khung 82–85 có TTC 1,36–1,49 s nên lệnh IDM bị giới hạn từ khoảng −4,44…−5,15 xuống −6 m/s².
Các khung thay đổi đều ở vùng TTC xấp xỉ ngưỡng 1,5 s: AEB chỉ can thiệp khi xe đang áp sát nhanh, không làm đổi những khung an toàn. Dung sai 0,001 tuyệt đối cộng 0,001 tương đối chấp nhận sai số số thực nhỏ mà vẫn bắt được thay đổi hành vi.
Phát nhanh nhất xử lý 59.659 khung mà không báo trễ hay rớt; phát thời gian thật báo 582 khung trễ và 574 khung rớt, khác dung sai ở 375 đoạn. Vì vậy chạy nhanh che mất việc xử lý không kịp deadline và giữ lại lệnh cũ.

## Câu 3 · Kịch bản
Lưới của bạn (tốc độ cruise, giới hạn khoảng cách, số ô): bao nhiêu % ô **không ai tránh được**? Độ trễ phản ứng 0 → 0,6 s làm vùng đâm của IDM tăng bao nhiêu? Log có phủ vùng khó không? (dẫn giới hạn phanh mạnh nhất của người lái trong log, và phần lưới đòi hỏi hơn thế)

Lưới dùng tốc độ 12 m/s, khoảng cách 2,76–59,31 m, tốc độ tiến gần 0–12 m/s và 2.700 ô. Vùng bất khả thi tăng từ 2,22% khi phản ứng tức thời lên 7,44% với trễ 0,6 s, tức tăng 5,22 điểm phần trăm; IDM đâm đúng vùng vật lý này và có 0% va chạm quy cho bộ điều khiển.
Giới hạn phanh người lái ở phân vị 99,9 là 3,62 m/s²; 8,56% ô đòi phanh mạnh hơn mức này, nên dữ liệu người lái không phủ đủ vùng khẩn cấp. Tôi khai thác được 614 sự kiện phanh xe dẫn, nhưng số sự kiện không đồng nghĩa đã quan sát đủ các tổ hợp tốc độ, khoảng cách và độ trễ khó.
Biên bất khả thi khớp mô phỏng phanh cực đại ở cả hai độ trễ, với 0 ô lệch xa biên trong mỗi lần kiểm chứng. Độ trễ làm vùng va chạm tăng, nhưng không thể quy vùng đó thành lỗi planner vì không bộ điều khiển nào còn đủ khoảng dừng.

## Câu 4 · Vòng hở và vòng kín
Dán bảng L2 và tỷ lệ va chạm của bốn bộ điều khiển (kèm hạng ở mỗi cách chấm). Cặp nào đổi thứ tự? Vì sao IDM có L2 như vậy mà vẫn lái tốt? Vì sao EgoMLP có L2 tốt hơn constant-velocity mà vẫn đâm?

| Bộ điều khiển | Hạng L2 (thấp tốt) | L2 (m) | Hạng va chạm (thấp tốt) | Va chạm (%) |
|---|---:|---:|---:|---:|
| PercMLP | 1 | 0,625 | 2 | 1,18 |
| EgoMLP | 2 | 0,768 | 3 | 19,32 |
| constant-velocity | 3 | 1,265 | 4 | 20,95 |
| IDM | 4 | 1,530 | 1 | 0,07 |

Ba cặp đổi thứ tự là constant-velocity/IDM, IDM/EgoMLP và IDM/PercMLP. IDM có L2 kém nhất vì dự báo luật của nó không bắt chước chính xác quỹ đạo người lái, nhưng phản hồi liên tục theo khoảng cách và tốc độ tương đối nên chỉ đâm 2/2.883 quỹ đạo.
EgoMLP có L2 tốt hơn constant-velocity (0,768 so với 1,265 m), nhưng chỉ nhìn lịch sử tốc độ bản thân; nó không biết xe dẫn đang giảm tốc hay khoảng cách tới xe trước. Vì vậy lỗi dự báo nhỏ trên trung bình có thể tích lũy thành va chạm trong vòng kín, nơi tỷ lệ va chạm trung bình của EgoMLP là 19,32%.

## Câu 5 · Một lần chạy chưa đủ
EgoMLP dao động bao nhiêu giữa các seed? Khoảng tin cậy Wilson nói gì? Nếu bạn chỉ chạy đúng một seed thì có thể báo kết quả nào, và vì sao điều đó nguy hiểm?

Qua 5 seed, EgoMLP có tỷ lệ va chạm từ 13,53% đến 28,13%, chênh 14,60 điểm phần trăm. Seed tốt nhất (4454) có CI Wilson 95% 12,33–14,82%; seed xấu nhất (4453) là 26,52–29,80%, hai khoảng không chồng lấp.
Nếu chỉ chạy một seed, tôi có thể báo con số gần 13,5% hoặc gần 28,1% tùy seed và đưa ra ấn tượng trái ngược về mức rủi ro. Khoảng tin cậy Wilson mô tả bất định của tỷ lệ trong từng lần chạy, còn dải 5 seed cho thấy thêm độ nhạy do khởi tạo huấn luyện.
Do đó một seed không đủ để kết luận mô hình ổn định hay đạt yêu cầu phát hành; cổng R5 cũng chặn EgoMLP vì độ rộng dải lớn hơn ngưỡng 1 điểm phần trăm.

## Câu 6 · Tác nhân phản ứng
Dán bảng "xe sau phát lại" và "xe sau phản ứng". `T` bạn hiệu chỉnh là bao nhiêu, xe sau chân thực đến mức nào so với xe không phản ứng? Vì sao IDM bị tính "va chạm giả" khi xe sau chỉ phát lại, và điều này làm đổi kết luận về bộ điều khiển ra sao?

Hiệu chỉnh được IDM xe sau với thời khoảng cách T = 1,4 s; khoảng cách Wasserstein giữa phân bố headway và người lái là 0,355, thấp hơn rõ rệt xe không phản ứng (0,843). Headway trung vị người lái là 1,854 s, nên IDM là xấp xỉ hành vi, không phải bản sao hoàn hảo.

| Bộ điều khiển | Va chạm bất kỳ: xe sau phát lại (%) | Va chạm bất kỳ: xe sau phản ứng (%) |
|---|---:|---:|
| constant-velocity | 77,63 | 77,63 |
| IDM | 75,74 | 7,44 |
| EgoMLP | 77,78 | 77,78 |
| PercMLP | 74,67 | 27,26 |
| PercMLP+ | 75,59 | 8,04 |

Ở IDM, 68,30% lưới ghi va chạm phía sau khi xe sau chỉ phát lại, nhưng con số này về 0% khi xe sau phản ứng. Xe phát lại tiếp tục chạy quỹ đạo cũ dù xe thử vừa phanh, nên nó tạo va chạm giả phía sau; kết luận IDM kém an toàn vì thế đổi mạnh khi dùng tác nhân có phản ứng.

## Câu 7 · Cổng phát hành và sửa
Quy tắc nào chặn PercMLP và nguyên nhân gốc là gì (dẫn số)? Bạn sửa bằng gì và vì sao nó chữa đúng nguyên nhân? Sau khi qua cổng, PercMLP+ có **tốt hơn** IDM không? Nó có nên được phát hành không, và bạn đề xuất làm gì trước khi quyết định?

PercMLP ban đầu bị chặn ở R2, R3, R4, R5 và R6. Trên log, cận trên Wilson ở R2 là 3,13% so với giới hạn 1,07%; trên lưới cut-in R4 là 19,81% vùng va chạm quy cho planner, và R6 là 27,26% va chạm trong thế giới có xe sau phản ứng. R4/R6 phản ánh tình huống cut-in thiếu trong log; R2 phản ánh sai số bắt chước nhỏ tích lũy khi tự lái.
Tôi bổ sung 20.000 kịch bản cut-in có nhãn IDM và chạy 2 vòng DAgger để lấy trạng thái mà chính mô hình đi vào rồi hỏi IDM dự báo ở các trạng thái đó. Sau sửa, vùng lỗi cut-in ở trễ 0,6 s giảm từ 19,81% xuống 0,59%; R2–R7 đều qua và gate cho PercMLP+ quyết định RELEASE.
Qua cổng nghĩa là đạt các ngưỡng không-kém-hơn đã đặt, không chứng minh tốt hơn IDM: tỷ lệ va chạm trung bình của PercMLP+ là 0,277% so với 0,069% của IDM, và PercMLP+ học từ IDM làm chuyên gia. Tôi chưa thay IDM; cần kiểm tra thêm trên tình huống và phần cứng độc lập, đặc biệt các ca ngoài phân bố.

## Câu 8 · Bài lab này không chứng minh được gì
Chọn **ít nhất ba** giới hạn quan trọng nhất đối với việc quyết định phát hành và giải thích vì sao mỗi cái làm kết luận của bạn kém chắc chắn. Điều gì cần làm tiếp để thu hẹp khoảng cách đến thế giới thật?

Thứ nhất, đây là mô phỏng dọc một chiều, không có cảm biến hay lỗi perception; vì thế nó không đo được cắt làn, cua, phanh mặt đường và sai số cảm biến của xe thật. Cần mở rộng sang mô hình đa làn, cảm biến và động lực học xe đã hiệu chuẩn.
Thứ hai, xe dẫn được phát lại từ log nên không phản ứng với xe thử; thí nghiệm xe sau cho thấy giả định tương tác này làm tỷ lệ va chạm của IDM đổi từ 75,74% xuống 7,44%. Cần kiểm chứng với nhiều tác nhân phản ứng và thử nghiệm kín có người giám sát.
Thứ ba, dữ liệu chỉ gồm 9.610 quỹ đạo Argoverse 2 đã khử trùng, 10 Hz, các đoạn dài 7–11 giây; việc tách theo quỹ đạo giảm rò rỉ nhưng không chứng minh khái quát sang thời tiết, đường sá hay đội xe khác. Cần đánh giá ngoài tập và thu thập dữ liệu đa điều kiện.
Cuối cùng, mô phỏng giả định phanh tối đa 9 m/s² và trễ cố định 0,6 s; 8,56% ô lưới cần phanh mạnh hơn giới hạn 99,9% quan sát ở người lái. Cần đo trễ và lực phanh thật, chạy kiểm thử HIL/closed-track, rồi mới cân nhắc triển khai từng bước.
