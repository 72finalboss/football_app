# ⚡ 03. Features Specification & Operation Flow

> Tài liệu này mô tả chi tiết các tính năng của ứng dụng **Live Match Win-Probability & "What-If" Simulator**, giao diện tương tác của từng màn hình và cách toàn bộ hệ thống vận hành trong thực tế.

---

## 1. Chi Tiết 3 Tính Năng Cốt Lõi

### 1.1. Biểu Đồ Momentum & Xác Suất Sống (Live Probability Ribbon)

Nằm ở nửa trên màn hình trận đấu, biểu đồ **Live Probability Ribbon** cung cấp cái nhìn toàn cảnh về diễn biến thế trận từ phút $0$ đến phút $90+$:

- **3 Đường Xác Suất Thời Gian Thực (Chủ – Hòa – Khách)**:
  - **Đường Chủ nhà (Home Win - Màu Xanh Ngọc `#10B981`)**: Biểu diễn $P_{\text{Home}}(t)$ từ $0' \to 90'+$.
  - **Đường Hòa (Draw - Màu Xám Bạc `#94A3B8`)**: Biểu diễn $P_{\text{Draw}}(t)$.
  - **Đường Đội khách (Away Win - Màu Cam Đỏ `#F97316`)**: Biểu diễn $P_{\text{Away}}(t)$.
  - Tổng giá trị của 3 đường tại mọi điểm cắt dọc theo trục hoành luôn bằng **100%**.
- **Hệ Thống Đánh Dấu Sự Kiện Trên Trục Thời Gian (Event Markers)**:
  - **Icon Bàn Thắng (`⚽`)**: Khi có bàn thắng ở phút $t_g$, một điểm neo có icon `⚽` xuất hiện ngay tại bước nhảy vọt (vertical jump) của đường xác suất. Bấm/chạm vào icon sẽ hiện Popover: *"62' ⚽ Nguyễn Văn A (1-0) — Xác suất thắng tăng vọt từ 44.2% $\to$ 71.8% (+27.6%)"*.
  - **Icon Thẻ Đỏ (`🟥`)**: Khi có thẻ đỏ ở phút $t_r$, một đường gióng dọc màu đỏ kèm icon `🟥` đánh dấu thời điểm giảm quân số và bước ngoặt xác suất.
- **Đường Giả Lập Tương Lai ("What-If Ghost Ribbon")**:
  - Khi người dùng hỏi Chatbot một kịch bản ở phút tương lai $t_{\text{sim}} > t_{\text{now}}$, biểu đồ sẽ vẽ thêm **3 đường nét đứt phát sáng** phóng chiếu từ phút hiện tại đến phút $t_{\text{sim}}$ và kéo dài tới phút $90$ để người xem so sánh trực quan với thực tế.
- **Thanh Trượt Tua Lại Lịch Sử (Interactive Time Scrubber)**:
  - Người xem có thể vuốt ngón tay dọc theo trục thời gian từ phút $0$ đến phút hiện tại để xem lại tại bất kỳ phút nào, xác suất 3 cửa và Bản đồ nhiệt 11×11 lúc đó ra sao.

---

### 1.2. Chế Độ "What-If Simulator" (Chatbot Giả Lập Kịch Bản Tương Lai)

Được thiết kế dưới dạng ngăn kéo thông minh (Bottom Sheet trên Mobile / Side Panel trên Web) để người xem vừa nhìn biểu đồ vừa đặt câu hỏi:

- **Ô Chat Ngôn Ngữ Tự Nhiên Tiếng Việt & Tiếng Anh**:
  - Hỗ trợ nhập hoặc nói (Voice-to-Text trên điện thoại) các câu hỏi giả định:
    - *"Nếu đội khách gỡ hòa ở phút 65 thì cán cân đảo chiều bao nhiêu %?"*
    - *"Nếu giữ nguyên tỉ số 1-0 này đến phút 75 thì xác suất thắng tăng từ 68% lên bao nhiêu %?"*
    - *"Nếu đội chủ nhà bị thẻ đỏ ở phút 70 thì nguy cơ mất điểm là bao nhiêu %?"*
    - *"Cần ghi thêm mấy bàn trước phút 80 để chắc chắn thắng trên 95%?"*
