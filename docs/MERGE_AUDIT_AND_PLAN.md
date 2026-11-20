# ລາຍງານ Audit, ການປຽບທຽບ ແລະ ແຜນລວມລະບົບ — Kwv Tij Hmoob Xyooj

ວັນທີ: 20 ພະຈິກ 2026 · ສະຖານະ: **ຂັ້ນຕອນ 1–3 ສຳເລັດ (Audit + Comparison + Plan). ຍັງບໍ່ໄດ້ແກ້ໄຂ Source Code ໃດໆ — ລໍຖ້າການຢືນຢັນ.**

---

## 0. ຂໍ້ສະຫຼຸບສຳຄັນທີ່ສຸດ: ສອງໂປຣເຈັກແມ່ນ Code ດຽວກັນ

| ໄຟລ໌ | ເນື້ອໃນ |
|---|---|
| Project A — `kwvtij-hmoobxyooj-main_2.zip` | 182 ໄຟລ໌ (ບໍ່ມີ `.git`) |
| Project B — `kwvtij-latest.zip` | 183 ໄຟລ໌ + `.git` (8 commits, branch `main`, working tree ສະອາດ) |

ຫຼັກຖານ:
- `diff -rq` ທັງສອງໂຟນເດີ: **ທຸກໄຟລ໌ທີ່ມີຮ່ວມກັນ ຄືກັນທຸກ byte** (182/182).
- ຄວາມແຕກຕ່າງມີແຕ່: B ມີ `.git/` ແລະ `.wrangler/deploy/config.json` (ໄຟລ໌ build ທີ່ສ້າງອັດຕະໂນມັດ: `{"configPath":"../../.output/server/wrangler.json"}`).
- Project A = commit `57c1882` (HEAD) ຂອງ Project B ທຸກປະການ.

➡️ **ການສ້າງຕາຕະລາງປຽບທຽບ "A vs B" ຈະເປັນການສ້າງຂໍ້ມູນທີ່ບໍ່ມີຫຼັກຖານ** ຈຶ່ງບໍ່ເຮັດ.
ສະບັບທີ່ແຕກຕ່າງກັນຈິງໆ ມີພຽງສອງອັນ ທີ່ຢູ່ໃນ Git ຂອງ Project B:

- **Baseline** — commit `27ea15e` "Baseline: original project snapshot" (ຕົ້ນສະບັບ)
- **Current** — commit `57c1882` (= A = B) ຫຼັງຈາກ refactor ໃຫຍ່

ການປຽບທຽບດ້ານລຸ່ມ ຈຶ່ງເປັນ **Baseline vs Current** ໂດຍມີຫຼັກຖານຈາກ Source Code ແລະ ການທົດສອບຈິງ.

> ຖ້າເຈົ້າມີສະບັບອື່ນ (ເຊັ່ນ ສະບັບໃນ Lovable ທີ່ຍັງບໍ່ໄດ້ push, ຫຼື ZIP ອື່ນ) ກະລຸນາສົ່ງມາ — ຂ້ອຍຈະປຽບທຽບໃໝ່.

---

## 1. ຂັ້ນຕອນ 1 — Audit ໂຄງສ້າງ (Current `57c1882`)

### 1.1 Stack
| ສ່ວນ | ເຕັກໂນໂລຊີ |
|---|---|
| Framework | TanStack Start 1.168 (SSR) + React 19 + Vite 8, Nitro (node-server/Cloudflare) |
| UI | Tailwind CSS 4, shadcn/ui (Radix), lucide, recharts, sonner |
| Backend | Supabase (Postgres + Auth + Realtime) ຜ່ານ Lovable Cloud |
| Deploy | Dockerfile, `railway.json`, `/healthz` |
| Tests | `node --test` (unit, 22 tests), Playwright + mock Supabase (smoke 80, CRUD 24) |

### 1.2 Routes / Modules
`/dashboard` · `/members` (ສະມາຊິກ + ຕົ້ນໄມ້ຄອບຄົວ) · `/leadership` · `/elders` · `/activities` (ກິດຈະກຳ, ຜູ້ເຂົ້າຮ່ວມ, ຮູບ) · `/news` · `/finance` 🔒 · `/fund-contributors` 🔒 · `/donation-tracking` 🔒 · `/login` · `/healthz` · `/sitemap.xml`

