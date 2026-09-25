# Bảng tổng kết các vòng

| round | train images | train boxes | AP50 | thay đổi so với vòng 0 | ghi chú |
| ---: | ---: | ---: | ---: | ---: | --- |
| 0 | 0 | 0 | 0.7714 | baseline | cold start |
| 1 | 12 | 276 | 0.2942 | -0.4772 | fine-tune sau sửa pre-label |

Dữ liệu lấy từ `outputs/outputs/metrics_round0.json` và `outputs/outputs/metrics_round1.json`.
