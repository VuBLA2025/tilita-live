# Tilita LIVE — Gói deploy VPS

Game vẽ tên/avatar người xem lên tường vô tận (Three.js), tương tác qua
comment / like / follow / quà TikTok LIVE. Bản gốc chạy local Windows,
gói này chỉ thích ứng tối thiểu để chạy sau reverse proxy trên VPS.

## Nguồn upstream

- Repo: https://github.com/ntanh23112005/tiktok-live-template
- Thư mục: `tilita-tiktok-live-threejs-theanh/` (v0.4.0)
- Commit: `fa60e5444c9c35f7c2335934c561383438b55c29` — "Publish Live Template for community init"
- Code gốc giữ nguyên, chỉ copy các file cần chạy: `package.json`,
  `package-lock.json`, `index.html`, `public/`, `src/`, `server/`, `tests/`.
  Không copy `output/` (ảnh QA), `design-system/`, `.agents/`,
  `node_modules/`, `dist/`, `data/`.

## Đã patch gì trong `server/index.js` (và vì sao)

Chỉ 4 chỗ, đều đánh dấu `// VPS adaptation`. Không đổi luật chơi,
không đổi tài khoản TikTok, không thêm mật khẩu panel:

1. `server.listen(port, '0.0.0.0')` thay vì `'127.0.0.1'` — trong
   container, Traefik nối tới IP container; bind loopback sẽ bị refused.
2. `allowedHosts` lấy thêm từ env `TILITA_ALLOWED_HOSTS` (danh sách cách
   nhau bằng dấu phẩy, ví dụ `"tilita.bla.io.vn:4205"`), cộng thêm
   `localhost` / `127.0.0.1`. Không đặt env thì giữ đúng hành vi gốc
   (chỉ localhost).
3. Kiểm tra Origin cho `/api/*` POST, `/events`, `/api/music/upload`:
   chấp nhận origin có host nằm trong `allowedHosts` với scheme `http`
   **HOẶC** `https`. Bản gốc chỉ chấp nhận đúng `http://<host>` nên sau
   HTTPS sẽ fail. Không nới lỏng gì khác (vẫn chặn host lạ, vẫn yêu
   cầu POST + JSON cho control API).
4. Không đổi cổng mặc định 5178 trong code; cổng chạy lấy từ env `PORT`.

## Chạy trên Dokploy

- Cổng container: **4205**. Domain đề xuất: **tilita.bla.io.vn**
  (trỏ tới compose, HTTPS qua Traefik).
- Env đặt trên Dokploy (đã có sẵn trong `docker-compose.yml`):
  - `PORT=4205`
  - `TILITA_ALLOWED_HOSTS=tilita.bla.io.vn:4205`
  - `TILITA_DATA_DIR=/app/data`
  - `NODE_ENV=production`
- Studio (panel điều khiển): `https://tilita.bla.io.vn/`
- Overlay cho OBS (1080×1920): `https://tilita.bla.io.vn/overlay`
- Mở panel → nhập username TikTok → bấm Connect. Nhạc nền upload trong
  panel (MP3/WAV/OGG/M4A/AAC/FLAC/WebM, tối đa 50MB).

## ⚠️ QUAN TRỌNG — không có mật khẩu panel

Bản gốc **KHÔNG có mật khẩu panel**: ai có link cũng điều khiển được
hàng chờ, bật/tắt nhạc, nối/ngắt TikTok và xem báo cáo. Chỉ chia link
cho người tin tưởng, không đăng link panel công khai.

## Dữ liệu bền (volume)

- `state.json` (hàng chờ, báo cáo phiên, cài đặt) và nhạc upload nằm
  trong volume Docker `tilita-data` → gắn tại `/app/data`.
- Redeploy giữ được dữ liệu. **Xóa volume thì mất hết** state + nhạc.
- Nhạc mẫu `public/audio/reference-shot.wav` (tiếng súng) nằm trong
  image, không nằm trong volume.

## Trạng thái

