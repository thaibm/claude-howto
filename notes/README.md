# Ghi chú học tập

Ghi chú cá nhân khi học repo này. Các file bài học gốc được giữ nguyên, nên khi merge bài mới từ `upstream` sẽ không bị conflict.

## Mục lục

| # | Module | Trạng thái |
|---|--------|------------|
| 01 | [Slash Commands](01-slash-commands.md) | ⬜ |
| 02 | [Memory](02-memory.md) | ⬜ |
| 03 | [Skills](03-skills.md) | ⬜ |
| 04 | [Subagents](04-subagents.md) | ⬜ |
| 05 | [MCP](05-mcp.md) | ⬜ |
| 06 | [Hooks](06-hooks.md) | ⬜ |
| 07 | [Plugins](07-plugins.md) | ⬜ |
| 08 | [Checkpoints](08-checkpoints.md) | ⬜ |
| 09 | [Advanced Features](09-advanced-features.md) | ⬜ |
| 10 | [CLI](10-cli.md) | ⬜ |

Ngoài ra: [Ghi chú chung](general.md) cho mẹo, phím tắt và kiến thức không thuộc riêng module nào.

Trạng thái: ⬜ chưa học · 🟨 đang học · ✅ đã xong

## Cách dùng

1. Học bài trong `vi/<module>/README.md`.
2. Ghi lại ý chính, gotcha và cách áp dụng vào file `notes/<module>.md` tương ứng.
3. Cập nhật trạng thái trong bảng trên.
4. Commit và push lên fork:

```bash
git add notes/ && git commit -m "docs(notes): ghi chú module 03" && git push origin main
```

## Lấy bài mới từ repo gốc

```bash
git fetch upstream && git merge upstream/main && git push origin main
```
