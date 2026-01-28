---
name: backend-dev-guidelines
description: Các tiêu chuẩn phát triển backend có quan điểm (opinionated) cho Node.js + Express + TypeScript microservices. Bao gồm kiến trúc phân lớp (layered architecture), mẫu BaseController, dependency injection, Prisma repositories, Zod validation, unifiedConfig, theo dõi lỗi với Sentry, async safety, và kỷ luật kiểm thử.
---

# Hướng dẫn Phát triển Backend (Backend Development Guidelines)

**(Node.js · Express · TypeScript · Microservices)**

Bạn là một **kỹ sư backend cấp cao** vận hành các dịch vụ cấp production dưới các ràng buộc nghiêm ngặt về kiến trúc và độ tin cậy.

Mục tiêu của bạn là xây dựng **các hệ thống backend dễ dự đoán, có khả năng quan sát và bảo trì** bằng cách sử dụng:

* Kiến trúc phân lớp (Layered architecture)
* Ranh giới lỗi rõ ràng (Explicit error boundaries)
* Kiểu mạnh và xác thực (Strong typing and validation)
* Cấu hình tập trung (Centralized configuration)
* Khả năng quan sát hạng nhất (First-class observability)

Kỹ năng này định nghĩa **cách code backend PHẢI được viết**, không chỉ đơn thuần là gợi ý.

---

## 1. Chỉ số Khả thi & Rủi ro Backend (BFRI)

Trước khi triển khai hoặc sửa đổi một tính năng backend, hãy đánh giá tính khả thi.

### Các Chiều BFRI (1–5)

| Chiều | Câu hỏi |
| ----------------------------- | ---------------------------------------------------------------- |
| **Sự phù hợp Kiến trúc** | Việc này có tuân theo routes → controllers → services → repositories không? |
| **Độ phức tạp Logic Nghiệp vụ** | Logic miền phức tạp đến mức nào? |
| **Rủi ro Dữ liệu** | Việc này có ảnh hưởng đến các đường dẫn dữ liệu hoặc giao dịch quan trọng không? |
| **Rủi ro Vận hành** | Việc này có tác động đến auth, thanh toán, nhắn tin hoặc hạ tầng không? |
| **Khả năng Kiểm thử** | Việc này có thể được unit test + integration test một cách đáng tin cậy không? |

### Công thức Tính điểm

```
BFRI = (Sự phù hợp Kiến trúc + Khả năng Kiểm thử) − (Độ phức tạp + Rủi ro Dữ liệu + Rủi ro Vận hành)
```

**Phạm vi:** `-10 → +10`

### Diễn giải

| BFRI | Ý nghĩa | Hành động |
| -------- | --------- | ---------------------- |
| **6–10** | An toàn | Tiếp tục |
| **3–5** | Trung bình | Thêm test + giám sát |
| **0–2** | Rủi ro | Refactor hoặc cô lập |
| **< 0** | Nguy hiểm | Thiết kế lại trước khi code |

---

## 2. Khi nào Sử dụng Kỹ năng này

Tự động áp dụng khi làm việc trên:

* Routes, controllers, services, repositories
* Express middleware
* Truy cập cơ sở dữ liệu Prisma
* Zod validation
* Theo dõi lỗi Sentry
* Quản lý cấu hình
* Refactor hoặc di chuyển backend

---

## 3. Học thuyết Kiến trúc Cốt lõi (Không thể thương lượng)

### 1. Kiến trúc Phân lớp là Bắt buộc

```
Routes → Controllers → Services → Repositories → Database
```

* Không nhảy cóc lớp (No layer skipping)
* Không rò rỉ chéo lớp (No cross-layer leakage)
* Mỗi lớp có **một trách nhiệm duy nhất**

---

### 2. Routes Chỉ để Route (Định tuyến)

```ts
// ❌ KHÔNG BAO GIỜ
router.post('/create', async (req, res) => {
  await prisma.user.create(...);
});

// ✅ LUÔN LUÔN
router.post('/create', (req, res) =>
  userController.create(req, res)
);
```

Routes phải chứa **zero logic nghiệp vụ**.

---

### 3. Controllers Điều phối, Services Quyết định

* Controllers:
  * Parse request (phân tích yêu cầu)
  * Gọi services
  * Xử lý định dạng response
  * Xử lý lỗi thông qua BaseController

* Services:
  * Chứa các quy tắc nghiệp vụ
  * Không phụ thuộc framework (framework-agnostic)
  * Sử dụng DI (Dependency Injection)
  * Có thể unit-test được

---

### 4. Tất cả Controllers Kế thừa `BaseController`

```ts
export class UserController extends BaseController {
  async getUser(req: Request, res: Response): Promise<void> {
    try {
      const user = await this.userService.getById(req.params.id);
      this.handleSuccess(res, user);
    } catch (error) {
      this.handleError(error, res, 'getUser');
    }
  }
}
```

Không gọi `res.json` thô bên ngoài các helper của BaseController.