### 1.3 Architecture (ປັດຈຸບັນ)
```
routes/*.tsx ──► components/{members,activities,finance,roster}/*  (feature UI)
      │               └─► components/enterprise/*  (design system: PageHeader, DataTable, FormDialog…)
      ▼
lib/*-store.ts ──► lib/store/realtime-store.ts (factory: load, realtime, optimistic, Result, guard)
      │                     └─► supabase-js (anon key + user JWT) ──► Postgres RLS
      ▼
lib/domain/*.ts  (pure logic: family tree, finance, fund, events, csv, format) + domain.test.ts
lib/permissions*.ts / auth-context.ts  (role from public.user_roles)
```
ຈຸດແຂງ: ແຍກ domain logic ອອກຈາກ UI, store factory ດຽວ, Result type, ກວດສິດກ່ອນຂຽນ, fail-closed role.

### 1.4 Database (13 migrations)
ຕາຕະລາງ: `profiles`, `user_roles`(enum guest/member/admin), `members`, `member_relations`, `events`, `transactions`, `fund_members`, `leadership`, `elders`, `news`, `gallery`, `participants`, `meetings`, `stories`, `files` + views `*_public`.
FK ຄົບ (CASCADE / SET NULL ຖືກຕ້ອງຕາມຄວາມໝາຍ). RLS ເປີດ **ທຸກຕາຕະລາງ** (ກວດແລ້ວ: 0 ຕາຕະລາງທີ່ບໍ່ມີ RLS).

---

## 2. ຜົນການທົດສອບທີ່ລັນຈິງ (20/11/2026)

| ການທົດສອບ | ຄຳສັ່ງ | ຜົນ |
|---|---|---|
| TypeScript | `npm run typecheck` | ✅ 0 error |
| ESLint | `npm run lint` | ✅ 0 error, 7 warning (ໃນ `components/ui/*` ຂອງ shadcn) |
| Unit tests | `npm test` | ✅ 22/22 |
| Production build | `npm run build` | ✅ ຜ່ານ |
| Browser smoke (10 ໜ້າ × 4 ຂະໜາດຈໍ × guest/member) | `node tests/e2e/smoke.cjs` | ✅ 80/80 (mock backend) |
| Browser CRUD | `node tests/e2e/crud.cjs` | ✅ 24/24 (mock backend) |
| Migrations ໃນ PostgreSQL 16 ຈິງ | apply 13 ໄຟລ໌ຕາມລຳດັບ | ✅ 13/13, migration ສຸດທ້າຍ re-run ໄດ້ (idempotent) |
| RLS probes (14 ກໍລະນີ) | `psql` ດ້ວຍ role anon / authenticated | ⚠️ ເບິ່ງຂໍ້ 3 |

ຂໍ້ຈຳກັດ: ບໍ່ໄດ້ທົດສອບກັບ Supabase production ຈິງ, Realtime server ຈິງ ຫຼື Google OAuth (ບໍ່ມີການເຂົ້າເຖິງ). ບໍ່ມີ performance test ແບບ load/Lighthouse.

---

## 3. ບັນຫາທີ່ພົບ (ຢືນຢັນດ້ວຍຫຼັກຖານ)

### 🔴 Critical
| # | ບັນຫາ | ຫຼັກຖານ |
|---|---|---|
| C1 | **ໃຜກໍໄດ້ສະໝັກບັນຊີໃໝ່ → ໄດ້ role `member` ທັນທີ → ລຶບ/ແກ້ທຸກຢ່າງໄດ້** | PG16: ບັນຊີໃໝ່ `DELETE FROM transactions` → **DELETE 15**, `DELETE FROM members` → **DELETE 15**, `UPDATE news` → **UPDATE 3** (rollback ແລ້ວ). trigger `handle_new_user` ໃສ່ `'member'` |
| C2 | **Admin ແລະ Member ມີສິດເທົ່າກັນໃນ DB** — ບໍ່ມີ policy ໃດແຍກ admin | `pg_policies`: 0 policy ທີ່ຕ້ອງການ admin ໂດຍສະເພາະ |
| C3 | **ບໍ່ມີທາງຈັດການຜູ້ໃຊ້ຈາກລະບົບ** — admin ອ່ານໄດ້ແຕ່ role ຂອງຕົນເອງ, ປ່ຽນ role ໄດ້ພຽງຜ່ານ SQL | PG16: admin ເຫັນ `user_roles` ໄດ້ 1 user (ຕົນເອງ); `profiles` read-own ເທົ່ານັ້ນ |

