# Subagents — Ghi chú

> Bài học: [Tiếng Việt](../vi/04-subagents/README.md) · [English](../04-subagents/README.md)
>
> Trạng thái: 🟨 Đang học · Ngày học: 2026-11-04

## Ý chính cần nhớ

Nguồn: bài viết [Building agents: when to fold subagents into your main agent loop](https://claude.com/resources/articles/building-agents-when-to-fold-subagents-into-your-main-agent-loop) (Diane Sarkis, 22/10/2026).

- **Thông điệp chính:** thường chỉ cần **một agent có skills + tools**, không cần cả một đội subagent. Cách này thường cho chất lượng tốt hơn, **rẻ hơn và nhanh hơn**. Một khách hàng đo được tốc độ chạy nhanh hơn khoảng 2 lần.

```text
TRƯỚC (multi-agent)                     SAU (single loop)

  User                                    User
   │                                       │
 Router ──► Planner                     ┌──▼─────────────────┐
   │                                    │   1 Agent chính    │
   ├─► Agent Đơn hàng ─┐                │   (1 context)      │
   ├─► Agent Vận chuyển├─► Tổng hợp     │ skills: hoàn tiền, │
   └─► Agent Hoàn tiền ┘                │   vận chuyển...    │
                                        │ tools: tra đơn...  │
  (mỗi lần chuyển giao = mất thông tin) └────────────────────┘
```

- **Chỉ dùng subagent khi có một trong ba dấu hiệu:**
  1. Agent chính kém đi thấy rõ vì context quá dài.
  2. Cần chạy **song song** nhiều việc, mỗi việc dùng context mới và bỏ đi sau khi xong.
  3. Agent chọn sai tool, hoặc system prompt có chỉ dẫn mâu thuẫn không gỡ được.
- **Nếu tách thì tách theo context, không tách theo loại vấn đề.** Mỗi subagent nhận việc ít phụ thuộc phần còn lại và trả về kết quả gọn. Hợp lý: nghiên cứu độc lập, component có interface rõ, kiểm tra kiểu hộp đen (verifier).
- **Progressive disclosure:** ban đầu agent chỉ thấy mô tả ngắn của skills/tools, khi cần mới nạp đầy đủ. Nhờ vậy context không phình, tức là mất lý do chính để tách subagent. Anthropic đã cắt **hơn 80%** system prompt của Claude Code nhờ cách này.
- **Ví dụ:** khách hỏi *"Đơn bao giờ tới, có được hoàn tiền không?"*
  - Multi-agent: Router → Agent Vận chuyển → Agent Hoàn tiền → tổng hợp. Có 2 lần chuyển giao, mỗi lần mất một phần thông tin.
  - Single loop: agent gọi song song `tra_don()` + `tra_van_chuyen()`, nạp skill "chính sách hoàn tiền", rồi trả lời với toàn bộ dữ kiện trong **cùng một context**.
- **Chia vai trò:**

| Thành phần | Dùng cho | Ví dụ |
|---|---|---|
| Tools (MCP) | Hành động, dữ liệu trực tiếp, ràng buộc cứng, phân quyền | `tra_don`, chặn thao tác khi thiếu điều kiện |
| Skills | Cách một team làm việc: quy trình, quy tắc | "Quy trình xử lý hoàn tiền" |
| Knowledge base | Tài liệu thỉnh thoảng mới cần | Chính sách, định nghĩa chỉ số |

- **Lộ trình chuyển đổi:** ① Viết eval end-to-end (10 ví dụ mẫu là đủ để bắt đầu) → ② Gộp các agent hay đi cùng nhau → ③ Biến subagent thành tool, agent chính vẫn giữ hội thoại → ④ Chuyển hẳn từng subagent thành skill/tool.
- **Tổ chức:** một team trung tâm nhỏ sở hữu system prompt và chất lượng tổng thể. Các team khác sở hữu **skill và tool**, không sở hữu agent.
- **Nguyên tắc:** *"Đo lường, mỗi lần đổi một thứ, giữ lại những gì hiệu quả."*

## Ví dụ thực tế từ bài viết

> Các tình huống dưới đây lấy từ bài viết. Tên tool và chi tiết minh họa (như `tra_don`) là do tôi đặt cho dễ hình dung.

### Ví dụ 1: Thiết kế "càng thêm càng phức tạp"

Bài viết kể lại một kiểu thiết kế mà nhiều doanh nghiệp đề xuất. Mỗi lớp được thêm vào đều nghe có vẻ hợp lý:

```text
Câu hỏi khách
  → Intent classifier (phân loại ý định)
  → Pipeline suy luận nhiều bước cố định
  → Chia theo nhóm khách hàng (segment)
  → Nhánh gợi ý riêng cho từng nhóm
  → Thêm một orchestrator multi-agent ở trên cùng
```

Góp ý của đội Anthropic thường là: **một system prompt rõ ràng + 4–5 data tool mỏng + guardrail chặt cho output** là đã đáp ứng phần lớn nhu cầu.

👉 Bài học: trước khi thêm một lớp, hãy thử xem model có tự làm được bước đó trong một vòng lặp hay không.

### Ví dụ 2: Một chục subagent chuyên trách làm hỏng thương hiệu

Một công ty dùng khoảng **12 subagent chuyên trách**. Mỗi subagent có prompt riêng do một team khác viết, nên:

- câu trả lời cho khách có giọng văn không đồng nhất, khó giữ đúng "brand voice";
- mỗi lần chuyển qua lại giữa các agent lại cộng thêm độ trễ, làm trải nghiệm người dùng kém đi.

👉 Gộp về một agent giúp có **một giọng nói duy nhất**, còn chuyên môn của từng team nằm trong skill.

### Ví dụ 3: Nhanh hơn khoảng 2 lần

Một khách hàng so sánh bộ multi-agent của họ với một single agent loop. Kết quả là **single loop chạy nhanh hơn khoảng 2 lần**. Lý do: mỗi lần gọi subagent là thêm một vòng gọi model lồng bên trong, còn single loop chạy các tool song song trong cùng một lượt.

### Ví dụ 4: Gộp xong thì trả lời sai ngày giao hàng

Một agent hỗ trợ khách hàng có hai phần: agent chính và **chuyên gia vận chuyển** (shipping specialist).

```text
TRƯỚC KHI GỘP (đúng)
  Khách: "Bao giờ đơn #123 tới?"
  Agent chính → hỏi chuyên gia vận chuyển → trả về ngày giao chính xác

SAU KHI GỘP (sai, chỉ thỉnh thoảng)
  Khách: "Bao giờ đơn #123 tới?"
  Agent chính → tự gọi tra_don() → tự đoán ngày giao từ dữ liệu đơn hàng
               (bỏ qua bước hỏi bên vận chuyển)
```

- Sau khi gộp, **độ trễ giảm**, nhưng một quy trình trước đó chạy đúng thì giờ bị hỏng.
- Lỗi chỉ xuất hiện ở **1 trong 2 lần chạy**. Nếu chỉ chạy eval một lần thì rất dễ bỏ sót.

👉 Bài học:

- Mỗi chuỗi agent đang chạy trên production cần một task eval riêng.
- Mỗi quy tắc "không được hỏng" (như "ngày giao phải lấy từ bên vận chuyển") cũng cần một task eval riêng.
- Mỗi task nên chạy nhiều lần.

### Ví dụ 5: Đặt ràng buộc cứng vào tool, không viết vào prompt

Bài viết nêu nguyên tắc: nếu bước Y cần kết quả của bước X thì **tool của Y tự kiểm tra** và trả về chỉ dẫn khi thiếu. Áp vào ví dụ 4:

```text
Agent gọi: tinh_ngay_giao(don="#123")
Tool trả về: "Chưa có dữ liệu vận chuyển. Hãy gọi tra_van_chuyen() trước."
```

Viết "nhớ hỏi bên vận chuyển trước" vào prompt thì model có thể quên. Đặt vào tool thì không thể bỏ qua.

### Ví dụ 6: Phân quyền theo người dùng nằm trong tool

Thay vì làm riêng "agent cho nhân viên" và "agent cho quản lý", bạn dùng **một vòng lặp chung**. Tool tự kiểm tra quyền và trả về lời từ chối đơn giản:

```text
Nhân viên gọi: hoan_tien(don="#123", so_tien=5_000_000)
Tool trả về: "Bạn không có quyền hoàn tiền trên 1 triệu. Hãy chuyển cho quản lý."
```

Cùng một agent vẫn phục vụ được nhiều nhóm người dùng có quyền khác nhau.

### Ví dụ 7: Code vẫn có chỗ đứng

Không phải mọi thứ đều để model tự lo. Những phần **rẻ và gần như lúc nào cũng cần** vẫn nên để code nạp sẵn. Ví dụ trong bài là *entity resolution*: xác định "đơn của tôi" là đơn nào, "khách này" là ai. Các bước máy móc cũng nên để code xử lý bên trong tool.

### Ví dụ 8: Bước trung gian là gọi subagent như một tool

Nếu chưa muốn gộp hẳn, hãy để agent chính **gọi subagent như một tool** thay vì chuyển giao cả cuộc hội thoại:

```text
Agent chính (vẫn nói chuyện với khách)
   ├─ gọi tool: chuyen_gia_van_chuyen("#123") → {"ngay_giao": "2026-11-10"}
   └─ tự tổng hợp câu trả lời
```

- **Lợi ích:** output có schema chung, kết quả được kiểm tra với dữ liệu thật, không phải viết lại hệ thống, và có ngay điểm baseline để so sánh.
- **Hạn chế còn lại:** mỗi chuyên gia vẫn có prompt, context, người phụ trách và eval riêng. Kết quả trả về vẫn chỉ là bản tóm tắt nên vẫn mất thông tin, và mỗi lần gọi vẫn cộng thêm độ trễ.
- Chỉ nên chuyển giao hẳn khi subagent **phải nói chuyện trực tiếp với người dùng**.

## Lệnh / cấu hình hay dùng

```bash
# Skill hỗ trợ chuyển đổi được nhắc trong bài
/claude-api build-eval      # tạo bộ eval
/claude-api prompt-audit    # tìm chỉ dẫn viết cho model cũ
/claude-api hillclimb       # so sánh bản gộp và bản giữ subagent bằng eval
/claude-api cost-optimize   # đề xuất tiết kiệm chi phí, mỗi lần một thay đổi
/skill-creator              # viết và dọn dẹp skill
/skill-doctor               # xem chi phí và mức dùng của từng skill
```

## Điểm dễ nhầm / gotcha

- **Bẫy định luật Conway:** công ty có bao nhiêu phòng ban thì làm bấy nhiêu agent, khiến kiến trúc thành bản sao sơ đồ tổ chức.
- **"Frankenprompt":** mỗi team thêm một câu vào system prompt, dẫn đến mâu thuẫn và giọng văn không thống nhất.
- **Tam sao thất bản:** mỗi lần chuyển giao giữa các agent chỉ còn lại bản tóm tắt, chi tiết rơi rớt dần.
- Khi có **hơn 30–50 tool** trong prompt, agent bắt đầu chọn sai. Nên gộp tool theo ý định người dùng và dùng *tool search* để nạp khi cần.
- **Gộp có thể làm hỏng quy trình đang chạy đúng**, và lỗi có thể chỉ thỉnh thoảng mới xuất hiện (xem [Ví dụ 4](#ví-dụ-4-gộp-xong-thì-trả-lời-sai-ngày-giao-hàng)).
- Khi so sánh eval, xem **task nào đổi kết quả**, không chỉ nhìn điểm tổng.

## Áp dụng vào dự án của tôi

-

## Câu hỏi còn thắc mắc

-

## Đọc thêm

- [Bài viết gốc](https://claude.com/resources/articles/building-agents-when-to-fold-subagents-into-your-main-agent-loop)
- [single-agent-loop-kit](https://github.com/anthropics/single-agent-loop-kit): bộ công cụ chuyển đổi, có demo trên hệ thống mô phỏng
