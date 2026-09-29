# ES Movie Club 촬영 배치도

촬영할 곳을 위에서 내려다본 그림 위에 **카메라 · 배우 · 마이크 · 조명**을
놓아 보는 도구입니다. 빌드도 설치도 필요 없고, `index.html` 한 파일로 돌아갑니다.

## 쓰는 법

1. `바닥 설정 › 사진 올리기` 로 위에서 내려다본 사진이나 그림을 올립니다.
2. 아래에서 놓을 것을 고르고 바닥을 누르면 그 자리에 놓입니다.
3. 놓인 것을 다시 누르면 점선 고리가 생기고, 고리를 끌면 방향이 돌아갑니다.
4. 카메라는 화면에 담기는 넓이, 마이크는 소리를 담는 넓이,
   조명은 빛이 퍼지는 넓이와 색깔을 고를 수 있습니다.
5. 장면을 여러 개 만들어 세팅을 비교할 수 있습니다.

만든 것은 브라우저에 자동 저장됩니다. 다른 기기로 옮기려면
`저장하기 › 파일로 저장` 을 쓰고, 종이로 뽑을 때는 `그림으로 저장` 을 씁니다.

## 휴대폰 홈 화면에 붙이기

아이폰은 Safari에서 열고 **공유 › 홈 화면에 추가**,
안드로이드는 Chrome 메뉴 **› 홈 화면에 추가**.
`ES Movie Club` 이름과 아이콘으로 붙고, 주소창 없이 전체 화면으로 열립니다.

## 방 기능 켜기 (여러 폰에서 같이 편집)

방을 만들면 6자리 코드와 **초대 링크**가 나온다. 친구가 그 링크를 누르면
설정도 코드 입력도 없이 바로 그 방으로 들어오고, 누가 카메라를 옮기면
다른 폰에서 바로 움직인다.

브라우저끼리 직접 연결할 방법이 없어서 중계 서버가 하나 필요하다.
무료로 쓸 수 있는 **Supabase** 를 쓴다. 준비는 딱 두 번이다.

### 한 번: 표 만들기

Supabase 프로젝트 › **SQL Editor** 에 붙여넣고 실행한다.

```sql
create table if not exists public.rooms (
  code       text primary key,
  data       jsonb not null,
  updated_at timestamptz not null default now()
);

alter table public.rooms enable row level security;

-- 코드를 아는 사람은 읽고 쓸 수 있다 (동아리용이라 이 정도면 충분하다)
create policy "rooms_read"   on public.rooms for select using (true);
create policy "rooms_insert" on public.rooms for insert with check (char_length(code) = 6);
create policy "rooms_update" on public.rooms for update using (true) with check (char_length(code) = 6);

grant select, insert, update on public.rooms to anon;
```

### 한 번: 앱에서 연결 값 넣기

앱을 열고 오른쪽 위 **사람 아이콘** → Supabase 프로젝트 ›
**Settings › API** 의 두 값을 붙여넣고 **저장하고 시작**.

- **Project URL** — `https://xxxxxxxx.supabase.co`
- **anon public 키** — `eyJhbGciOi...`

> **service_role 키는 넣지 말 것.** 모든 권한을 가진 키라 브라우저에 들어가면 안 된다.
> 바로 위의 `anon public` 키가 맞는 키이고, 이건 브라우저에 넣으라고 있는 키다.

코드를 고칠 필요도, 다시 배포할 필요도 없다. 이 기기가 값을 기억한다.
여러 사람이 늘 쓰는 사이트라면 `index.html` 위쪽 `SUPA_URL` · `SUPA_KEY` 에
박아 두어도 된다 — 그러면 앱에서 넣는 단계마저 사라진다.

### 쓰는 법

1. 오른쪽 위 **사람 아이콘** → **새 방 만들기**
2. **초대 링크 복사** → 친구에게 보내기
3. 친구는 링크만 누르면 끝. 설정도 코드 입력도 없다
4. 버튼에 뜨는 숫자가 지금 들어와 있는 사람 수다
5. 나갈 때는 같은 버튼 → **방 나가기**

코드만 알려주고 친구가 직접 입력해서 들어와도 된다.

### 어떻게 도는가

- 움직임은 **브로드캐스트**로 바로 주고받는다 (서버에 쌓이지 않는다)
- 2초에 한 번씩만 `rooms` 표에 저장한다. 나중에 들어온 사람은 이것을 받아서 시작한다
- 내가 무언가를 끌고 있는 동안 들어온 남의 변경은, 손을 뗀 뒤에 반영한다
- 바닥 그림은 용량이 커서 실시간으로는 보내지 않고, 저장될 때 같이 올라간다
- 초대 링크에는 서버 주소와 anon 키가 들어 있다. 링크를 열면 주소창에서 바로 지운다

### 주의

- 링크나 코드를 아는 사람은 누구나 그 방을 고칠 수 있다. 아무 데나 올리지 말 것
- 같은 곳을 동시에 고치면 **나중에 손댄 쪽이 남는다**

## 배포

빌드 명령이 없는 정적 사이트입니다.

- **Netlify** — 저장소를 연결하면 `netlify.toml` 대로 이 폴더를 그대로 올립니다.
  폴더를 끌어다 놓는 방식([app.netlify.com/drop](https://app.netlify.com/drop))으로도 됩니다.
- **로컬** — `index.html` 을 브라우저로 바로 열어도 똑같이 동작합니다.

## 파일

```
index.html         앱 전체 (마크업 · 스타일 · 로직)
icon.png           홈 화면 아이콘 512×512
site.webmanifest   앱 이름과 아이콘 정보
netlify.toml       배포 설정
```
