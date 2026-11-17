# tinyphoto — Clean Architecture and agent rules

Version 1.0.0. Read 01-master-plan.md, OpenAPI, realtime-events.json and design-tokens.json before implementation. This document specifies intended structure; it does not assert those application files already exist.

## Dependency rule

Dependencies point inward: presentation → application → domain. Infrastructure implements application/domain ports; only the composition root imports concrete infrastructure and presentation together. Domain knows no HTTP, UI, database, Google, Supabase, R2, camera SDK or serialization annotations. DTOs and generated models live at transport boundaries and map explicitly to domain values. A framework's dependency injection decorator belongs outside domain. Platform SDKs never leak into use-case inputs or outputs.

Contract versions are pinned in each repository's contracts.lock.json with release tag and SHA-256 for each consumed contract. Generate clients from that exact release; generated output is read-only. Implementations may use internal fields but cannot change wire types without updating contracts first. Breaking changes require a new protocol major version and parallel migration; additive optional fields require a minor release. The closed event schemas intentionally reject unknown fields: deploy updated consumers before new payload fields, or negotiate a new event version. Never assume strict-schema additions are universally compatible.

## Backend: Python / FastAPI

```text
tinyphoto-backend/
  pyproject.toml
  uv.lock
  contracts.lock.json
  src/tinyphoto/
    domain/
      entities/{room,round,media,memory,user}.py
      values/{room_code,clock,quota}.py
      policies/{membership,capture,retention}.py
      errors.py
    application/
      ports/{unit_of_work,room_repository,media_repository}.py
      ports/{identity_provider,object_store,event_publisher,clock}.py
      use_cases/{authenticate,refresh_session,logout}.py
      use_cases/{create_room,join_room,set_ready,leave_room,close_room}.py
      use_cases/{start_round,materialize_shutter,cancel_round,get_snapshot}.py
      use_cases/{reserve_upload,complete_upload,grant_download,list_memories,delete_memory}.py
      services/{quota,outbox,cleanup}.py
      dto.py
    adapters/
      inbound/http/{app,dependencies,error_handlers,middleware}.py
      inbound/http/routes/{auth,time,rooms,rounds,media,memories}.py
      inbound/http/schemas/
      outbound/postgres/{repositories,unit_of_work,rpc}.py
      outbound/supabase/{auth,jwks,realtime}.py
      outbound/r2/{signer,metadata,delete}.py
    bootstrap/{settings,container,main}.py
  migrations/
  tests/{unit,integration,contract,security}/
  scripts/{drain_outbox,cleanup,check_dependencies}.py
  Dockerfile
  render.yaml
```

Brace groups indicate individual concrete modules, not Python brace syntax. Domain constructors enforce six numeric code characters and legal transitions. Use cases accept actor IDs from verified authentication, never from request JSON. UnitOfWork owns transaction boundaries; the database arbitrates uniqueness, locks and versions. Repository ports return domain types. ObjectStore port exposes presign_put, head, presign_get and delete only; no upload-bytes method. HTTP accepts small JSON bodies only (16 KB cap), never multipart images. Map domain validation to 422, conflict to 409, auth to 401/403, unknown or inaccessible resources to 404, quota to 429 and dependency outage to 503 with Retry-After.

No global room dictionary, asyncio.sleep countdown, filesystem drafts, background-only commit, or server affinity. Signed URLs are returned only after quota reservation commits. Outbox drain failures do not roll back a successful room command; snapshot is authoritative. Secret settings are injected at deploy time and excluded from logs. HTTP database adapter permits a future Worker runtime; baseline PostgreSQL transaction adapter can use a pooled TLS connection. Avoid runtime migrations at every cold start; migration is a separate release step.

## Web: Next.js App Router / React / Tailwind

```text
tinyphoto-web/
  package.json
  pnpm-lock.yaml
  contracts.lock.json
  app/
    layout.tsx
    page.tsx
    camera/page.tsx
    rooms/page.tsx
    rooms/[roomId]/page.tsx
    memories/page.tsx
    globals.css
  src/
    domain/{room,round,media,memory,errors}.ts
    application/
      ports/{api,camera,compositor,realtime,session,clock}.ts
      use-cases/{authenticate,shoot,join-room,start-round,recover-room}.ts
      use-cases/{upload-media,compose-group,list-memories}.ts
    infrastructure/
      api/{generated,client,mappers}/
      auth/{google,memory-session}.ts
      camera/{media-stream,frame-capture}.ts
      composition/{canvas,stamp-mask,strip-layout}.ts
      media/{r2-fetch,object-url-registry}.ts
      realtime/{supabase,event-mapper}.ts
      clock/monotonic.ts
    presentation/
      screens/{home,camera,room,memories}/
      components/{button,stamp-frame,viewfinder,countdown,flash,status}/
      hooks/{use-camera,use-room,use-memory-session}.ts
      state/{room-reducer,capture-controller}.ts
    bootstrap/container.ts
    generated/{tokens.css,tailwind-tokens.ts}/
  public/fonts/
  scripts/{generate-tokens,check-boundaries}.mjs
  tests/{unit,contract,e2e}/
  next.config.ts
```

App Router files are thin screen entry points. Camera and in-memory auth run under client components. Server rendering ships the shell, not user images. Browser globals remain in infrastructure adapters. Compositor accepts temporary frame handles, not URLs to cache. Keep Blob/ImageBitmap/Canvas resources in disposable controllers or refs, not serialized state, router params or persisted query caches. React state contains status and IDs; unmount cancels pending tasks, stops tracks and revokes object URLs. Release ImageBitmap immediately after draw. An upload abort releases its reservation only after server expiry/cleanup; abort does not imply R2 deletion.

