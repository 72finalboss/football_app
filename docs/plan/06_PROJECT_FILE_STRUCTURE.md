# 📁 06. Project File Structure & Module Catalog

> Tài liệu này liệt kê đầy đủ toàn bộ các file và thư mục cấu thành nên ứng dụng **Live Match Win-Probability & "What-If" Simulator**, tên file chính xác và trách nhiệm của từng file. Tất cả các file này đã được khởi tạo sẵn trong repository.

---

## 1. Sơ Đồ Cây Thư Mục Toàn Dự Án

```text
football_app/
├── PLAN.md                                      # Tài liệu Master Plan tổng quan
├── README.md                                    # Giới thiệu dự án & hướng dẫn nhanh
├── .gitignore                                   # Bỏ qua thư mục tạm (node_modules, __pycache__, .next...)
├── docker-compose.yml                           # Khởi chạy đồng bộ Backend + Frontend + Redis
│
├── docs/
│   └── plan/                                    # Bộ tài liệu quy hoạch kiến trúc & kỹ thuật
│       ├── 01_TECH_STACK_AND_ARCHITECTURE.md
│       ├── 02_MATHEMATICAL_LOGIC_AND_ENGINE.md
│       ├── 03_FEATURES_AND_OPERATION_FLOW.md
│       ├── 04_IMPLEMENTATION_ROADMAP.md
│       ├── 05_DEPLOYMENT_AND_DISTRIBUTION.md
│       └── 06_PROJECT_FILE_STRUCTURE.md
│
├── backend/                                     # TẦNG BACKEND & QUANTITATIVE ENGINE (Python FastAPI)
│   ├── pyproject.toml                           # Quản lý gói thư viện Python (uv / pip)
│   ├── requirements.txt                         # Danh sách thư viện: fastapi, numpy, scipy, uvicorn, redis...
│   ├── Dockerfile                               # Đóng gói container cho Backend
│   ├── app/
│   │   ├── __init__.py
│   │   ├── main.py                              # Điểm khởi chạy ứng dụng FastAPI & cấu hình CORS
│   │   ├── config.py                            # Quản lý biến môi trường và siêu tham số mô hình
│   │   ├── engine/                              # PHÂN HỆ LÕI TOÁN HỌC & HÀM tr.query(minute)
│   │   │   ├── __init__.py
│   │   │   ├── trajectory.py                    # Định nghĩa class TrajectoryEngine (tr) & hàm tr.query(minute)
│   │   │   ├── poisson_dixon_coles.py           # Tính phân phối Poisson 2 chiều & hiệu chỉnh Dixon-Coles
│   │   │   ├── markov_red_card.py               # Hệ số điều chỉnh cường độ ghi bàn theo Thẻ đỏ & Phút thi đấu
│   │   │   ├── score_matrix.py                  # Sinh & chuẩn hóa Ma trận nhiệt tỉ số 11x11 (0-10 x 0-10)
│   │   │   └── momentum.py                      # Tính chuỗi xác suất 3 đường (Chủ-Hòa-Khách) từ 0' -> 90'+
│   │   ├── chatbot/                             # PHÂN HỆ "WHAT-IF SIMULATOR" CHATBOT
│   │   │   ├── __init__.py
│   │   │   ├── intent_parser.py                 # Bóc tách tham số (minute, tỉ số, thẻ đỏ) từ câu hỏi tiếng Việt
│   │   │   ├── scenario_simulator.py            # Gọi cặp tr.query(minute) để tính độ đảo chiều xác suất (Delta %)
│   │   │   ├── response_generator.py            # Sinh câu trả lời phân tích trực quan cho người dùng
│   │   │   └── prompts.py                       # System prompt & Tool schema cho LLM Fallback
│   │   ├── api/                                 # PHÂN HỆ GIAO TIẾP API & WEBSOCKET
│   │   │   ├── __init__.py
│   │   │   ├── routes_match.py                  # REST API lấy thông tin trận đấu, Ribbon & 11x11 Heatmap
│   │   │   ├── routes_whatif.py                 # REST API xử lý truy vấn What-If Chatbot
│   │   │   └── ws_live.py                       # WebSocket phát sóng xác suất sống từng phút
│   │   ├── services/                            # PHÂN HỆ DỊCH VỤ DỮ LIỆU & CACHE
│   │   │   ├── __init__.py
│   │   │   ├── live_feed_ingestor.py            # Thu thập sự kiện trận đấu thực tế & Bộ phát Replay giả lập
│   │   │   └── cache_service.py                 # Lưu đệm kết quả tr.query() trên Redis
│   │   └── schemas/                             # PHÂN HỆ ĐỊNH NGHĨA DỮ LIỆU (Pydantic Schemas)
│   │       ├── __init__.py
│   │       ├── match_schema.py                  # Schema cho Trận đấu, Sự kiện, Ribbon, Ma trận 11x11
│   │       └── whatif_schema.py                 # Schema cho Câu hỏi & Kết quả so sánh What-If
│   └── tests/                                   # KIỂM THỬ ĐƠN VỊ (Unit Tests)
│       ├── test_tr_query.py                     # Kiểm thử hàm tr.query(minute) ở các mốc phút & tỉ số
│       ├── test_score_matrix_11x11.py           # Kiểm thử tính chuẩn hóa 100% của ma trận 11x11
│       └── test_whatif_parser.py                # Kiểm thử khả năng hiểu câu hỏi tiếng Việt của Chatbot
│
└── frontend/                                    # TẦNG GIAO DIỆN WEB & MOBILE (Next.js + PWA + Capacitor)
    ├── package.json                             # Quản lý thư viện JS/TS (react, next, echarts, tailwindcss...)
    ├── tsconfig.json                            # Cấu hình TypeScript
    ├── next.config.mjs                          # Cấu hình Next.js & PWA export
    ├── tailwind.config.ts                       # Cấu hình bảng màu thể thao (Home/Draw/Away colors)
    ├── capacitor.config.ts                      # Cấu hình đóng gói ứng dụng gốc Android (.apk) & iOS (.ipa)
    ├── public/
    │   ├── manifest.json                        # Web App Manifest giúp cài đặt trực tiếp lên điện thoại
    │   └── sw.js                                # Service Worker hỗ trợ hoạt động mượt và thông báo đẩy
    └── src/
        ├── app/
        │   ├── layout.tsx                       # Khung giao diện gốc (Dark theme, Mobile viewport)
        │   ├── page.tsx                         # Màn hình sảnh chính: Danh sách trận đấu Live & Replay
        │   └── match/
        │       └── [matchId]/
        │           └── page.tsx                 # Màn hình Dashboard trận đấu (Ribbon + 11x11 Heatmap + Chatbot)
        ├── components/
        │   ├── match/
        │   │   ├── MatchHeader.tsx              # Thanh tỉ số trực tiếp, đồng hồ phút thi đấu, chỉ báo thẻ đỏ
        │   │   └── EventTimelineMarkers.tsx     # Các icon ⚽ Bàn thắng và 🟥 Thẻ đỏ gắn trên trục thời gian
        │   ├── charts/
        │   │   ├── LiveProbabilityRibbon.tsx    # Biểu đồ 3 đường xác suất Chủ - Hòa - Khách (0' -> 90'+)
        │   │   └── ScoreMatrixHeatmap11x11.tsx  # Bản đồ nhiệt tỉ số chính xác 11x11 (0-10 x 0-10)
        │   └── whatif/
        │       ├── WhatIfChatWidget.tsx         # Khung chat tương tác "What-If Simulator"
        │       ├── QuickScenarioChips.tsx       # Thanh nút bấm kịch bản nhanh (Gỡ hòa, Giữ tỉ số, Thẻ đỏ)
        │       └── ProbabilitySwingCard.tsx     # Thẻ hiển thị độ đảo chiều xác suất (+% / -%) trực quan
        ├── hooks/
        │   ├── useLiveMatchSocket.ts            # Custom hook kết nối WebSocket nhận xác suất thời gian thực
        │   └── useWhatIfSimulator.ts            # Custom hook quản lý trạng thái kịch bản giả lập
        ├── lib/
        │   ├── apiClient.ts                     # Các hàm gọi REST API xuống Backend FastAPI
        │   └── probabilityFormatters.ts         # Hàm định dạng phần trăm, thang màu nhiệt độ cho Heatmap
        └── types/
            └── match.ts                         # Định nghĩa kiểu dữ liệu TypeScript cho toàn bộ Frontend
```

