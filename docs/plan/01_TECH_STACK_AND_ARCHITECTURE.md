# 🏗️ 01. Tech Stack & System Architecture

> Tài liệu này phân tích chi tiết các công nghệ (Tech Stack) cần thiết để xây dựng ứng dụng **Live Match Win-Probability & "What-If" Simulator**, lý do lựa chọn từng công nghệ và kiến trúc vận hành tổng thể.

---

## 1. Phân Tích Lựa Chọn Tech Stack

Để đáp ứng yêu cầu vừa có **Engine tính toán xác suất chuẩn xác (`tr.query(minute)`) bằng Python**, vừa có **Giao diện biểu đồ thời gian thực mượt mà trên cả Web & Mobile (iOS/Android)**, kiến trúc tối ưu nhất là mô hình **Decoupled Full-Stack (Python FastAPI Backend + Next.js/Capacitor Cross-Platform Frontend)**.

### 1.1. Tầng Lõi Toán Học & Backend (Quantitative Engine & API)

| Thành phần | Công nghệ đề xuất | Vai trò trong ứng dụng |
| :--- | :--- | :--- |
| **Ngôn ngữ chính** | **Python 3.11+** | Ngôn ngữ chuẩn mực cho mô hình định lượng thể thao, nơi đặt class `TrajectoryEngine` (`tr`) và hàm `tr.query(minute)`. |
| **Thư viện Toán học** | **NumPy 2.x & SciPy (`scipy.stats.poisson`, `scipy.linalg.expm`)** | Vector hóa phép tính ma trận $11 \times 11$ ($121$ ô tỉ số), tính hàm mật độ xác suất Poisson hai chiều, hiệu chỉnh Dixon-Coles và ma trận chuyển trạng thái Markov liên tục theo thời gian. |
| **Web Framework** | **FastAPI + Uvicorn (ASGI)** | Cung cấp REST API tốc độ cao (độ trễ `<5ms`) cho What-If Simulator và hỗ trợ **WebSockets** nguyên bản để đẩy chuỗi xác suất sống mỗi giây tới hàng ngàn người xem cùng lúc. |
| **Xác thực kiểu dữ liệu** | **Pydantic v2** | Đảm bảo đầu vào/đầu ra của `tr.query(minute)` và JSON trả về cho biểu đồ luôn đúng cấu trúc, không bao giờ xảy ra lỗi sai lệch tổng xác suất $\neq 100\%$. |

### 1.2. Tầng AI / NLP Cho "What-If Simulator" Chatbot

| Thành phần | Công nghệ đề xuất | Vai trò trong ứng dụng |
| :--- | :--- | :--- |
| **Tầng 1: Fast Vietnamese Intent Parser** | **Python Regex + Rule-based Slot Filling** | Bắt trực tiếp các câu hỏi tiếng Việt phổ biến (VD: *"Nếu đội khách gỡ hòa ở phút 65..."*, *"Nếu giữ nguyên tỉ số 1-0 đến phút 75..."*, *"Nếu chủ nhà bị thẻ đỏ phút 50..."*) với **độ trễ 0ms và không tốn phí API**. |
| **Tầng 2: LLM Function Calling (Fallback)** | **Gemini 2.5 Flash / OpenAI GPT-4o-mini (Structured Outputs)** | Khi người dùng hỏi câu phức tạp hoặc nhiều điều kiện lồng nhau (VD: *"Giả sử phút 70 đội khách vừa bị thẻ đỏ vừa bị dẫn thêm 1 bàn thì cơ hội gỡ hòa còn bao nhiêu?"*), LLM tự động trích xuất JSON tham số để gọi `tr.query(...)`. |

### 1.3. Tầng Frontend & Biểu Đồ Trực Quan (Web & Mobile UI)