Use fetch with bearer headers directly to backend; no image route handler or Next server action. User images use temporary blob src and plain img with an accessible description, not next/image. API DTO generation can use an OpenAPI TypeScript tool pinned by lockfile. Runtime validate realtime events before reducers see them. Supabase persistSession=false and in-memory storage adapter are mandatory; reconnect must refresh its bearer token. Auth session persistence is permitted solely via `src/infrastructure/auth/token-storage.ts` (`localStorage` with schema validation, cross-tab refresh locks, and `storage` event sync); media, passwords, raw nonces, and E2EE private keys stay strictly in RAM. CSP permits configured API, Google, Supabase wss and R2 S3 endpoints, blob images and packaged fonts; do not permit arbitrary remote image hosts. Service worker must not intercept camera/media/auth requests.

## Android: Kotlin / Jetpack Compose

```text
tinyphoto-android/
  settings.gradle.kts
  build.gradle.kts
  gradle/libs.versions.toml
  contracts.lock.json
  app/src/main/
    AndroidManifest.xml
    java/app/tinyphoto/{TinyphotoApp,MainActivity}.kt
    java/app/tinyphoto/di/AppContainer.kt
  core/domain/src/main/kotlin/app/tinyphoto/domain/
    model/{Room,Round,Media,Memory,User}.kt
    policy/{CapturePolicy,QuotaPolicy}.kt
    port/{CameraPort,CompositorPort,ApiPort,RealtimePort,SessionPort,ClockPort}.kt
    usecase/{Authenticate,Shoot,JoinRoom,StartRound,RecoverRoom,UploadMedia}.kt
  core/data/src/main/kotlin/app/tinyphoto/data/
    api/{generated,DtoMapper,ApiClient}/
    auth/{GoogleIdentity,MemorySessionStore}.kt
    camera/{CameraXAdapter,InMemoryCapture}.kt
    composition/{BitmapCompositor,StampMask,StripLayout}.kt
    media/{R2Client,MemoryImageLoader}.kt
    realtime/{SupabaseAdapter,EventMapper}.kt
    clock/ElapsedRealtimeClock.kt
  core/designsystem/src/main/kotlin/app/tinyphoto/designsystem/
    generated/{TinyphotoColors,TinyphotoDimensions,TinyphotoMotion}.kt
    {TinyphotoTheme,StampFrame,BrutalButton,Viewfinder,FlashOverlay}.kt
  feature/{home,camera,room,memories}/src/main/kotlin/
  core/domain/src/test/
  core/data/src/test/
  app/src/androidTest/
  tools/generate-tokens.kts
```

Gradle arrows: app → feature modules + core:data + core:designsystem; feature → core:domain + core:designsystem; core:data → core:domain; core:domain → Kotlin/coroutines abstractions only. Domain has no android.*, Retrofit, Ktor, Bitmap, Uri, Compose, Supabase annotations or kotlinx.serialization DTOs. In-memory frame handles are interfaces with close/dispose, with concrete Bitmap/ByteArray ownership in core:data. Keep UI state in ViewModel StateFlow, bind with lifecycle-aware collection. SavedStateHandle stores navigation IDs only, never images/tokens. Camera permission is requested at entry; a backgrounded device becomes unready. Foreground timers use SystemClock.elapsedRealtime, never repeated wall-clock delays.

Use a single HTTP stack with no disk cache and a bounded in-memory decoder. Disable Coil disk cache if used, including default loader creation. No Room image database, WorkManager upload queue, cacheDir files, photo saving, or unauthorized MediaStore/SharedPreferences. ByteArrays are released after upload completion or cancellation; immutable garbage-collected memory cannot promise secure overwriting. Disable backup in manifest and apply sensitive-window protection on capture/memories screens according to supported Android versions. Credential Manager handles Google identity; application session tokens persist solely through `EncryptedSessionStorage` (Android Keystore AES-256-GCM + `MODE_PRIVATE` `SharedPreferences`), while images, passwords, raw nonces, and E2EE private keys remain strictly RAM-only.

## Cross-repository delivery and checks

Each implementation repo receives its own AGENTS.md when scaffolded: read pinned contracts, keep domain independent, prohibit persistent image storage, do not add paid infrastructure, validate wire changes in contracts first, and report tests actually run. This document is the baseline for those rules; no agent may assume it authorizes production deployment or secret sharing.

CI verifies imports/module graph, generated-client diffs, token generator determinism and event fixtures against JSON Schema. Backend tests exercise transaction races and unauthorized commands. Web/Android tests exercise early/late shutter handling, offset uncertainty and duplicate/gapped events. Storage QA inspects browser stores, Android app/cache files and server logs with a synthetic capture. Rendering QA compares identical generated stamp masks and pixel-space strip layout on both platforms. Clock tests inject a fake monotonic clock; network integration tests include cold start and disconnect. Contract CI rejects unresolvable refs, unknown event variants, mismatched enums, accidental base64/media-body endpoints, and unversioned breaking changes. Run meaningful tests per change; do not write tests that merely restate constant values.

Only the composition roots know environment locations. Deployments supply API base URL, Supabase URL/publishable key and allowed Google client IDs; server-only settings include Supabase privileged credentials, database DSN and R2 signing credentials. No production hostname or secret is checked into contracts. HTTP examples use relative paths, which work against the configured deployment origin without invented infrastructure.