### 🟠 High
| # | ບັນຫາ | ຫຼັກຖານ |
|---|---|---|
| H1 | ສະຄຣິບແກ້ `proposed/role_hardening.sql` ກັນການຂຽນໄດ້ ແຕ່ **ຜູ້ໃຊ້ທີ່ລໍຖ້າອະນຸມັດ ເຫັນ 0 ສະມາຊິກ / 0 ກິດຈະກຳ** (ໜ້ອຍກວ່າຄົນບໍ່ເຂົ້າລະບົບ) | PG16 ຫຼັງ apply: guest ທີ່ login → members 0, events 0 |
| H2 | ບັນຊີທີ່ສະໝັກແລ້ວກ່ອນໜ້ານີ້ ຍັງເປັນ `member` (ສະຄຣິບບໍ່ແຕະ) — ຕ້ອງທົບທວນລາຍຊື່ | PG16: ບັນຊີເກົ່າຫຼັງ apply → DELETE 15 ຍັງໄດ້ |
| H3 | **ບໍ່ມີ Audit Log** — ລຶບ/ແກ້ການເງິນແລ້ວ ບໍ່ຮູ້ວ່າໃຜເຮັດ | ບໍ່ມີຕາຕະລາງ/trigger ໃດ |
| H4 | **ບໍ່ມີ Password Recovery** | ບໍ່ມີ `resetPasswordForEmail` / `updateUser` ໃນ `src/` |
| H5 | ຂຽນຫຼາຍຂັ້ນຕອນບໍ່ເປັນ transaction (ປ່ຽນພໍ່ → ລຶບ/ເພີ່ມ relation → ອັບເດດລຸ້ນລູກຫຼານທີລະແຖວ) — ລົ້ມກາງທາງຂໍ້ມູນບໍ່ສອດຄ່ອງ | `lib/members-data.ts` `updateMember` |
| H6 | Guest (anon) ອ່ານ `fund_members`, `participants` ໄດ້ (ລາຍຊື່ຜູ້ເຂົ້າກອງທຶນ ເປັນຂໍ້ມູນພາຍໃນ) | migration `20260709022119` grant anon SELECT 13 ຕາຕະລາງ |

### 🟡 Medium
| # | ບັນຫາ |
|---|---|
| M1 | ຮູບເກັບເປັນ base64 ໃນແຖວ DB (`lib/image.ts`) — `select *` ໂຫຼດຮູບທັງໝົດ; ບໍ່ໃຊ້ Supabase Storage |
| M2 | ໂຫຼດທັງຕາຕະລາງ, ບໍ່ມີ pagination (`.range` ບໍ່ມີ) — ຊ້າເມື່ອຂໍ້ມູນຫຼາຍ |
| M3 | Realtime payload ອາດມີ column ສ່ວນຕົວ — app ບໍ່ subscribe guest ແລ້ວ ແຕ່ DB ບໍ່ກັນ (NEEDS VERIFICATION ກັບ Realtime ຈິງ) |
| M4 | `src/integrations/supabase/types.ts` ບໍ່ມີ column ໃໝ່ (`status`, `published`) |
| M5 | ຕາຕະລາງ `meetings`, `stories`, `files` ບໍ່ມີ UI ໃດໃຊ້ (ເກັບໄວ້ — ບໍ່ລຶບຂໍ້ມູນ) |
| M6 | ບໍ່ມີ CI ລັນ typecheck/lint/test/build ອັດຕະໂນມັດ |

---

## 4. ຂັ້ນຕອນ 2 — ການປຽບທຽບ Baseline vs Current

