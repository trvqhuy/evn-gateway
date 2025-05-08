# 🚀 EVN Gateway 3.0 (Sắp ra mắt) – "xịn sò" hơn

![EVN Gateway 3.0](https://img.shields.io/badge/EVN--Gateway-3.0-blueviolet?style=for-the-badge)

Chào mọi người,

Sau một thời gian duy trì các phiên bản trước của EVN Gateway, mình nhận ra hệ thống dần trở nên không ổn định – đặc biệt là từ khi các chi nhánh EVN bắt đầu siết chặt bảo mật bằng CAPTCHA.

---

## ❌ Những vấn đề thường gặp ở các bản cũ

- 🔐 CAPTCHA mới của các chi nhánh EVN (nhất là NPC và SPC) khiến bot gần như không thể login tự động nữa.
- 🕒 Website EVN hay **chậm, lỗi, hoặc logout bất ngờ** dù vừa login xong.
- 📉 Integration thường xuyên không cập nhật được dữ liệu mới do vấn đề của API request.

---

## 🔍 Mình đang cần gì để giải quyết những vấn đề trên?

Nút nghẽn lớn nhất của toàn bộ dự án chính là CAPTCHA EVN. Vì vậy, mình cần một giải pháp:

- ✅ **Gọn nhẹ**, deploy được trên **Render Free Tier (1 vCPU + ~512MB RAM)**
- ✅ **Độ chính xác cao** (>80%) với CAPTCHA EVN vốn bị méo, dính chữ, nhiễu hạt
- ✅ **Tốc độ xử lý nhanh**, đủ dùng trong thực tế
- ✅ **Scale tốt cho vài ngàn user**, mà **không phải trả tiền cloud**

Nghe thì có vẻ đơn giản, nhưng thực tế **rất khó** – vì:
- CAPTCHA EVN được thiết kế để chống bot, ký tự thường **biến dạng**, **dính liền nhau**, **nhiễu**
- Deploy 1 API **miễn phí** cho hàng trăm, thậm chí hàng ngàn người sử dụng là 1 thử thách lớn hơn.

---

## 🔬 Mình đã thử gì?

### 1. ❌ OCR truyền thống thất bại trên các nền tảng tài nguyên thấp

Mình từng thử nhiều thư viện OCR phổ biến nhưng đều không đủ chính xác hoặc quá nặng để chạy trên các nền tảng như Render/Vercel/Railway:

| OCR Engine     | Lý do không dùng được trên Render/Vercel Free Tier |
|----------------|-----------------------------------------------------|
| **Tesseract**  | Phải build từ source, không chạy được trên môi trường read-only. Độ chính xác cực thấp với CAPTCHA EVN. |
| **EasyOCR**    | Phụ thuộc PyTorch (~700MB), dễ vượt giới hạn RAM 512MB. Dù build được thì tốc độ predict rất chậm. |
| **PaddleOCR**  | Cần Docker + nhiều lib hệ thống. Quá nặng để deploy trên Vercel hoặc Render Free. |
| **TrOCR (HF)** | Model ~1.5GB, không thể chạy trên gói free. Chạy local thì ổn nhưng không scale được. |

👉 **Kết luận**: OCR truyền thống không thể giải được CAPTCHA EVN **trong điều kiện tài nguyên giới hạn**.

---

### 2. 🔍 Nghiên cứu giải pháp bằng Deep Learning

Mình chuyển hướng sang Deep Learning sau khi nhận ra chỉ có cách này mới đủ "thông minh" để học được các mẫu CAPTCHA méo mó từ EVN.

#### Mô hình được chọn:
- 🧠 **CRNN**: kết hợp CNN (trích đặc trưng ảnh) + BiLSTM (xử lý chuỗi)
- 🌀 **CTC Loss**: cho phép model học nhận diện chuỗi ký tự **mà không cần align từng ký tự thủ công**

#### Các thử nghiệm và tối ưu mình thực hiện:
- So sánh kiến trúc: `GRU` vs `BiLSTM`, chọn BiLSTM vì accuracy tốt hơn
- So sánh framework: `PyTorch` vs `ONNXRuntime`, dùng ONNX để giảm size và tăng tốc độ inference
- Xây dựng full pipeline:
  - 📸 Preprocessing ảnh: CLAHE, thresholding, resize
  - 🧪 Huấn luyện mô hình từ dataset CAPTCHA EVN (~15,000 ảnh)
  - 📦 Convert sang ONNX: mô hình cuối cùng chỉ ~**2MB**
 

#### Và mình đã thành công
- Sau hàng tháng nghiên cứu, thử và sai, mình đã thiết kế được 1 API miễn phí hoàn toàn đạt được những đầu bài phía trên.
- Tuy nhiên, mình sẽ chỉ nêu những kết quả mình đạt được ở repo này, nếu có dịp thì mình sẽ hướng dẫn chi tiết sau, bởi vì khối lượng công việc khá lớn không thể gom gọn vào 1 project, từ cách mình tạo ra 1 dataset hơn 15,000 CAPTCHAs cho mỗi chi nhánh EVN, cho đến cách mình thiết kế, train, validate, sử dụng CRNN model, và mục tiêu cuối cùng, là thiết kế 1 low-cost backend archiecture cho tối đa 1,000 users.

---

## ✅ Kết quả đạt được

Sau tối ưu:

- ⚡ **Inference trung bình < 1.0s** - deploy trên **Render Free Tier (1 vCPU + 512MB RAM)**
- 🎯 **Độ chính xác đạt 85–99%**, tuỳ độ méo của ảnh
- 💾 Mô hình chỉ ~2MB, deploy cực nhẹ
- 🔁 Scale tốt cho hàng ngàn người dùng (nếu sử dụng các kỹ thuật backend-side hợp lý)

> ✨ Từ một bài toán tưởng như không thể, mình đã tạo ra một hệ thống OCR bằng Deep Learning có thể chạy mượt trên cloud miễn phí và nhận diện được CAPTCHA phức tạp.

---

## 💡 Chia 1 bài toán lớn thành 2 bài toán nhỏ: CAPTCHA Resolver và Gateway

Để tối ưu hiệu năng và maintain dễ hơn, mình chia toàn bộ dự án thành hai phần:

---

### 📦 [`evn-captcha-resolver`](https://crnn-captcha-resolver.onrender.com/) – **Đã xong/đang test**

- REST API dùng Deep Learning (CRNN + ONNX) để giải mã CAPTCHA EVN
- Train trên dataset crawl thủ công từ nhiều chi nhánh EVN
- Mỗi ảnh xử lý ~600–1500ms, độ chính xác từ 85% trở lên
- Deploy trên Render Free Tier, hoàn toàn miễn phí

---

### 🔄 `evn-gateway` – **Sắp phát triển**

- Đóng vai trò là client-side integration (Home Assistant, webhook,...)
- Tự động login, fetch dữ liệu hoá đơn, chỉ số, biểu đồ,...
- Kết nối trực tiếp với CAPTCHA Resolver để vượt qua bước xác thực
- Có thể mở rộng dashboard theo dõi trạng thái hệ thống EVN

---

## 🧪 Dùng thử CAPTCHA API miễn phí

> 👉 https://crnn-captcha-resolver.onrender.com/

- Đăng nhập bằng GitHub để dùng thử
- Giao diện đơn giản: upload ảnh hoặc chọn ảnh mẫu → nhận kết quả text
- Mình tạm mở public để test – rất mong nhận được feedback & ảnh fail từ cộng đồng

---

## 📌 Roadmap tiếp theo

- [x] ✅ Hoàn thiện CAPTCHA resolver (v1)
- [ ] 🔄 Triển khai `evn-gateway` theo kiến trúc mới
- [ ] 📊 Tích hợp dashboard Home Assistant (hoá đơn, chỉ số,...)
- [ ] 🚀 Thêm tính năng: cảnh báo khi có hoá đơn mới qua Zalo/Email

---
## 🙏 Góp ý giúp mình

Mình rất cần feedback từ các bạn để tiếp tục hoàn thiện:

- CAPTCHA nào đang fail nhiều nhất?
- Bạn đang dùng integration EVN nào (NPC, SPC,...)? Gặp lỗi gì thường xuyên?
- Bạn cảm thấy sao về việc tích hợp API thông báo hoá đơn mới qua Zalo/Email?

👉 **Hãy tham gia thảo luận tại đây**:  
[📢 Góp ý trước khi release chính thức EVN Gateway 3.0](https://github.com/trvqhuy/evn-gateway/discussions/1)

> 🗣 Đây là thời điểm **trước khi chính thức release** – mọi ý kiến đóng góp lúc này sẽ **ảnh hưởng trực tiếp đến cách phiên bản mới được thiết kế**.  
> Đừng ngần ngại chia sẻ: càng chi tiết, càng tốt! Mình đọc hết tất cả phản hồi và sẽ tổng hợp lại trước khi chốt lộ trình bản chính thức.

Cảm ơn mọi người đã đọc đến đây!  
Nếu thấy dự án hữu ích, **hãy star repo**, **follow mình**, để nhận thông báo khi EVN Gateway v3.0 chính thức ra mắt.

> 🛠 Dự án làm bằng **PyTorch**, **OpenCV**, đam mê và hàng chục lần fix bug mỗi đêm 😅
