# Vì sao chọn lô này?

Trong 50 dòng đứng đầu `outputs/outputs/selection_round1.csv`, tôi ưu tiên 5 frame nếu chỉ có ngân sách rà 5 ảnh: `frame_0182.jpg` (score 0.9591), `frame_0369.jpg` (0.9324), `frame_0380.jpg` (0.9170), `frame_0326.jpg` (0.9155) và `frame_0331.jpg` (0.9154). Năm frame này đều có điểm cao, số xe quan sát và độ bất định đáng chú ý, đồng thời không quá gần nhau về thời điểm nên đủ đa dạng để hiệu quả học tập cao hơn.

Ba frame thuộc lô 12 ảnh mà model đã chọn và là ví dụ mạnh nhất là `frame_0182.jpg`, `frame_0369.jpg` và `frame_0380.jpg`. Ở ba frame này, model có nhiều xe ở điều kiện sáng tối, xe sát mép ảnh hoặc có xe lớn và xe trung bình cùng xuất hiện trong khung, nên sửa nhãn ở đây có khả năng giúp model học được các trường hợp khó hơn. Chúng cũng đồng thời phản ánh mức độ bất định cao trong `U` và `A`, nên rất phù hợp với chiến lược uncertainty sampling.

Một frame có điểm cao nhưng tôi không chọn là `frame_0372.jpg` (0.9101). Mặc dù điểm cao hơn một số frame đã chọn, nó rất gần với `frame_0369.jpg` về thời gian và cảnh, chỉ chênh nhau ~1.2 giây. Hai frame này gần như cùng một cảnh, nên chọn cả hai sẽ gây tốn thời gian mà không mang lại nhiều thêm thông tin cho mô hình. Đây là trường hợp điển hình của việc cần xét `MIN_GAP_S` và tránh ảnh gần trùng.

Việc chọn lô này không chứng minh mô hình sẽ tốt hơn ngay lập tức, vì nó chỉ là lựa chọn theo điểm bất định và chi phí rà nhãn. Một ảnh có điểm cao không đồng nghĩa với việc sửa xong sẽ cải thiện AP50 mạnh, đặc biệt với các xe rất xa, xe bị che mép ảnh hoặc xe nhỏ. Do đó, mục tiêu của lô này là tăng độ đa dạng và giảm sai sót rõ ràng của pre-label, chứ không phải “đánh bẫy” số điểm theo một frame duy nhất.