ຄະແນນ 1–10 ອີງຕາມຫຼັກຖານ (ໂຄດ + ຜົນທົດສອບ).

| ລາຍການ | Baseline `27ea15e` | Current `57c1882` | ເລືອກ |
|---|---|---|---|
| **ຟັງຊັ່ນ** | ຫຼາຍປຸ່ມມີແຕ່ toast (Export/Import/ພິມ/ສ້າງປະກາດ), ແຈ້ງເຕືອນ hardcode ("ວັນເກີດ 3 ຄົນ", "₭500,000"), ຄຳເຫັນຂ່າວເກັບໃນ browser — **3/10** | ປຸ່ມປອມລຶບອອກ, Export CSV ຈິງ, ແຈ້ງເຕືອນຄິດຈາກຂໍ້ມູນ, ຂ່າວສະບັບຮ່າງຈິງ — **7/10** | Current |
| **UI/UX** | ແຕ່ລະໜ້າອອກແບບຕ່າງກັນ, ໜ້າ 1,000–1,600 ແຖວ — **4/10** | Design system ດຽວ (`components/enterprise`), Loading/Empty/Error states, Ctrl+K search — **8/10** | Current |
| **Logic** | Infinite recursion ໃນຕົ້ນໄມ້, balance ລົບສະແດງເປັນບວກ (`Math.abs`), filter ບໍ່ເຮັດວຽກ, toast ສຳເລັດກ່ອນບັນທຶກ — **3/10** | domain layer + 22 unit tests, cycle guard, Result type — **8/10** (ຫັກ: H5 ບໍ່ເປັນ transaction) | Current |
| **Type safety** | `@ts-nocheck` 28 ໄຟລ໌ — **2/10** | 1 ໄຟລ໌ (auto-generated), tsc 0 error — **9/10** | Current |
| **Database** | anon UPDATE members ໄດ້, ບໍ່ມີ CHECK — **4/10** | ກັນ anon, CHECK/index ເພີ່ມ, idempotent — **6/10** (ຫັກ: C1–C3, H3, M1) | Current + ແກ້ |
| **Security** | CSS ເຊື່ອງປຸ່ມຕາມ icon, role fail-open ເປັນ `member` — **2/10** | ກວດສິດໃນ store, fail-closed, CSV injection guard — **5/10** (C1 ຍັງເປີດ) | Current + ແກ້ |
| **Performance** | — **4/10** | code-split ຕໍ່ route, ແຕ່ base64 + ບໍ່ມີ pagination — **5/10** | Current + ແກ້ |
| **Responsive** | ບໍ່ໄດ້ທົດສອບ — **4/10** | 80/80 ຢູ່ 390/820/1280/1440px, ບໍ່ມີ overflow — **8/10** | Current |
| **User Experience** | — **4/10** | deep links, ເຊື່ອມ module (ກິດຈະກຳ→ການເງິນ→ການບໍລິຈາກ) — **7/10** (ຫັກ: ບໍ່ມີ reset password, ບໍ່ມີໜ້າລໍຖ້າອະນຸມັດ) | Current |
| **Tests** | ບໍ່ມີ — **0/10** | unit + e2e mock — **6/10** (ບໍ່ມີ RLS test ໃນ repo, ບໍ່ມີ CI) | Current + ແກ້ |

**ສິ່ງທີ່ຄວນເອົາຈາກ Baseline:** ບໍ່ມີຟັງຊັ່ນໃດໃນ Baseline ທີ່ດີກວ່າ Current. ຟັງຊັ່ນທີ່ Current ລຶບອອກ (Import, ຄຳເຫັນຂ່າວ, "ຈັດກອງປະຊຸມ", "ຈັດການເອກະສານ") **ບໍ່ມີ logic ຈິງໃນ Baseline** (ມີແຕ່ toast ຫຼື localStorage) — ບໍ່ຄວນກູ້ຄືນແບບເກົ່າ. ຖ້າຕ້ອງການ ຕ້ອງສ້າງໃໝ່ໃຫ້ມີ DB ຈິງ (ຕາຕະລາງ `meetings`, `files` ຍັງຢູ່ຄົບ).