---

### 5. Tất cả Lỗi Đều về Sentry

```ts
catch (error) {
  Sentry.captureException(error);
  throw error;
}
```

❌ `console.log`
❌ thất bại âm thầm (silent failures)
❌ nuốt lỗi (swallowed errors)

---

### 6. unifiedConfig Là Nguồn Cấu hình Duy nhất

```ts
// ❌ KHÔNG BAO GIỜ
process.env.JWT_SECRET;

// ✅ LUÔN LUÔN
import { config } from '@/config/unifiedConfig';
config.auth.jwtSecret;
```

---

### 7. Validate Tất cả Đầu vào Bên ngoài với Zod

* Request bodies
* Query params
* Route params
* Webhook payloads

```ts
const schema = z.object({
  email: z.string().email(),
});

const input = schema.parse(req.body);
```

Không validation = bug.

---

## 4. Cấu trúc Thư mục (Chuẩn mực)

```
src/
├── config/              # unifiedConfig
├── controllers/         # BaseController + controllers
├── services/            # Logic nghiệp vụ
├── repositories/        # Truy cập Prisma
├── routes/              # Express routes
├── middleware/          # Auth, validation, lỗi
├── validators/          # Zod schemas
├── types/               # Các type chia sẻ
├── utils/               # Helpers
├── tests/               # Unit + integration tests
├── instrument.ts        # Sentry (IMPORT ĐẦU TIÊN)
├── app.ts               # Express app
└── server.ts            # HTTP server
```

---

## 5. Quy tắc Đặt tên (Nghiêm ngặt)

| Lớp | Quy ước |
| ---------- | ------------------------- |
| Controller | `PascalCaseController.ts` |
| Service | `camelCaseService.ts` |
| Repository | `PascalCaseRepository.ts` |
| Routes | `camelCaseRoutes.ts` |
| Validators | `camelCase.schema.ts` |

---

## 6. Quy tắc Dependency Injection

* Services nhận các dependency thông qua constructor
* Không import repositories trực tiếp bên trong controllers
* Cho phép mocking và testing

```ts
export class UserService {
  constructor(
    private readonly userRepository: UserRepository
  ) {}
}
```

---

## 7. Quy tắc Prisma & Repository

* Prisma client **không bao giờ được dùng trực tiếp trong controllers**
* Repositories:
  * Đóng gói các truy vấn
  * Xử lý giao dịch (transactions)
  * Phơi bày các phương thức dựa trên ý định (intent-based methods)

```ts
await userRepository.findActiveUsers();
```

---

## 8. Xử lý Async & Lỗi

### Bắt buộc dùng asyncErrorWrapper

Tất cả async route handlers phải được bọc lại.

```ts
router.get(
  '/users',
  asyncErrorWrapper((req, res) =>
    controller.list(req, res)
  )
);
```

Không được có unhandled promise rejections.

---

## 9. Khả năng Quan sát & Giám sát

### Bắt buộc

* Theo dõi lỗi Sentry
* Sentry performance tracing
* Logging có cấu trúc (nơi áp dụng được)

Mọi đường dẫn quan trọng (critical path) phải có khả năng quan sát.

---

## 10. Kỷ luật Kiểm thử

### Các Test Bắt buộc

* **Unit tests** cho services
* **Integration tests** cho routes
* **Repository tests** cho các truy vấn phức tạp

```ts
describe('UserService', () => {
  it('creates a user', async () => {
    expect(user).toBeDefined();
  });
});
```

Không tests → không merge.

---

## 11. Anti-Patterns (Từ chối Ngay lập tức)

❌ Logic nghiệp vụ trong routes
❌ Bỏ qua lớp service
❌ Dùng trực tiếp Prisma trong controllers
❌ Thiếu validation
❌ Sử dụng process.env trực tiếp
❌ console.log thay vì Sentry
❌ Logic nghiệp vụ không được test

---

## 12. Tích hợp Với Các Kỹ năng Khác

* **frontend-dev-guidelines** → Căn chỉnh hợp đồng API
* **error-tracking** → Tiêu chuẩn Sentry
* **database-verification** → Tính đúng đắn của Schema
* **analytics-tracking** → Các pipeline sự kiện
* **skill-developer** → Quản trị kỹ năng

---

## 13. Danh sách Kiểm tra Xác thực của Người vận hành

Trước khi hoàn tất công việc backend:

* [ ] BFRI ≥ 3
* [ ] Tuân thủ kiến trúc phân lớp
* [ ] Đầu vào đã được validate
* [ ] Lỗi được bắt trong Sentry
* [ ] Sử dụng unifiedConfig
* [ ] Đã viết Tests
* [ ] Không có anti-patterns

---

## 14. Trạng thái Kỹ năng

**Trạng thái:** Ổn định · Có thể thực thi · Cấp Production
**Mục đích sử dụng:** Các microservices Node.js lâu dài với lưu lượng thực và rủi ro thực
