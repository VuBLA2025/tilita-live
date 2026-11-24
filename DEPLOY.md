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