---

## 2. Chi Tiết Vai Trò Từng Nhóm File Cốt Lõi

### 2.1. Nhóm File Lõi Toán Học (`backend/app/engine/`)
- **[`trajectory.py`](file:///home/antigravity/72finalboss/football_app/backend/app/engine/trajectory.py)**: Nơi định nghĩa lớp `TrajectoryEngine` và đối tượng `tr` với phương thức `tr.query(minute, home_goals, away_goals, home_reds, away_reds, added_time)`. Đây là trung tâm điều phối mọi phép tính xác suất của ứng dụng.
- **[`poisson_dixon_coles.py`](file:///home/antigravity/72finalboss/football_app/backend/app/engine/poisson_dixon_coles.py)**: Chứa hàm tính mật độ xác suất Poisson hai chiều và hệ số hiệu chỉnh tương quan $\tau_\rho$ của Dixon-Coles cho các tỉ số thấp ($0\text{–}0, 1\text{–}0, 0\text{–}1, 1\text{–}1$).
- **[`markov_red_card.py`](file:///home/antigravity/72finalboss/football_app/backend/app/engine/markov_red_card.py)**: Chứa ma trận hệ số suy giảm/gia tăng tốc độ ghi bàn khi có thẻ đỏ và hàm tích phân thời gian thi đấu còn lại $\phi_{\text{time}}(t, T)$.
- **[`score_matrix.py`](file:///home/antigravity/72finalboss/football_app/backend/app/engine/score_matrix.py)**: Chịu trách nhiệm sinh ma trận $11 \times 11$ ($121$ ô tỉ số từ $0\text{–}0$ đến $10\text{–}10$), khóa các tỉ số không còn khả thi ($i < g_H$ hoặc $j < g_A$) và tính tổng tam giác trên/dưới/đường chéo để suy ra $P_{\text{Home}}, P_{\text{Draw}}, P_{\text{Away}}$.
- **[`momentum.py`](file:///home/antigravity/72finalboss/football_app/backend/app/engine/momentum.py)**: Chạy `tr.query(m)` xuyên suốt từ phút $0$ đến phút hiện tại $t$, kết hợp với danh sách sự kiện Bàn thắng / Thẻ đỏ để tạo dữ liệu đầu vào cho biểu đồ **Live Probability Ribbon**.

### 2.2. Nhóm File "What-If Simulator" Chatbot (`backend/app/chatbot/`)
- **[`intent_parser.py`](file:///home/antigravity/72finalboss/football_app/backend/app/chatbot/intent_parser.py)**: Chuyển đổi câu hỏi tiếng Việt của người dùng (VD: *"Nếu đội khách gỡ hòa ở phút 65..."* hoặc *"Nếu giữ nguyên tỉ số 1-0 này đến phút 75..."*) thành cấu trúc tham số chuẩn `(target_minute, target_home_goals, target_away_goals, target_home_reds, target_away_reds)`.
- **[`scenario_simulator.py`](file:///home/antigravity/72finalboss/football_app/backend/app/chatbot/scenario_simulator.py)**: Gọi `tr.query()` cho cả trạng thái gốc (Baseline) và trạng thái giả định (Counterfactual), tính độ lệch phần trăm ($\Delta \%$) và sự dịch chuyển trên ma trận $11 \times 11$.
- **[`response_generator.py`](file:///home/antigravity/72finalboss/football_app/backend/app/chatbot/response_generator.py)**: Định dạng kết quả tính toán thành câu trả lời dễ hiểu kèm dữ liệu cho thẻ đồ họa `ProbabilitySwingCard`.

### 2.3. Nhóm File Giao Diện 3 Tính Năng Chính (`frontend/src/components/`)
- **[`LiveProbabilityRibbon.tsx`](file:///home/antigravity/72finalboss/football_app/frontend/src/components/charts/LiveProbabilityRibbon.tsx)** & **[`EventTimelineMarkers.tsx`](file:///home/antigravity/72finalboss/football_app/frontend/src/components/match/EventTimelineMarkers.tsx)**: Hiển thị 3 đường xác suất Chủ – Hòa – Khách từ phút $0 \to 90+$ và các icon `⚽` Bàn thắng / `🟥` Thẻ đỏ trên trục thời gian.
- **[`ScoreMatrixHeatmap11x11.tsx`](file:///home/antigravity/72finalboss/football_app/frontend/src/components/charts/ScoreMatrixHeatmap11x11.tsx)**: Hiển thị bản đồ nhiệt $11 \times 11$ tỉ số chính xác với khả năng chuyển đổi giữa trạng thái Live và trạng thái What-If.
- **[`WhatIfChatWidget.tsx`](file:///home/antigravity/72finalboss/football_app/frontend/src/components/whatif/WhatIfChatWidget.tsx)**, **[`QuickScenarioChips.tsx`](file:///home/antigravity/72finalboss/football_app/frontend/src/components/whatif/QuickScenarioChips.tsx)** & **[`ProbabilitySwingCard.tsx`](file:///home/antigravity/72finalboss/football_app/frontend/src/components/whatif/ProbabilitySwingCard.tsx)**: Cụm giao diện Chatbot giả lập kịch bản tương lai.
