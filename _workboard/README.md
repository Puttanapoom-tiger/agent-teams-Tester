# 🗂️ Workboard — Kanban จำลองแบบไฟล์ (no-DB mode)

ระบบ track งานแบบไฟล์ `.md` ใช้แทนหน้า Kanban UI ในกรณีที่รัน agent
ผ่าน session (Claude Code) โดยไม่ได้เปิด backend/DB ของ platform จริง

## โครงสร้าง 2 ชั้น

```
_workboard/
├── README.md            ← ไฟล์นี้ (คู่มือ + convention)
├── _index.md            ← ชั้น 1: ทะเบียนรวมทุก project
├── _templates/
│   └── board.md         ← แม่แบบ board ของ project ใหม่
└── NN-<slug>/           ← ชั้น 2: หนึ่งโฟลเดอร์ = หนึ่ง project
    └── board.md         ← ชื่อ project + task board + AC
```

- **ชั้น 1 (`_index.md`)** = หน้าแรก ดูว่ามี project อะไรบ้าง + ความคืบหน้ารวม
- **ชั้น 2 (`NN-<slug>/board.md`)** = เข้าไปในแต่ละ project เจอ task ทั้งหมด

## กติกา ID

- Project ใช้เลขนำหน้าโฟลเดอร์ 2 หลัก: `01-`, `02-`, ...
- Task ใช้รูปแบบ `<project>.<task>` เช่น `1.2` = project 1 task ที่ 2
  (ไม่ซ้ำ ไม่สับสนข้าม project)

## คีย์สถานะ (แทน process_status ของ Kanban จริง)

| สัญลักษณ์ | สถานะ | ความหมาย |
|---|---|---|
| ⚪ | TODO | ยังไม่เริ่ม |
| 🟡 | IN PROGRESS | กำลังทำ (agent ทำงานอยู่) |
| 🔵 | REVIEW | รอตรวจ/รอ verify |
| 🔴 | BLOCKED | ติดขัด (ระบุใน "อ้างอิง/รอ") |
| 🟢 | DONE | เสร็จ + AC ผ่านครบ + verify แล้ว |

## Acceptance Criteria (AC)

ทุก task ควรมี AC เป็นข้อๆ — สถานะ ⬜ pending · ✅ passed · ❌ failed · ➖ n/a
**กติกา:** ห้ามเปลี่ยน task เป็น 🟢 DONE จนกว่า AC จะ ✅ ครบทุกข้อ (mirror วินัยของ Lead จริง)

## Flow การทำงาน

1. งานใหม่เข้ามา → เพิ่มแถวใน `_index.md` (ถ้าเป็น project ใหม่) + สร้าง `NN-<slug>/board.md`
2. Lead spawn subagent ที่เหมาะสม → เปลี่ยน task เป็น 🟡
3. Agent เสร็จ → Lead verify → ติ๊ก AC + เปลี่ยนเป็น 🟢 พร้อมใส่ "ตรวจโดย" + หลักฐาน
4. อัปเดตตัวเลขความคืบหน้าใน `_index.md` ให้ตรงกับ board เสมอ
