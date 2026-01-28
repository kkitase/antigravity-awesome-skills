---
name: code-review-excellence
description: Làm chủ các thực hành review code hiệu quả để cung cấp phản hồi mang tính xây dựng, bắt lỗi sớm và thúc đẩy chia sẻ kiến thức trong khi duy trì tinh thần đồng đội. Sử dụng khi xem xét các pull request, thiết lập tiêu chuẩn review, hoặc hướng dẫn (mentoring) các developer.
---

# Review Code Xuất sắc (Code Review Excellence)

Biến việc review code từ "gác cổng" (gatekeeping) thành chia sẻ kiến thức thông qua phản hồi mang tính xây dựng, phân tích có hệ thống và cải tiến cộng tác.

## Sử dụng kỹ năng này khi

- Review pull requests và các thay đổi code
- Thiết lập tiêu chuẩn review code
- Hướng dẫn developers thông qua phản hồi review
- Kiểm toán (audit) về tính đúng đắn, bảo mật hoặc hiệu năng

## Không sử dụng kỹ năng này khi

- Không có thay đổi code nào để review
- Nhiệm vụ chỉ là thảo luận thiết kế mà không có code
- Bạn cần thực hiện sửa lỗi (implement fixes) thay vì review

## Hướng dẫn

- Đọc bối cảnh, yêu cầu và các tín hiệu kiểm thử (test signals) trước.
- Review về tính đúng đắn, bảo mật, hiệu năng và khả năng bảo trì.
- Cung cấp phản hồi có thể hành động (actionable) với mức độ nghiêm trọng và lý do.
- Đặt câu hỏi làm rõ khi ý định chưa rõ ràng.
- Nếu cần danh sách kiểm tra chi tiết, hãy mở `resources/implementation-playbook.md`.

## Định dạng Đầu ra

- Tóm tắt cấp cao về các phát hiện
- Các vấn đề được nhóm theo mức độ nghiêm trọng (chặn đứng - blocking, quan trọng, nhỏ)
- Đề xuất và câu hỏi
- Ghi chú về kiểm thử và độ bao phủ (coverage)

## Tài nguyên

- `resources/implementation-playbook.md` cho các mẫu review chi tiết và templates.