- Kiểm chứng local bản đã patch ngày 2026-11-24: `npm ci` + `npm run build`
  OK; chạy `PORT=4205` — `/api/health`, `/`, `/overlay` đều 200;
  Host `tilita.bla.io.vn:4205` được chấp nhận; simulate gift 30 xu vào
  hàng chờ đúng bậc `coin30` với cả Origin `http://127.0.0.1:4205` và
  `https://tilita.bla.io.vn:4205`.
- **Chưa nối TikTok thật, chưa deploy lên Dokploy.**

## ĐÃ LIVE 2026-11-24 ~22:56 ICT — https://tilita.bla.io.vn/
- Repo: `VuBLA2025/tilita-live`, nhánh `main`. GitHub tự tạo thêm commit Initial (README) `bd3873b7` trước snapshot `8fb7c1b3` (29 file text); sau đó 3 commit sửa docker-compose.yml: `99c5bba9` (network), `504d09c3` (default network), `670955d6` (allowed hosts). Đồng bộ push ghi ở `~/workspace/tilita-deploy.muse-github-snapshot-sync.json`.
- Dokploy: project "Tilita LIVE" (`E12oHjitbxrNNGwLqBK7s`, env production `tyQ8ryJQNNf-faZrjX_7X`), compose `tilita-live` (`YGyml5R38f7cLQ3SyH1zK`, appName `compose-reboot-haptic-driver-dyz93g`, sourceType github, githubId `0-f_X4vn1CYb3EnTqIaQo`), domain `tilita.bla.io.vn` (`SI85FMmm2gcBRMVjMKSDy`, port 4205, https:true, Let's Encrypt, serviceName `app`). Deployment thành công `1-Jua33lW2Qsq41OrRbWn` (22:55 UTC+7 ~15:55 UTC); IDs trong `dokploy-ids.json` (file này KHÔNG push lên GitHub).
- Kiểm chứng live: `/api/health` 200 `{app: tilita-live-threejs-theanh, version 0.4.0}`, `/` studio 200, `/overlay` 200, `/api/state` đủ 9 bậc/ưu tiên. Simulate gift 30 xu qua Origin `https://tilita.bla.io.vn` → accepted, hàng chờ `coin30` priority 50 `@kiemtra`; sau đó đã `/api/clear` → hàng chờ 0. TikTok chưa nối (connection disconnected, host trống) — đại ca nhập username đang live ở panel rồi bấm Connect.
- 3 lần deploy lỗi trước khi thành công (bài học):
  1. Deploy 1+2 lỗi ~13s/2s, không errorMessage. Log `deployment.readLogs` cho thấy build xong rồi chết ở `Network ..._default Creating: all predefined address pools have been fully subnetted` — Docker trên host hết subnet cho network mới (quá nhiều compose, mỗi compose 1 network).
  2. Chỉ gán service vào network external `dokploy-network` KHÔNG đủ — docker compose vẫn tạo network default của project. Phải định nghĩa lại chính network `default` thành external `dokploy-network` thì mới hết lỗi (deploy `AczOvKqDZWfBXCvhsxOWm` done).
  3. Sau đó `/api/health` trả 403 "only accepts localhost": Traefik gửi Host `tilita.bla.io.vn` không kèm cổng, trong khi env chỉ có `tilita.bla.io.vn:4205`. Sửa env thành `tilita.bla.io.vn,tilita.bla.io.vn:4205,tilita.bla.io.vn:443` (cả trong compose file lẫn Dokploy env) rồi deploy lại thì 200.
- Chưa push: `public/audio/reference-shot.wav` (8,5KB, file nhị phân — snapshot push của GitHub connector chỉ nhận file text). Tiếng súng dùng mẫu tổng hợp dự phòng theo code gốc; muốn có đúng mẫu gốc thì upload file này lên repo bằng trình duyệt rồi deploy lại.
- Lưu ý: bản gốc KHÔNG có mật khẩu panel — ai có link cũng điều khiển được hàng chờ/kết nối TikTok. State + nhạc nằm trong volume `tilita-data` (redeploy giữ, xóa volume thì mất). Chưa kiểm chứng: nối TikTok thật, giọng/nhạc bằng tai, overlay 9:16 trên OBS.