**ສະຫຼຸບ:** ໃຊ້ **Current ເປັນພື້ນຖານ 100%** ແລ້ວປິດຊ່ອງຫວ່າງ C1–C3, H1–H6 ແລະ ເພີ່ມສິ່ງທີ່ Master Prompt ຕ້ອງການແຕ່ຍັງບໍ່ມີ.

---

## 5. ຂັ້ນຕອນ 3 — ອອກແບບລະບົບເປົ້າໝາຍ

### 5.1 Permission Matrix (ສະເໜີ — ກວດທີ່ DB ດ້ວຍ RLS)

| Module | Guest (ບໍ່ login / ລໍຖ້າອະນຸມັດ) | Member | Admin |
|---|---|---|---|
| ສະມາຊິກ, ຕົ້ນໄມ້ | ເບິ່ງ (ບໍ່ມີ ເບີໂທ/ທີ່ຢູ່/ວັນເກີດ/ໝາຍເຫດ) | ເບິ່ງທັງໝົດ, ເພີ່ມ, ແກ້ | + ລຶບ |
| ຄະນະນຳ, ຜູ້ອາວຸໂສ | ເບິ່ງ (ສາທາລະນະ) | ເບິ່ງ, ເພີ່ມ, ແກ້ | + ລຶບ |
| ກິດຈະກຳ, ຮູບ | ເບິ່ງ | ເບິ່ງ, ເພີ່ມ, ແກ້, ຈັດການຜູ້ເຂົ້າຮ່ວມ | + ລຶບ |
| ຂ່າວ | ເບິ່ງສະເພາະທີ່ເຜີຍແຜ່ | ເບິ່ງທັງໝົດ, ຂຽນ, ແກ້ | + ລຶບ |
| ການເງິນ, ກອງທຶນ, ການບໍລິຈາກ | ❌ | ເບິ່ງ, ບັນທຶກ | + ແກ້, ລຶບ |
| ຜູ້ໃຊ້ & ສິດ, ອະນຸມັດບັນຊີ | ❌ | ❌ | ✅ |
| Audit Log | ❌ | ❌ | ✅ ອ່ານຢ່າງດຽວ |

> ຈຸດທີ່ຕ້ອງການການຕັດສິນໃຈ: Member ແກ້/ລຶບທຸລະກຳການເງິນໄດ້ບໍ? ສະເໜີ: **ບໍ່** (ບັນທຶກໄດ້ຢ່າງດຽວ, admin ແກ້/ລຶບ) ເພື່ອຄວາມໂປ່ງໃສ.

### 5.2 Authentication Flow
```
Sign up ──► handle_new_user: profile + role 'guest' (status = pending)
   │            └─► ໜ້າ "ລໍຖ້າອະນຸມັດ" (ເຫັນຂໍ້ມູນສາທາລະນະເທົ່າກັບ visitor)
   ▼
Admin ► /admin/users ► approve → RPC admin_set_role(user, 'member')  (SECURITY DEFINER, ກວດ is_admin())
   │                                    └─► audit_log
Login (email/password, Google) ─► resolveRole() ─► UI + RLS
Forgot password ─► resetPasswordForEmail ─► /reset-password ─► updateUser({password})
Logout / Logout ທຸກອຸປະກອນ (signOut({scope:'global'}))
```