- **Các Nút Gợi Ý Kịch Bản 1-Chạm (Quick Scenario Chips)**:
  - Vì người xem bóng đá trên điện thoại cần thao tác nhanh, phía trên ô nhập chat luôn có các nút bấm tự động thích ứng với tỉ số hiện tại:
    - `[⚽ Khách gỡ hòa phút 65']`
    - `[🛡️ Giữ nguyên tỉ số thêm 15']`
    - `[🟥 Chủ nhà nhận thẻ đỏ]`
    - `[⚽ Chủ nhà nâng tỉ số lên 2-0]`
- **Thẻ Kết Quả So Sánh Trực Quan (Probability Swing Card)**:
  - Thay vì chỉ trả lời bằng chữ khô khan, câu trả lời của Chatbot đi kèm một thẻ đồ họa hiển thị:
    - **Trước giả lập (Baseline)** vs **Sau giả lập (What-If)**.
    - **Mũi tên biến thiên ($\Delta \% $)**: Ví dụ `Chủ thắng: 71.2% ➔ 36.4% (▼ -34.8%)`.
    - **Nút `"Áp dụng lên Heatmap 11×11"`**: Bấm vào sẽ chuyển toàn bộ Bản đồ nhiệt 11×11 sang hiển thị phân phối tỉ số của kịch bản giả lập đó.

---

### 1.3. Bản Đồ Nhiệt Tỉ Số Chính Xác (11×11 Score Matrix Heatmap)

Hiển thị ma trận $11 \times 11$ ($121$ ô) đại diện cho mọi tỉ số cuối trận từ $0\text{–}0$ đến $10\text{–}10$:

- **Cấu Trúc Lưới $11 \times 11$**:
  - **Trục dọc (Hàng $i = 0 \dots 10$)**: Số bàn thắng chung cuộc của **Đội Chủ Nhà**.
  - **Trục ngang (Cột $j = 0 \dots 10$)**: Số bàn thắng chung cuộc của **Đội Khách**.
- **Mã Hóa Màu Sắc Theo Vùng Kết Quả & Mật Độ Xác Suất**:
  - **Vùng Tam Giác Dưới ($i > j$ — Chủ thắng)**: Thang màu Xanh lục đậm dần theo xác suất.
  - **Đường Chéo Chính ($i = j$ — Hòa)**: Thang màu Vàng Hổ Phách / Xám sáng đậm dần theo xác suất.
  - **Vùng Tam Giác Trên ($i < j$ — Khách thắng)**: Thang màu Cam / Đỏ đậm dần theo xác suất.
  - **Vùng Bất Khả Thi ($i < g_H(t)$ hoặc $j < g_A(t)$)**: Khi tỉ số hiện tại là $1\text{–}0$, toàn bộ hàng $i = 0$ (Chủ ghi 0 bàn) sẽ bị gạch chéo mờ (`0.0% — Impossible`) vì đội chủ nhà đã có 1 bàn.
- **Chế Độ Xem Linh Hoạt Cho Màn Hình Điện Thoại (Compact 6×6 vs Full 11×11)**:
  - Trên màn hình điện thoại nhỏ, cung cấp nút chuyển nhanh giữa chế độ **Full 11×11 ($0\text{–}10$)** (có hỗ trợ pinch-to-zoom & tooltip chi tiết) và chế độ **Focus Zoom** quanh các tỉ số có xác suất cao nhất.