| Thành phần | Công nghệ đề xuất | Vai trò trong ứng dụng |
| :--- | :--- | :--- |
| **Framework UI** | **Next.js 15 (React 19) + TypeScript** | Xây dựng giao diện đáp ứng (Responsive Mobile-First), hỗ trợ xuất tĩnh hoặc SSR, dễ dàng nhúng vào vỏ ứng dụng di động. |
| **Styling & Animation** | **Tailwind CSS + Framer Motion + Lucide Icons** | Thiết kế giao diện Dark Mode chuẩn thể thao (giống FotMob / SofaScore / Opta Analyst), hiệu ứng chuyển số mượt mà khi xác suất đảo chiều. |
| **Live Probability Ribbon** | **Apache ECharts (hoặc Visx / D3.js)** | Vẽ biểu đồ 3 đường (Chủ – Hòa – Khách) từ phút $0 \to 90+$, hỗ trợ vùng tô màu (Area Gradient), đường nét đứt cho kịch bản giả lập ("What-If Ghost Line"), và ghim các icon ⚽ Bàn thắng / 🟥 Thẻ đỏ lên đúng tọa độ phút. |
| **11×11 Score Matrix Heatmap** | **Custom CSS Grid / Canvas + D3 Scale Chromatic** | Render bảng ma trận $11 \times 11$ ($0\text{–}10 \times 0\text{–}10$) với thang màu nhiệt độ (Color Interpolation), làm mờ (dim/lock) các ô tỉ số không còn khả thi (nhỏ hơn tỉ số hiện tại) và làm nổi bật top 3 tỉ số dễ xảy ra nhất. |
| **Quản lý State** | **Zustand + TanStack Query** | Đồng bộ trạng thái giữa 3 thành phần: khi người dùng hỏi Chatbot hoặc kéo thanh trượt phút `minute`, cả **Biểu đồ Ribbon** và **Bản đồ nhiệt 11×11** tự động cập nhật đồng thời. |

### 1.4. Tầng Đóng Gói Mobile App (Cho Người Dùng Tải Về Trực Tiếp)

| Thành phần | Công nghệ đề xuất | Vai trò trong ứng dụng |
| :--- | :--- | :--- |
| **Progressive Web App (PWA)** | **`manifest.json` + Workbox Service Worker** | Người dùng mở link web trên Safari (iOS) hoặc Chrome (Android) $\to$ bấm **"Thêm vào màn hình chính" (Add to Home Screen)** là dùng ngay như App gốc toàn màn hình, không cần qua chợ ứng dụng. |
| **Native Mobile Wrapper** | **Capacitor 6 (Android `.apk`/`.aab` & iOS `.ipa`)** | Đóng gói trực tiếp codebase Next.js thành file cài đặt **`.apk`** để người dùng Android tải về cài đặt 1 chạm, hoặc đẩy lên Google Play Store / Apple App Store. |

### 1.5. Tầng Dữ Liệu, Cache & Live Sports Feed

| Thành phần | Công nghệ đề xuất | Vai trò trong ứng dụng |
| :--- | :--- | :--- |
| **In-Memory Cache & Pub/Sub** | **Redis (Upstash / Redis Stack)** | Lưu cache kết quả `tr.query(minute)` cho mọi tổ hợp trạng thái khả dĩ của trận đấu đang diễn ra, giúp phản hồi dưới `1ms` và phát sóng qua WebSocket Pub/Sub. |
| **Cơ sở dữ liệu** | **PostgreSQL (Supabase / Neon)** | Lưu thông tin trận đấu, lịch sử biến động xác suất từng phút (Time-series snapshots), các sự kiện bàn thắng/thẻ đỏ và lịch sử câu hỏi What-If. |
| **Nguồn dữ liệu bóng đá** | **API-Football / Sportmonks / Mock Live Simulator** | Cung cấp dữ liệu trận đấu thời gian thực (phút thi đấu, tỉ số, thẻ đỏ, tỉ lệ cược/xG đầu trận để ước lượng $\lambda_{H,0}, \lambda_{A,0}$) + Chế độ Replay trận đấu kinh điển để demo bất cứ lúc nào. |

---

## 2. Kiến Trúc Tổng Thể Hệ Thống (System Architecture)

