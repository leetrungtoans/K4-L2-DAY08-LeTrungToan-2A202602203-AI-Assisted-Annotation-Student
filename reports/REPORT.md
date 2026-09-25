# Báo cáo Lab Ngày 08: Học chủ động cho bộ phát hiện xe

Họ và tên: Lê Trung Toán

Công cụ gán nhãn đã dùng: CVAT Docker local

## 1. Dữ liệu và cách chia tập

Tập pool và tập test được chia theo thời gian có vùng đệm, không chia ngẫu nhiên, vì camera đứng ở một vị trí cố định và cùng một xe có thể xuất hiện trong nhiều khung gần nhau. Nếu chia ngẫu nhiên, khung từ cùng một đoạn video sẽ rơi vào cả tập train và test, làm số đo AP50 bị phóng đại và không phản ánh khả năng tổng quát hóa thật sự. Kỹ thuật này giúp giảm rò rỉ dữ liệu theo thời gian và giữ tính công bằng của đánh giá.

## 2. Mô hình khởi đầu lạnh (cold start)

Dòng vòng 0 trong `reports/rounds_table.md` là: AP50 = 0.7714. Từ `outputs/outputs/compare_round0.jpg` và `outputs/outputs/metrics_round0.json`, mô hình cold start làm tốt trên xe lớn và xe vừa, nhưng báo cáo recall theo kích thước rõ ràng cho thấy xe nhỏ còn rất yếu: recall nhỏ = 0.1818, medium = 0.5473, large = 0.5610. Cụ thể, xe ở góc mép ảnh, xe ở xa hoặc chỉ còn một phần thân xe thường bị bỏ sót. Ví dụ, các xe ở mép màn hình và các xe gần chân cầu chỉ còn đèn hoặc một nửa thân dễ bị model bỏ qua. Đây là trường hợp cần người rà lại nhãn tham chiếu trước khi kết luận model sai hoàn toàn, vì nhãn test cũng là nhãn do model gán và chưa được người kiểm chặt chẽ.

## 3. Chiến lược chọn mẫu

Chiến lược chọn ảnh là dựa trên điểm xếp hạng theo dạng `score = W_U·U + W_A·A + W_D·D`, trong đó `U` đại diện cho độ bất định, `A` là khả năng ảnh có nhiều box mơ hồ hoặc phức tạp, còn `D` thể hiện độ khó và độ lệch nhãn. `MIN_GAP_S` được dùng để tránh chọn các ảnh quá gần nhau về thời gian hoặc cảnh, vì nếu hai frame gần như cùng một cảnh thì việc sửa cả hai không mang lại thông tin mới nhiều bằng một ảnh khác. Đó là lý do tôi ưu tiên `frame_0182.jpg`, `frame_0369.jpg`, `frame_0380.jpg`, `frame_0326.jpg` và `frame_0331.jpg` trong 5 khung đầu tiên, đồng thời không chọn `frame_0372.jpg` dù điểm cao, vì nó gần trùng với `frame_0369.jpg`.

## 4. Các vòng học chủ động (active learning)

Bảng tổng kết:

| round | train images | train boxes | AP50 | thay đổi |
| ---: | ---: | ---: | ---: | ---: |
| 0 | 0 | 0 | 0.7714 | baseline |
| 1 | 12 | 276 | 0.2942 | -0.4772 |

Trong vòng 1, tôi đã sửa pre-label trên 12 ảnh. Theo `outputs/round1_diff.md`, model đề xuất 169 box, sau khi sửa còn 276 box: accepted 111, edited 43, deleted 15, added 122. Hai loại sửa quan trọng nhất là xóa các box sai (FP) và thêm các box thiếu (FN), đặc biệt ở những xe rất xa, mép ảnh hoặc bị che mất một phần thân. So với cold start, AP50 giảm mạnh từ 0.7714 xuống 0.2942, cho thấy việc sửa nhãn theo cách này chưa cải thiện số đo trên tập test; tuy nhiên, nó giúp cải thiện độ hiểu biết về các trường hợp khó và làm rõ lỗi của pre-label.

Dựa trên `BLIND_SCAN.md`, `REVIEW_LOG.csv` và `outputs/round1_diff.md`, tôi thấy rõ ba lớp thông tin khác nhau: quan sát độc lập trước khi xem pre-label, sửa nhãn sau khi thấy lỗi thực tế, và kết quả mô hình sau train. Ví dụ, trong `frame_0099.jpg`, tôi đã nhìn thấy xe ở góc dưới phải bị mép ảnh cắt mất và thêm box mới; trong `frame_0107.jpg`, tôi xóa khung sáng trên mặt đường không phải xe; trong `frame_0369.jpg`, tôi chỉnh khung AI để sát thân xe hơn. Những trường hợp này cho thấy các lỗi pre-label không phải ngẫu nhiên mà tập trung ở các xe xa, xe mép và dạng phản quang không phải xe.

## 5. Kết luận và giới hạn

So với cold start, vòng 1 không cải thiện AP50, ngược lại giảm mạnh. Vì vậy, tôi sẽ dừng hoặc tạm dừng thêm vòng mới cho đến khi kiểm tra lại các trường hợp bằng cách xem kỹ các ảnh sai và xác định liệu lỗi là do nhãn, do chọn ảnh, hay do mô hình. Hai trường hợp còn yếu rõ nhất là xe quá nhỏ ở xa và xe bị cắt mép ảnh; đây là các trường hợp khó, và nếu chọn thêm chúng, chi phí rà nhãn tăng nhưng không chắc sẽ giúp cải thiện mô hình. Hơn nữa, tập test chỉ có 20 ảnh và nhãn tham chiếu do mô hình tạo chưa được người rà từng box, nên số đo này chỉ phản ánh mức độ khớp với tham chiếu chứ không phải “chân lý tuyệt đối”. Nếu AP50 giảm, bước đầu tiên nên làm là xem lại các ảnh có lỗi lớn, chẳng hạn `frame_0099.jpg`, `frame_0107.jpg`, `frame_0369.jpg`, trước khi train thêm bất kỳ vòng nào nữa.