- **Chế Độ Xem Độ Chênh Lệch (`Delta Heatmap Mode`)**:
  - Khi đang bật chế độ **What-If Simulator**, người dùng có thể bật công tắc `"Xem biến thiên (Δ%)"` trên Heatmap 11×11 để thấy ô tỉ số nào tăng xác suất (màu xanh) và ô tỉ số nào giảm xác suất (màu đỏ) dưới kịch bản giả định.

---

## 2. Cách Vận Hành Của Ứng Dụng (End-to-End Operation Flow)

Ứng dụng được thiết kế để vận hành ở **2 chế độ song song**, đảm bảo người dùng mở app bất kỳ lúc nào (dù đang có trận cầu đinh trực tiếp hay lúc nửa đêm không có trận nào) đều có thể trải nghiệm trọn vẹn:

```mermaid
stateDiagram-v2
    [*] --> MatchSelection: Người dùng mở App (Web / Mobile)

    state MatchSelection {
        LiveMatches: 🔴 Trận Đấu Đang Diễn Ra (Real-time Feed)
        ReplayMatches: 🎬 Trận Đấu Giả Lập / Kinh Điển (Interactive Replay)
    }

    LiveMatches --> MatchDashboard: Chọn trận đấu
    ReplayMatches --> MatchDashboard: Chọn trận đấu & Tốc độ phát (1x, 5x, 10x)

    state MatchDashboard {
        direction LR
        WebSocketStream: Nhận dữ liệu phút t qua WebSocket
        RibbonUpdate: Vẽ Live Probability Ribbon (0' -> t')
        HeatmapUpdate: Cập nhật 11x11 Score Matrix
        WebSocketStream --> RibbonUpdate
        WebSocketStream --> HeatmapUpdate
    }

    MatchDashboard --> WhatIfMode: Hỏi Chatbot hoặc bấm Quick Chip
    state WhatIfMode {
        ParseIntent: Bóc tách tham số (minute, goals, red_cards)
        RunTrQuery: Gọi cặp tr.query(minute)
        RenderComparison: Hiển thị % Đảo chiều + Ghost Line + Ghost Heatmap
        ParseIntent --> RunTrQuery --> RenderComparison
    }
    WhatIfMode --> MatchDashboard: Quay về theo dõi trực tiếp (Live Sync)
```

### Quy trình vận hành thực tế theo thời gian:
1. **Trước trận đấu ($t = 0'$)**:
   - Hệ thống nạp thông số sức mạnh hai đội ($\lambda_{H,0}, \lambda_{A,0}$).
   - Gọi `tr.query(0, 0, 0, 0, 0)` để hiển thị xác suất khởi đầu (VD: Chủ $48\%$ – Hòa $26\%$ – Khách $26\%$) và bản đồ nhiệt 11×11 ban đầu (ô $1\text{–}1$ và $1\text{–}0$ sáng nhất).
2. **Trong suốt 90 phút thi đấu ($t = 1' \to 90'+$)**:
   - Mỗi phút trôi qua, tiến trình `live_feed_ingestor.py` cập nhật trạng thái trận đấu và gọi `tr.query(minute)`.
   - Nếu không có bàn thắng, xác suất Hòa hoặc xác suất thắng của đội đang dẫn bàn tăng dần tự nhiên theo sự hao hụt thời gian $\phi_{\text{time}}(t, T)$.
   - Ngay khi có **Bàn thắng (`⚽`)** hoặc **Thẻ đỏ (`🟥`)**, hệ thống lập tức tính lại `tr.query(minute)` với trạng thái mới, tạo ra cú nhảy xác suất (Momentum Swing) trên biểu đồ và khóa các hàng/cột tỉ số cũ trên Heatmap 11×11.
3. **Tương tác của người xem (On-demand What-If)**:
   - Bất kỳ lúc nào, người xem có thể mở Chatbot để đặt câu hỏi `"Nếu..."`. Toàn bộ tính toán giả lập diễn ra độc lập cho từng người dùng mà không làm gián đoạn luồng dữ liệu Live của trận đấu.
