# 🗂️ Workboard — Kanban จำลองแบบไฟล์ (no-DB mode)

ระบบ track งานแบบไฟล์ ใช้แทนหน้า Kanban UI ในกรณีที่รัน agent ผ่าน
session (Claude Code) โดยไม่ได้เปิด backend/DB ของ platform จริง

## ไฟล์หลัก

- **`board.html`** ← เปิดด้วย browser เพื่ออ่าน (มี badge สี, progress bar, การ์ด)
  เป็น **แหล่งข้อมูลเดียว** — ทั้ง Project Index (ชั้น 1) และ Task Board
  ของแต่ละ project (ชั้น 2) อยู่ในไฟล์นี้ไฟล์เดียว self-contained (CSS ฝังในตัว)

## โครงสร้าง 2 ชั้น (ภายใน board.html)

- **ชั้น 1 — Project Index**: ตารางรวมทุก project + ความคืบหน้า (เสร็จ/ทั้งหมด)
- **ชั้น 2 — Project Boards**: หนึ่ง section ต่อหนึ่ง project → task board + AC

## กติกา ID

- Project ใช้เลขลำดับ: `1`, `2`, ...
- Task ใช้รูปแบบ `<project>.<task>` เช่น `1.2` = project 1 task ที่ 2

## คีย์สถานะ (แทน process_status ของ Kanban จริง)

| สัญลักษณ์ | สถานะ | ความหมาย |
|---|---|---|
| ⚪ | TODO | ยังไม่เริ่ม |
| 🟡 | IN PROGRESS | กำลังทำ (agent ทำงานอยู่) |
| 🔵 | REVIEW | รอตรวจ/รอ verify |
| 🔴 | BLOCKED | ติดขัด (ระบุใน "รอ") |
| 🟢 | DONE | เสร็จ + AC ผ่านครบ + verify แล้ว |

## Acceptance Criteria (AC)

ทุก task ควรมี AC เป็นข้อๆ — ✅ passed · ⬜ pending · ❌ failed · ➖ n/a
**กติกา:** ห้ามเปลี่ยน task เป็น 🟢 DONE จนกว่า AC จะ ✅ ครบทุกข้อ (mirror วินัยของ Lead จริง)

## Flow การทำงาน

1. งานใหม่เข้ามา → เพิ่มแถวใน Project Index + เพิ่ม section board (ถ้าเป็น project ใหม่)
2. Lead spawn subagent ที่เหมาะสม → เปลี่ยน task เป็น 🟡
3. Agent เสร็จ → Lead verify → ติ๊ก AC + เปลี่ยนเป็น 🟢 พร้อมใส่ "ตรวจโดย" + หลักฐาน
4. อัปเดตตัวเลข/progress bar ใน Index ให้ตรงกับ board เสมอ
5. commit/push เก็บถาวรผ่าน git