### 5.3 Database Changes (migration ໃໝ່ — additive, ບໍ່ລຶບຂໍ້ມູນ)
1. `public.is_admin()`, `public.is_member()` helper functions.
2. `handle_new_user` → `'guest'`; ຂຽນທັບ policy SELECT ໃຫ້ authenticated guest ອ່ານ column ສາທາລະນະເທົ່າກັບ anon.
3. ແຍກ DELETE (ແລະ UPDATE ການເງິນ) ເປັນ admin-only.
4. `profiles`: ເພີ່ມ `status` (pending/active/suspended), `approved_by`, `approved_at`; admin ອ່ານໄດ້ທຸກຄົນ.
5. RPCs: `admin_list_users()`, `admin_set_role(uuid, app_role)`, `admin_set_status(uuid, text)` — ກັນ admin ຖອນສິດຕົນເອງເປັນຄົນສຸດທ້າຍ.
6. `audit_log` (id, at, actor, table, op, row_id, old jsonb, new jsonb) + trigger ທຸກຕາຕະລາງຂໍ້ມູນ; ຂຽນໄດ້ແຕ່ trigger, ອ່ານໄດ້ແຕ່ admin; ບໍ່ບັນທຶກ column ຮູບ base64.
7. RPC `move_member_parent(id, parent)` — ປ່ຽນພໍ່ + relation + ລຸ້ນລູກຫຼານ ໃນ transaction ດຽວ (ແກ້ H5).
8. ຖອນ anon SELECT ຈາກ `fund_members` (H6) — `participants` ຂຶ້ນກັບການຕັດສິນໃຈ.
9. (ໄລຍະ 2) Storage bucket `photos` + policies; ຍ້າຍ base64 ເທື່ອລະໜ້ອຍ ໂດຍເກັບ column ເກົ່າໄວ້ຈົນກວ່າຢືນຢັນ.

**Backup / Rollback:** ກ່ອນ apply: `pg_dump` (ຫຼື Supabase backup) ທັງ schema + data; ທຸກ migration ມີ rollback block; ທົດສອບໃນ PG16 ກ່ອນ → staging → production.

### 5.4 Frontend ໃໝ່
- `/admin/users` — ລາຍຊື່ຜູ້ໃຊ້, ອະນຸມັດ, ປ່ຽນ role, ລະງັບ (ສະເພາະ admin; ກວດທີ່ RPC ອີກຊັ້ນ).
- `/admin/audit` — Audit log ພ້ອມ filter (ຕາຕະລາງ, ຜູ້ໃຊ້, ວັນທີ) + Export CSV.
- `/forgot-password`, `/reset-password`; ໜ້າ/ແຖບ "ລໍຖ້າອະນຸມັດ".
- `permissions-core.ts` ອັບເດດຕາມ matrix ໃໝ່ (delete → admin).
- Sidebar: ກຸ່ມ "ຜູ້ດູແລລະບົບ" (ສະແດງສະເພາະ admin).
- Folder ເພີ່ມ: `src/routes/admin.*.tsx`, `src/lib/admin-store.ts`, `src/components/admin/*`.

### 5.5 ການທົດສອບທີ່ຈະເພີ່ມ
- `supabase/tests/rls.sql` — RLS probe suite (ຄືທີ່ລັນໃນລາຍງານນີ້) ລັນໄດ້ໃນ PG16 ທ້ອງຖິ່ນ: guest/pending/member/admin × CRUD ທຸກຕາຕະລາງ.
- Unit tests ສຳລັບ permission matrix ໃໝ່.
- e2e: admin approve flow, password reset UI, pending user.
- GitHub Actions: typecheck + lint + test + build + RLS suite.

---

## 6. ແຜນດຳເນີນງານ (ລໍຖ້າຢືນຢັນ)

| ໄລຍະ | ເນື້ອໃນ | ຄວາມສ່ຽງ |
|---|---|---|
| **P1 Security & RBAC** | migration 5.3 (ຂໍ້ 1–8) + RLS test suite | ປ່ຽນສິດຜູ້ໃຊ້ — ຕ້ອງ backup + ທົບທວນລາຍຊື່ member ທີ່ມີຢູ່ |
| **P2 User management** | `/admin/users`, pending page, password recovery, logout ທຸກອຸປະກອນ | ຕ່ຳ |
| **P3 Audit log UI** | `/admin/audit` | ຕ່ຳ |
| **P4 Performance** | Storage ສຳລັບຮູບ, pagination/column select | ກາງ (ຍ້າຍຂໍ້ມູນ) |
| **P5 QA & Delivery** | CI, e2e ເພີ່ມ, Test Report, Deployment + Rollback guide ອັບເດດ | ຕ່ຳ |

ຂໍ້ຄວນລະວັງ: repo ນີ້ເຊື່ອມກັບ Lovable — ທຸກ commit ທີ່ push ໄປ `main` ຈະ sync ກັບ Lovable. ແນະນຳໃຫ້ເຮັດວຽກໃນ branch ແຍກ.