```mermaid
flowchart TB
    subgraph External["Nguồn Dữ Liệu Ngoài"]
        FEED["Live Football API\n(API-Football / Opta / Replay Simulator)"]
        LLM["LLM Provider (Optional Fallback)\n(Gemini Flash / GPT-4o-mini)"]
    end

    subgraph Backend["Backend Server (Python FastAPI)"]
        INGEST["Live Feed Ingestor\n(services/live_feed_ingestor.py)"]
        ENGINE["Quantitative Core Engine\nTrajectoryEngine: tr.query(minute)\n(engine/trajectory.py)"]
        MATRIX["11x11 Score Matrix Generator\n(engine/score_matrix.py)"]
        PARSER["Hybrid What-If Intent Parser\n(chatbot/intent_parser.py)"]
        SIM["Scenario Simulator\n(chatbot/scenario_simulator.py)"]
        WS["WebSocket Broadcaster\n(api/ws_live.py)"]
        REST["REST API Endpoints\n(api/routes_match.py, routes_whatif.py)"]
    end

    subgraph Storage["Lưu Trữ & Bộ Nhớ Đệm"]
        REDIS[("Redis Cache & Pub/Sub\nPre-computed tr.query states")]
        PG[("PostgreSQL Database\nMatch History & Events")]
    end

    subgraph Client["Client App (Web / PWA / Android APK / iOS)"]
        HEADER["Match Header\n(Live Score, Minute, Red Cards)"]
        RIBBON["Live Probability Ribbon\n(3 đường Chủ-Hòa-Khách + Icon ⚽/🟥)"]
        HEATMAP["11x11 Score Matrix Heatmap\n(0-10 x 0-10 Exact Score Grid)"]
        CHAT["What-If Simulator Chatbot\n(Hỏi kịch bản & So sánh % Đảo chiều)"]
    end

    FEED -->|Polling / Webhook mỗi 5s| INGEST
    INGEST -->|Cập nhật phút t, tỉ số, thẻ đỏ| ENGINE
    ENGINE --> MATRIX
    MATRIX --> REDIS
    INGEST --> PG
    REDIS --> WS
    WS -->|Stream JSON mỗi giây| HEADER
    WS -->|Cập nhật điểm mới phút t| RIBBON
    WS -->|Cập nhật xác suất 121 ô| HEATMAP

    CHAT -->|Câu hỏi tiếng Việt| REST
    REST --> PARSER
    PARSER -.->|Câu hỏi phức tạp| LLM
    PARSER -->|Baseline State & What-If State| SIM
    SIM -->|Gọi 2 lần tr.query(minute)| ENGINE
    SIM -->|Trả về Delta % + Ghost Matrix 11x11| CHAT
    CHAT -->|Preview kịch bản lên biểu đồ| RIBBON
    CHAT -->|Preview kịch bản lên bảng nhiệt| HEATMAP
```

---

## 3. Luồng Giao Tiếp Giữa Frontend và Backend

1. **Khi người dùng mở trận đấu (`GET /api/v1/matches/{match_id}/state`)**:
   - Backend trả về toàn bộ chuỗi lịch sử xác suất từ phút $0$ đến phút hiện tại $t_{now}$ (để vẽ ngay biểu đồ **Live Probability Ribbon**), danh sách sự kiện (Bàn thắng, Thẻ đỏ) và **Ma trận nhiệt 11×11** tại phút $t_{now}$.
2. **Khi trận đấu đang diễn ra (`WS /ws/matches/{match_id}`)**:
   - Cứ mỗi phút (hoặc ngay khi có sự kiện Bàn thắng / Thẻ đỏ), Backend gọi `tr.query(minute=t_now, ...)` và đẩy gói tin qua WebSocket.
   - Biểu đồ Ribbon nối dài thêm 1 điểm dữ liệu, đồng thời bảng Heatmap 11×11 đổi màu mượt mà theo phân phối xác suất mới.
3. **Khi người dùng hỏi Chatbot What-If (`POST /api/v1/matches/{match_id}/what-if`)**:
   - Frontend gửi câu hỏi (VD: *"Nếu đội khách gỡ hòa ở phút 65 thì cán cân đảo chiều bao nhiêu %?"*) kèm trạng thái trận đấu hiện tại.
   - Backend phân tích câu hỏi $\to$ gọi `tr.query()` cho trạng thái gốc và trạng thái giả lập $\to$ trả về câu trả lời văn bản + số liệu đảo chiều ($\Delta P_H, \Delta P_D, \Delta P_A$) + ma trận 11×11 giả lập để người dùng xem trực quan trên màn hình.
