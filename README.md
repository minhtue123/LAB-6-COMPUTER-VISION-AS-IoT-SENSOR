# Lab 6 — Computer Vision as IoT Sensor

> **Môn học**: Triển khai, phát triển ứng dụng AI và IoT  
> **Vị trí**: Buổi 6 trong chuỗi AIoT Deployment Pipeline  
> **Tác giả**: PHUPHU2310  
> **Cập nhật**: 2026-06-10

---

## Tổng quan

Lab này đưa camera hoặc ảnh vào hệ thống AIoT như một **cảm biến trực quan**. Thay vì đọc giá trị số từ cảm biến nhiệt độ hay độ ẩm, hệ thống đọc frame từ camera, trích xuất metadata, sinh visual event và hiển thị trên dashboard thời gian thực.

```
Camera / Ảnh
  → Stream / Snapshot / Upload
  → ROI crop (optional)
  → Xử lý ảnh (resize, grayscale, threshold, edge, mask, quality info)
  → Tính brightness + blur_score
  → Ghi metadata  → image_metadata.csv
  → Sinh event    → image_event_log.csv
  → Ghi tham số  → parameter_experiment_log.csv
  → 🔔 Motion Notification (banner + toast + âm thanh)
  → Dashboard (http://127.0.0.1:8000 | http://127.0.0.1:8001)
```

---

## Cấu trúc repository

```
lab6_cv_as_iot_ready_code/
├── lab6_cv_as_iot_sensor/              # Lab 6 cơ bản
│   ├── app.py                          # Backend FastAPI
│   ├── index.html                      # Dashboard
│   ├── run_lab6_demo.py                # Smoke test (không cần camera)
│   ├── requirements.txt
│   ├── data/
│   │   ├── raw_images/                 # Ảnh gốc
│   │   ├── processed_images/           # Contact sheet 4 bước
│   │   └── videos/                     # Video ngắn
│   ├── outputs/
│   │   ├── image_metadata.csv
│   │   └── image_event_log.csv
│   └── docs/
│       ├── PHAN_TICH_CODE_LAB6.md
│       ├── RUBRIC_LAB6.md
│       ├── CAU_HOI_HIEU_BAN_CHAT.md
│       ├── CHECKLIST_NOP_BAI.md
│       └── HUONG_DAN_CHAY_VA_QUAN_SAT.md
│
└── lab6_cv_as_iot_sensor_advanced/     # Lab 6 nâng cao (Parameter Experiment)
    ├── app.py                          # Backend FastAPI nâng cao
    ├── index.html                      # Dashboard nâng cao
    ├── run_lab6_advanced_demo.py       # Smoke test 11 bước
    ├── requirements.txt
    ├── data/
    │   ├── raw_images/
    │   ├── processed_images/           # Contact sheet 6 bước
    │   └── videos/
    ├── outputs/
    │   ├── image_metadata.csv
    │   ├── image_event_log.csv
    │   └── parameter_experiment_log.csv   # ← điểm cốt lõi của phần nâng cao
    └── docs/
        ├── HUONG_DAN_CHAY_VA_QUAN_SAT_NANG_CAO.md
        ├── CAU_HOI_VA_YEU_CAU_THAY_DOI_THAM_SO.md
        └── PHAN_TICH_CODE_NANG_CAO.md
```

---

## Lab 6 Cơ bản

### Mục tiêu

- Chạy camera stream từ laptop hoặc IP camera (có fallback stream mô phỏng)
- Chụp snapshot, ghi video ngắn, motion capture
- Tạo contact sheet 4 bước: **resize → grayscale → threshold → edge**
- Ghi `image_metadata.csv` và `image_event_log.csv`
- Quan sát toàn bộ pipeline trên dashboard

### Cài đặt và chạy

```bash
cd lab6_cv_as_iot_sensor

python -m venv .venv
# Windows
.venv\Scripts\activate
# macOS/Linux
source .venv/bin/activate

pip install -r requirements.txt
```

**Smoke test (không cần camera):**
```bash
python run_lab6_demo.py
```
Kiểm tra `RUN_TEST_LOG.txt` — dòng đầu phải là `LOCAL_PIPELINE_TEST_PASS`.

**Khởi động server:**
```bash
uvicorn app:app --reload --host 0.0.0.0 --port 8000
```

Mở trình duyệt: **http://127.0.0.1:8000/**
<img width="1830" height="903" alt="image" src="https://github.com/user-attachments/assets/0313073e-a395-4454-9046-c8061791ccd3" />


### API endpoints

| Endpoint | Mô tả |
|---|---|
| `GET /` | Dashboard HTML |
| `GET /video_feed?source=0` | MJPEG stream |
| `GET /snapshot?source=0` | Chụp 1 ảnh, chạy pipeline |
| `GET /record-video?seconds=5` | Ghi video ngắn |
| `GET /motion-capture?threshold=25&min_area=800` | Motion detection |
| `POST /upload-image` | Upload ảnh vào pipeline |
| `GET /metadata` | Đọc image_metadata.csv |
| `GET /events` | Đọc image_event_log.csv |
| `GET /latest` | Ảnh và event mới nhất |
| `GET /health` | Kiểm tra trạng thái server |
| `GET /docs` | Swagger UI |

### Camera source

| Giá trị | Ý nghĩa |
|---|---|
| `0` | Camera laptop (webcam mặc định) |
| `http://192.168.x.x:8080/video` | IP camera (IP Webcam app) |
| `rtsp://...` | RTSP stream |
| *(không có camera)* | Tự fallback sang stream mô phỏng |

---

