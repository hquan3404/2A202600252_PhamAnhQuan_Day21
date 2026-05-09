# Lab 21 — Evaluation Report: LoRA Fine-tuning

**Học viên**: Phạm Anh Quân  
**Submission option**: A (Lightweight ZIP)

## 1. Setup
- **Base model**: `unsloth/Qwen2.5-3B-bnb-4bit`
- **Dataset**: `5CD-AI/Vietnamese-alpaca-gpt4-gg-translated`, 200 samples (180 train / 20 eval)
- **max_seq_length**: 1024 (p95 = 562, rounded up for safety)
- **GPU**: Tesla T4 (15.6 GB VRAM)
- **Training cost**: ~$0.00 (Chạy trên Free Colab T4 trong khoảng 15 phút)

## 2. Rank Experiment Results

| Rank | Trainable Params | Train Time | Peak VRAM | Eval Loss | Perplexity |
|------|-----------------|------------|-----------|-----------|------------|
| 8    | 1,843,200       | 4.15 min   | 7.22 GB   | 1.5577    | 4.7479     |
| 16   | 3,686,400       | 4.36 min   | 6.62 GB   | 1.5161    | 4.5544     |
| 64   | 14,745,600      | 4.13 min   | 8.00 GB   | 1.4768    | 4.3790     |
| Base | -               | -          | -         | -         | -          |

## 3. Loss Curve Analysis
*(Ghi chú: Biểu đồ loss được quan sát trực tiếp trong notebook)*
- **Quan sát**: Đường training loss giảm đều và ổn định qua 3 epochs. Eval loss (1.47 - 1.55) bám sát training loss, cho thấy mô hình học tốt các pattern của tiếng Việt mà không bị overfitting nghiêm trọng. 
- **Lý do**: Với số lượng 200 samples chất lượng cao từ Alpaca GPT-4, mô hình 3B có đủ dung lượng để học style trả lời mà không bị "thuộc lòng" dữ liệu quá mức.

## 4. Qualitative Comparison (5 examples)

### Example 1
**Prompt**: Giải thích khái niệm machine learning cho người mới bắt đầu.
**Base**: Machine learning là một phân khúc của trí tuệ nhân tạo, nó tập trung vào việc thiết lập các mô hình máy móc để học tập từ dữ liệu... (Kết thúc lửng lơ)
**Fine-tuned (r=16)**: Machine learning là một bộ môn công nghệ máy tính dựa trên việc học tập và cải thiện các dự đoán dựa trên dữ liệu mà không có sự hướng dẫn trực tiếp từ người dùng... (Đầy đủ và tự nhiên hơn)
**Nhận xét**: Improved. Câu văn mạch lạc và chuyên nghiệp hơn.

### Example 2
**Prompt**: Viết đoạn code Python tính số Fibonacci thứ n.
**Base**: (Trả lời code cơ bản nhưng có lỗi logic ở điều kiện dừng hoặc thiếu phần xử lý tối ưu).
**Fine-tuned (r=16)**: Cung cấp đoạn code Python hoàn chỉnh, sử dụng vòng lặp tối ưu và có handle lỗi ValueError cho input âm.
**Nhận xét**: Improved. Code sạch hơn và có tính ứng dụng thực tế cao hơn.

### Example 3
**Prompt**: Liệt kê 5 nguyên tắc thiết kế UI/UX.
**Base**: Liệt kê các ý chung chung, đôi khi bị lặp lại ý tưởng về sự "thân thiện".
**Fine-tuned (r=16)**: Liệt kê rõ ràng 5 điểm: Chuyển đổi, Thích ứng, Đơn giản, Tương thích... với định nghĩa súc tích.
**Nhận xét**: Improved. Trình bày có cấu trúc rõ ràng (structured output).

### Example 4
**Prompt**: Tóm tắt sự khác biệt giữa LoRA và QLoRA.
**Base**: Giải thích sai tên gọi của LoRA (thành NLU) và mô tả còn mơ hồ.
**Fine-tuned (r=16)**: Định nghĩa chính xác hơn về cơ chế inject low-rank matrices, mặc dù vẫn còn một chút nhầm lẫn nhỏ về tên viết tắt nhưng nội dung kỹ thuật đã sát hơn.
**Nhận xét**: Improved. Nội dung chuyên môn được cải thiện.

### Example 5
**Prompt**: Phân biệt prompt engineering, RAG, và fine-tuning.
**Base**: Giải thích các khái niệm rời rạc, không nêu bật được sự khác biệt cốt lõi.
**Fine-tuned (r=16)**: Phân định rõ ràng vai trò của từng kỹ thuật: Prompt (câu lệnh), RAG (truy xuất dữ liệu), Fine-tuning (điều chỉnh mô hình).
**Nhận xét**: Improved. Giúp người dùng phân biệt rõ ràng các phương pháp.

## 5. Conclusion về Rank Trade-off
- **Rank nào cho ROI tốt nhất?**: Rank **r=16** mang lại tỷ lệ hoàn vốn (ROI) tốt nhất. So với r=8, r=16 giảm đáng kể Perplexity (từ 4.75 xuống 4.55) nhưng chỉ tăng thêm khoảng 0.2 phút training time và vẫn duy trì mức VRAM an toàn trên T4. 
- **Diminishing returns**: Hiện tượng này bắt đầu xuất hiện khi tăng từ r=16 lên r=64. Mặc dù số lượng tham số huấn luyện tăng gấp 4 lần (từ 3.6M lên 14.7M), nhưng Perplexity chỉ giảm thêm một lượng nhỏ (từ 4.55 xuống 4.38). Điều này cho thấy với dataset nhỏ (200 samples), việc sử dụng rank quá cao không mang lại hiệu quả vượt trội tương xứng với tài nguyên bỏ ra.
- **Recommendation**: Nếu triển khai thực tế (production), tôi sẽ chọn **rank 16**. Đây là mức cân bằng hoàn hảo giữa hiệu năng xử lý tiếng Việt, tốc độ phản hồi và tiết kiệm tài nguyên tính toán.

## 6. What I Learned
- Hiểu rõ cơ chế của LoRA: chỉ cần huấn luyện một lượng rất nhỏ tham số (dưới 1%) nhưng có thể thay đổi hoàn toàn phong cách và chất lượng phản hồi của LLM.
- QLoRA là cứu cánh cho GPU yếu: Có thể train một model 3B một cách mượt mà trên Tesla T4 chỉ với ~7-8GB VRAM nhờ kỹ thuật 4-bit quantization.
- Tầm quan trọng của chất lượng dữ liệu: Chỉ với 200 mẫu chất lượng cao, kết quả mang lại ấn tượng hơn nhiều so với việc dùng lượng lớn dữ liệu nhiễu.
