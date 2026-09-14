-- ============================================================
-- نظام حضور الباركود للقاعات الوجاهية - Schema مستقل تمامًا
-- (مشروع Supabase جديد منفصل عن منصة الحضور الحالية)
-- التعديل الأخير: كود قاعة ثابت بدل التحقق بالموقع الجغرافي GPS
-- ============================================================

create extension if not exists "pgcrypto";

-- ---------------------------------------------
-- 1) الجامعات المستثناة (يدعم أكثر من جامعة مستقبلًا)
-- ---------------------------------------------
create table universities (
  id uuid primary key default gen_random_uuid(),
  name text not null,
  code text unique not null,           -- رمز مختصر، مثال: HATTEEN
  created_at timestamptz not null default now()
);

-- ---------------------------------------------
-- 2) الطلاب
-- ---------------------------------------------
create table students (
  id uuid primary key default gen_random_uuid(),
  university_id uuid not null references universities(id) on delete cascade,
  university_number text not null,      -- الرقم الجامعي / الأكاديمي
  full_name text not null,
  section text,                         -- الشعبة
  created_at timestamptz not null default now(),
  unique (university_id, university_number)
);
create index idx_students_university on students(university_id);

-- ---------------------------------------------
-- 3) القاعات — كود ثابت بدل الإحداثيات الجغرافية
-- ---------------------------------------------
create table halls (
  id uuid primary key default gen_random_uuid(),
  university_id uuid not null references universities(id) on delete cascade,
  name text not null,                   -- مثال: قاعة 12 - مبنى العلوم
  hall_code text not null,              -- كود ثابت مميز للقاعة، مثال: HALL-12
  created_at timestamptz not null default now(),
  unique (university_id, hall_code)
);

-- ---------------------------------------------
-- 4) جلسات المحاضرات (كل محاضرة وجاهية = جلسة)
-- ---------------------------------------------
create table lecture_sessions (
  id uuid primary key default gen_random_uuid(),
  hall_id uuid not null references halls(id) on delete cascade,
  section text not null,                -- الشعبة المستهدفة بهذه الجلسة
  lecture_title text not null,          -- مثال: محاضرة العلوم العسكرية الثامنة
  session_date date not null,
  starts_at timestamptz not null,
  ends_at timestamptz,                  -- تُملأ عند إغلاق الجلسة
  grace_period_minutes integer not null default 15,  -- مهلة قبول المسح بعد بدء المحاضرة
  status text not null default 'open' check (status in ('open','closed')),
  created_at timestamptz not null default now()
);
create index idx_sessions_hall on lecture_sessions(hall_id);
create index idx_sessions_date on lecture_sessions(session_date);

-- ---------------------------------------------
-- 5) رموز QR المتجددة (كل رمز يحمل كود القاعة + صالح لثوانٍ معدودة)
-- ---------------------------------------------
create table qr_tokens (
  id uuid primary key default gen_random_uuid(),
  session_id uuid not null references lecture_sessions(id) on delete cascade,
  hall_code text not null,              -- منسوخ من الجلسة وقت التوليد، يُقارَن به وقت المسح
  token text not null unique,           -- قيمة عشوائية موقّعة (وليس رقم تسلسلي متوقَّع)
  issued_at timestamptz not null default now(),
  expires_at timestamptz not null       -- issued_at + 10-15 ثانية غالبًا
);
create index idx_qr_tokens_session on qr_tokens(session_id);
create index idx_qr_tokens_expiry on qr_tokens(expires_at);

-- ---------------------------------------------
-- 6) سجل كل محاولة مسح (للتدقيق ومنع التزوير - يشمل المرفوضة)
-- ---------------------------------------------
create table attendance_scans (
  id uuid primary key default gen_random_uuid(),
  session_id uuid not null references lecture_sessions(id) on delete cascade,
  student_id uuid not null references students(id) on delete cascade,
  token_id uuid references qr_tokens(id),
  scanned_hall_code text,               -- كود القاعة المدموج بالرمز الممسوح
  device_fingerprint text,              -- بصمة الجهاز/المتصفح
  client_ip text,
  scanned_at timestamptz not null default now(),
  status text not null check (
    status in ('accepted','rejected_expired_token','rejected_wrong_hall',
               'rejected_duplicate_device','rejected_outside_window')
  )
);
create index idx_scans_session on attendance_scans(session_id);
create index idx_scans_student on attendance_scans(student_id);
create index idx_scans_device_time on attendance_scans(device_fingerprint, scanned_at);

-- يمنع أكثر من مسح "مقبول" واحد لنفس الطالب بنفس الجلسة
create unique index uniq_accepted_scan_per_student_session
  on attendance_scans(session_id, student_id)
  where status = 'accepted';

-- ---------------------------------------------
-- 7) النتيجة النهائية المحسوبة لكل طالب بكل جلسة (تُستخدم مباشرة للتصدير)
-- ---------------------------------------------
create table attendance_records (
  id uuid primary key default gen_random_uuid(),
  session_id uuid not null references lecture_sessions(id) on delete cascade,
  student_id uuid not null references students(id) on delete cascade,
  status text not null check (status in ('حاضر','غائب')),
  check_in_time timestamptz,
  duration_minutes integer,
  created_at timestamptz not null default now(),
  unique (session_id, student_id)
);
create index idx_records_session on attendance_records(session_id);

-- ============================================================
-- RLS: كل الجداول مقفولة افتراضيًا. الكتابة تتم فقط عبر
-- Edge Functions بصلاحية service_role (وليس مباشرة من جهاز الطالب
-- أو شاشة القاعة) — هذا هو الحاجز الحقيقي ضد التزوير.
-- ============================================================
alter table universities        enable row level security;
alter table students            enable row level security;
alter table halls               enable row level security;
alter table lecture_sessions    enable row level security;
alter table qr_tokens           enable row level security;
alter table attendance_scans    enable row level security;
alter table attendance_records  enable row level security;

-- قراءة عامة فقط لجلسات مفتوحة (تحتاجها صفحة عرض QR وصفحة المسح)
create policy "قراءة عامة للجلسات المفتوحة"
  on lecture_sessions for select
  using (status = 'open');

-- لا توجد أي policy للكتابة على مستوى anon/authenticated:
-- كل INSERT/UPDATE يمر إجباريًا عبر Edge Functions بمفتاح service_role.
