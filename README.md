# fashion-3d-web (skeleton)

A **web-based**, **mobile-first**, and **advanced** 3D fashion design skeleton.
Stack: React + TypeScript + Vite + Babylon.js, PWA (SW + IndexedDB via Dexie), Workers (WASM-ready/Comlink), Apollo GraphQL, Zustand.
Includes server stubs for GraphQL + REST + Webhooks.

## Highlights
- **WebGPU opt-in** with Babylon WebGPUEngine fallback to WebGL2.
- **3D loader**: glTF 2.0 + Draco/Meshopt (place decoders into `apps/web/public/libs`).
- **Mobile-friendly UI**: bottom action bar, bottom sheet properties, larger hit targets, gesture controls.
- **PWA**: offline cache masterlist/presets/assets shells; IndexedDB autosave + crash recovery.
- **Workers**: BOM calculation/validation + (placeholder) Draco/Meshopt decoding off-main-thread via Comlink.
- **Schema-first**: shared types in `packages/schemas`. Snap/constraints engine core in `packages/engine`.
- **Perf budgets**: LCP ≤ 2.5s (4G), init JS ≤ 250KB gz (excl. 3D assets), ≤200 draw calls, ≤150k tris.

## Getting started
```bash
# with pnpm (recommended for workspaces)
pnpm install
pnpm -r build
pnpm --filter web dev   # or: pnpm --filter web preview

# with npm (works, but no workspace hoist)
npm install
npm run build -w apps/web
npm run dev -w apps/web
```

> **NOTE**: Draco/Meshopt decoders are not bundled. Download and place them into:
> - `apps/web/public/libs/draco/`  -> `draco_decoder.wasm`, `draco_wasm_wrapper.js`, `draco_decoder.js`
> - `apps/web/public/libs/meshopt/` -> `meshopt_decoder.js`, `meshopt_decoder.wasm`
> (Paths are configurable in `src/3d/loaders.ts`)
>
> For a quick start without heavy 3D assets, use the demo scene with a simple box (already included).

## Monorepo layout
```
apps/web/              # React + Vite + TS + Babylon.js (PWA, Workers, Apollo, Zustand)
packages/engine/       # core snap/constraints logic (shareable)
packages/schemas/      # shared TS types for components/BOM/operations
packages/tools/        # CLI utilities (asset tagging, KTX2/meshopt pipeline hooks - stubs)
server/                # Node + TS server (GraphQL/REST/Webhooks stubs)
```

## Env
- `apps/web/.env.example` lists FE variables.
- `server/.env.example` lists BE variables (S3, JWT, etc.).

## Production notes
- Serve `apps/web/dist/` behind CDN; set long cache headers with hashed filenames.
- Host `server` separately (Fargate/Cloud Run/etc.).
- Set **feature flags** to gradually enable WebGPU and advanced modules.


## Demo glTF with anchors
Check `apps/web/public/assets/collar_demo.gltf` and `pocket_demo.gltf`. Each contains a node with `extras.anchor`, e.g.:
```json
{
  "nodes": [
    {
      "name": "COLLAR_BASE",
      "extras": { "anchor": { "type": "snap", "compatible": ["BODY_NECKLINE"], "tolerance_mm": 2, "priority": 10 } }
    }
  ]
}
```
The app reads these at runtime and uses them for **snap** compatibility.


## v4 updates
- Multi-anchor support (component can define multiple `nodes[].extras.anchor`; union of `compatible` used for snap).
- Added **Placket (demo)** with `PLACKET_BASE → BODY_CENTER_FRONT` compatibility.
- **Tech Pack PDF export** via jsPDF (button in bottom bar).
- **Save/Load Last** design using IndexedDB (Dexie).


## v5 updates
- **BODY** anchors đọc từ `/assets/body_demo.gltf` (không dùng box hard-code). Tự render marker anchor dựa trên `extras.anchor` + `translation`.
- **Gizmo** (move/rotate) tự gắn vào part sau khi snap xong hoặc khi chọn lại part (tap lên part).
- **Auto Routing**: sinh `Operation[]` cơ bản dựa trên component types (Collar/Pocket/Placket) khi **Publish**.


## v6 updates
- **Constraint engine** (simple): kiểm tra `allowedBodies`, `fabricClass`, `forbidWith` trước khi highlight/snap.
- **Orientation snap**: khi gắn, app ước lượng **qDelta = qBody * inverse(qCompAnchor)** để xoay part khớp hướng anchor; sau đó căn vị trí anchor-đến-anchor.
- **Anchor highlight 3 trạng thái**: xanh (ok), đỏ (vi phạm), mặc định (off).


## v7 updates
- **DXF overlay + metrics**: viewer 2D (canvas) và tính **seam length** demo từ DXF ASCII (LWPOLYLINE/LINE).
- **Persistence đầy đủ**: lưu/khôi phục `placements` (transform + anchor) qua Dexie + localStorage; `rehydrate` tái dựng scene.
- **Rule engine data-driven**: `assets/rules.json` nạp lúc khởi động; `constraints.ts` đọc từ rules động.
- **Snap thông minh**: chọn **component anchor gần nhất** so với body anchor, rồi căn rotation theo quaternion delta.
- **Mirror**: nút **Mirror** lật part đang chọn (x *= -1), tắt back-face culling để tránh “lọt mặt”.
- **Tech Pack**: chèn **Seam Metrics** (tổng chiều dài demo) vào PDF.
- **Pattern overlay**: nút **Patterns** mở lớp DXF 2D.



## v8 updates
- **WebGPU opt-in** (`localStorage f3d:webgpu=1` hoặc nếu trình duyệt hỗ trợ) → fallback WebGL2.
- **DXF Worker**: tính seam length trên Worker, không block UI.
- **Undo/Redo** (history) + **Auto Pair Mirror** (toggle) khi gắn vào anchor _L/_R.
- **Placements** lưu trong **Dexie** (có cả transforms) → Load Last phục hồi chính xác.
- **Tech Pack PDF (with Shot)**: chèn ảnh chụp viewport vào PDF.
- **Anchor offset (mm)**: hỗ trợ `extras.anchor.offset_mm` để đẩy part theo vector sau khi snap.


## v9 updates (Siêu nâng cao)
- **DXF bulge arcs**: tính độ dài cung (bulge) cho LWPOLYLINE → seam metrics chính xác hơn (dù mới chỉ demo 2D).
- **Rule engine JSONLogic-lite** chạy trong **Worker**: nạp `/assets/rules-logic.json`, check constraint không chặn UI.
- **Surface Attach mode**: gắn trực tiếp lên **bề mặt BODY** (tam giác được pick), lưu **barycentric** để có thể tái tạo/biến dạng trong tương lai (skeleton).
- **WebXR / AR** (demo): nút **AR** để vào trải nghiệm AR nếu trình duyệt hỗ trợ.
- **Collab (BroadcastChannel)**: join/leave “room” để đồng bộ **components & placements** giữa các tab (demo CRDT-free).
- **Adaptive LOD**: theo dõi FPS, tự giảm hardware scaling khi <45 FPS, phục hồi khi >58 FPS.
