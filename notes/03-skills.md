# Skills — Ghi chú

> Bài học: [Tiếng Việt](../vi/03-skills/README.md) · [English](../03-skills/README.md)
>
> Trạng thái: 🟨 Đang học · Ngày học: 2026-11-04

## Ý chính cần nhớ

- Dùng **Skill** khi có một quy trình hoặc một mảng kiến thức **lặp lại nhiều lần**, và muốn Claude tự áp dụng mà không phải giải thích lại.
- Các trường hợp nên dùng Skill:
  1. **Quy trình nhiều bước lặp lại**: code review theo checklist, refactor theo quy tắc riêng, sinh tài liệu, deploy.
  2. **Kiến thức chuyên môn hoặc kiến thức nền** chỉ cần cho một loại việc nhất định: brand voice, cách hệ thống legacy vận hành, quy ước API nội bộ.
  3. **Cần kèm file hỗ trợ** như script, template, tài liệu tham khảo dài. Nhờ *progressive disclosure*, các file này chỉ được tải khi skill thực sự kích hoạt.
  4. **Có nhiều chuyên môn khác nhau.** Cài nhiều skill gần như không tốn context, vì lúc đầu Claude chỉ thấy `name` và `description`.
- So sánh với các tính năng khác:

| Nhu cầu | Nên dùng |
|---|---|
| Bối cảnh dự án **luôn cần có** trong mọi phiên | `CLAUDE.md` (Memory) |
| Tác vụ **cô lập**, không làm đầy context chính | Subagent |
| Truy cập dữ liệu hoặc dịch vụ bên ngoài (DB, API, Slack…) | MCP |
| Hành động **tự động chạy** theo sự kiện, không để Claude tự quyết | Hooks |
| Quy trình lặp lại, chuyên môn tái sử dụng | **Skill** |

- Mẹo: nếu hay dán đi dán lại cùng một đoạn hướng dẫn vào prompt, nên chuyển nó thành skill. Nếu Claude cần thông tin đó trong *mọi* tác vụ thì đặt vào `CLAUDE.md`.

## Lệnh / cấu hình hay dùng

Ai được gọi skill (frontmatter trong `SKILL.md`):

| Frontmatter | Tôi gọi `/name` | Claude tự gọi | Dùng cho |
|---|---|---|---|
| (mặc định) | Có | Có | Đa số skill |
| `disable-model-invocation: true` | Có | Không | Thao tác có tác dụng phụ: `/deploy`, `/commit` |
| `user-invocable: false` | Không | Có | Kiến thức nền, không phải lệnh |

```yaml
---
name: deploy
description: Deploy ứng dụng lên production. Dùng khi người dùng yêu cầu deploy.
disable-model-invocation: true
---
```

## Điểm dễ nhầm / gotcha

- Claude quyết định kích hoạt skill dựa trên `description`, nên phần mô tả phải nói rõ skill **làm gì** và **khi nào dùng**.
- Skill có tác dụng phụ nên đặt `disable-model-invocation: true` để Claude không tự ý chạy.

## Áp dụng vào dự án của tôi

-

## Câu hỏi còn thắc mắc

-

## Đọc thêm

- [Kiểm Soát Gọi Skill](../vi/03-skills/README.md#kiểm-soát-gọi-skill)
- [Skills So Với Các Tính Năng Khác](../vi/03-skills/README.md#skills-so-với-các-tính-năng-khác)
- [Thực Hành Tốt Nhất](../vi/03-skills/README.md#thực-hành-tốt-nhất)
