# Contributing to Tatar Relay

Contributions and forks are welcome. Tatar Relay only stays composable if its
core shape is kept, so please build **with** the architecture, not around it.

## Ground rules

- **Authorized testing only.** Examples, tests, and issues use **synthetic data
  only** — never real hosts, keys, captures, or client/vendor names.
- **Keep it reproducible.** New transforms are reversible and covered by tests.
- **Be honest in docs.** Don't claim a capability the code doesn't have.

## The architecture to preserve

**The Python core is authoritative.** Every frontend (Burp/Java, mitmproxy, CLI)
talks to it; none re-implements crypto on its own side. New capability goes into
the core as a **step or a profile**, not into a frontend.

**The pipeline is fixed: Envelope → Transform → Reseal.**
- *Envelope* locates the ciphertext (and any encrypted headers) and keeps the rest.
- *Transform* is an ordered, **reversible** chain: `forward` decrypts, `backward`
  is its exact inverse — `decrypt(encrypt(x)) == x` must hold.
- *Reseal* re-applies integrity (nonce / timestamp / HMAC) on the way out.

**The five frozen contracts** (`CONTRACTS.md`) — add, never rename/remove:
1. **Profile v1 schema** (`profile.py`) — `name`, fail-closed `scope.hosts`, `vars`, `request`/`response` each `{envelope, transform, reseal}`.
2. **Context API** (`context.py`) — per-flow state lives in `session`, never module globals.
3. **Step interface** (`steps/base.py`) — `forward`/`backward` (exact inverse) + `in_type`/`out_type`, registered with `@register("name")`.
4. **Error taxonomy** (`errors.py`) — raise `DecryptError` with a category from the closed vocabulary; the proxy never crashes.
5. **Bridge JSON-RPC** (`bridge.py`) — the message contract between core and frontends.

**Config-first.** Behavior lives in YAML profiles, not code. Reach for a profile
plus an existing step first; add a new step only when no composition covers it.

**Security boundary — keep it fail-closed.**
- A profile with no allowed `scope.hosts` must not load.
- Untrusted / community profiles stay **declarative-only**
  (`allow_python_hooks: false` by default); Python hooks are opt-in for trusted
  local profiles only.
- A new step must degrade safely on hostile input (categorized `DecryptError`,
  no uncaught exception, no unbounded allocation).

## How to extend without breaking it

- **Add a cipher/codec** → a new `@register` step with a real `backward`, a
  round-trip test (`decrypt(encrypt(x)) == x`), and a hostile-input test.
- **Support a new target** → a YAML profile under `examples/`, synthetic data only.
- **New frontend** → speak the bridge JSON-RPC; don't duplicate crypto.
- **Changing a frozen contract** → additive only; a rename/removal is breaking
  and needs a version bump + discussion first.

## Workflow

1. Fork, branch from `main`.
2. `pip install -e . pytest` and make sure `pytest -q` is green (and stays green).
3. Keep commits focused; write clear messages.
4. Open a PR describing *what* changed and *why*. CI runs the tests and builds
   the Burp jar — keep both green.

By submitting a contribution you agree it is licensed under the project's MIT
license.

---

# Tatar Relay-д хувь нэмэр оруулах

Хувь нэмэр, fork тавтай морил. Tatar Relay-ийн гол бүтцийг хадгалж байж л нийлмэл
(composable) хэвээр үлдэнэ — архитектурыг тойрч биш, **түүнтэй нийцүүлж** хөгжүүл.

## Үндсэн дүрэм

- **Зөвхөн зөвшөөрөлтэй тест.** Жишээ, тест, issue бүгд **зохиомол өгөгдөлтэй** —
  бодит host, түлхүүр, capture, клиент/vendor нэр хэзээ ч бүү тавь.
- **Давтагдах чадвартай бай.** Шинэ transform нь буцаах боломжтой, тесттэй.
- **Баримтдаа шударга бай.** Кодод байхгүй боломжийг байгаа мэт бүү бич.

## Хадгалах архитектур

**Python core нь эрх мэдэлтэй.** Бүх frontend түүн рүү хандана; аль нь ч
шифрлэлтийг өөр дээрээ дахин бичихгүй. Шинэ боломж нь **step эсвэл profile** болж
core-д орно.

**Pipeline тогтмол: Envelope → Transform → Reseal.** Transform нь буцаах боломжтой
(`decrypt(encrypt(x)) == x`). Reseal гарахдаа бүрэн бүтэн байдлыг дахин тавина.

**Таван frozen contract** (`CONTRACTS.md`) — зөвхөн нэм, бүү нэрлэ/устга:
Profile схем · Context API (`session`, module global биш) · Step интерфейс
(`forward`/`backward`, `@register`) · Алдааны ангилал (`DecryptError`) ·
Bridge JSON-RPC.

**Config-first.** Үйлдэл YAML profile-д, кодод биш.

**Аюулгүй байдлын хил — fail-closed.** `scope.hosts`-гүй profile ачаалагдахгүй;
untrusted profile нь declarative-only; шинэ step хорлонтой өгөгдөлд аюулгүй унана.

## Эвдэлгүйгээр өргөтгөх

- Cipher/codec → жинхэнэ `backward`-тай `@register` step + round-trip + hostile тест.
- Шинэ target → `examples/`-д YAML profile, зөвхөн зохиомол өгөгдөл.
- Шинэ frontend → bridge JSON-RPC ярь; шифрлэлт бүү давхардуул.
- Frozen contract өөрчлөх → зөвхөн additive; нэрлэх/устгах нь breaking.

## Ажлын урсгал

1. Fork, `main`-ээс салбар гарга.
2. `pip install -e . pytest`, `pytest -q` ногоон байг.
3. Commit-оо цэгцтэй, тодорхой мессежтэй бич.
4. Юу, яагаад өөрчилснийг тайлбарласан PR нээ. CI тест + jar build-ийг шалгана.

Хувь нэмэр оруулснаар түүнийгээ төслийн MIT лицензийн дор нийтлэхийг зөвшөөрч байна.
