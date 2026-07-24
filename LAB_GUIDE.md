# LAB GUIDE — K3 Ngày 1: Khám Phá LLM API
## Hướng dẫn chi tiết từng bước | 9h00–13h00

Tài liệu này dắt bạn qua từng bước của buổi lab. Mỗi block kết thúc bằng một
**CHECKPOINT** có mốc giờ — nếu đến giờ mà bạn chưa xong, đọc mục
**"Nếu bạn bị chậm"** để biết mức tối thiểu cần đạt trước khi đi tiếp.

Toàn bộ code viết trong `template.py`. Project gọi **Google Gemini API**
bằng SDK chính thức `google-genai`.
Toàn bộ test chạy bằng mock — **không tốn tiền API khi chạy pytest**.

> 💡 **Quy tắc quan trọng nhất của buổi lab:** import `genai` **bên trong hàm**
> (`from google import genai` nằm trong thân hàm, không nằm đầu file).
> Lý do: các bài test thay thế (mock) `google.genai.Client` — nếu bạn import ở đầu
> file, hàm của bạn giữ tham chiếu đến class thật và test sẽ gọi API thật
> → fail vì không có key.

---

# 🕘 9h00–10h00 · Mở Đầu & Setup

Giảng viên giới thiệu tổng quan (10'). Song song, bạn setup môi trường:

**Bước 1.** Mở terminal tại thư mục lab, tạo môi trường ảo và cài thư viện.

macOS / Linux:
```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

Windows (PowerShell):
```powershell
python -m venv .venv
.venv\Scripts\Activate.ps1
pip install -r requirements.txt
```

Dấu hiệu venv đã bật: đầu dòng lệnh hiện `(.venv)`. Nếu PowerShell chặn
script, chạy một lần `Set-ExecutionPolicy -ExecutionPolicy RemoteSigned
-Scope CurrentUser`, hoặc dùng Command Prompt: `.venv\Scripts\activate.bat`.

**Bước 2.** Tạo Gemini API key tại
[Google AI Studio](https://aistudio.google.com/app/apikey), rồi thiết lập qua
file `.env`:
```bash
cp .env.example .env             # Windows: copy .env.example .env
```
Mở file `.env` vừa tạo, thay `your-gemini-api-key-here` bằng key thật:
```dotenv
GEMINI_API_KEY=AIza...key-cua-ban
LAB_MODEL=gemini-2.5-pro
LAB_MINI_MODEL=gemini-2.5-flash
```
`template.py` đã gọi sẵn `load_dotenv()` và ánh xạ cấu hình cho client tương
thích, nên không cần `export`.
Key chỉ cần cho phần **chạy thật** (demo, exercises); pytest không cần key.
`.env` đã nằm trong `.gitignore` — không bao giờ commit key.

> Xem hướng dẫn tạo và kiểm tra Gemini key chi tiết ở
> [Phụ lục B](#phụ-lục-b--cấu-hình-google-gemini-api).

**Bước 3.** Chạy thử bộ test:
```bash
pytest tests/ -v
```

### ✅ CHECKPOINT 0 (10h00)
Lệnh trên phải **chạy được và báo fail hàng loạt** với thông báo
`NotImplementedError` — đó là dấu hiệu môi trường đã đúng, chỉ còn thiếu code
của bạn. Nếu gặp `ModuleNotFoundError: No module named 'google.genai'` → môi trường
ảo chưa activate hoặc chưa `pip install`.

---

# 🕘 10h00–10h40 · BLOCK 1: API Cơ Bản

### Mục tiêu
- Gọi Chat Completions API, đo độ trễ
- Hiểu tham số `model`, `temperature`, `top_p`, `max_tokens`
- So sánh Gemini 2.5 Pro với Gemini 2.5 Flash về chất lượng / độ trễ / chi phí

### Kiến thức nền (giảng viên demo 10')

Một lời gọi Gemini cơ bản bằng SDK chính thức:

```python
from google import genai

client = genai.Client(api_key=os.getenv("GEMINI_API_KEY"))
response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents="Xin chào!",
    config={
        "temperature": 0.7,
        "top_p": 0.9,
        "max_output_tokens": 256,
    },
)
text = response.text
```

Ví dụ chạy sẵn để tham khảo thêm: [Google Colab của khóa](https://colab.research.google.com/drive/172zCiXpLr1FEXMRCAbmZoqTrKiSkUERm?usp=sharing)

### Task 1.1 — `call_gemini` (~20')

**Bước 1.** Mở `template.py`, tìm hàm `call_gemini`. Đọc kỹ docstring —
chữ ký hàm và kiểu trả về là "hợp đồng" mà test sẽ kiểm tra, đừng sửa chúng.
**Bước 2.** Xóa dòng `raise NotImplementedError(...)`, viết phần thân:
```python
from google import genai           # import TRONG hàm
client = genai.Client(api_key=os.getenv("GEMINI_API_KEY"))
```

**Bước 3.** Đo thời gian quanh lời gọi API — `latency` là thời gian **chỉ của
lời gọi mạng**, nên `time.time()` phải nằm sát trước và sau `create(...)`:
```python
start = time.time()
response = client.models.generate_content(
    model=model,
    contents=prompt,
    config={"temperature": temperature, "top_p": top_p,
            "max_output_tokens": max_tokens},
)
latency = time.time() - start
```

**Bước 4.** Trả về tuple `(text, latency)`:
```python
return response.text, latency
```

**Bước 5.** Kiểm tra ngay (đừng đợi xong hết mới test):
```bash
pytest tests/test_part1.py -k CallGemini -v
```

### Task 1.2 — `call_gemini_flash` (~5')

**Bước 1.** Hàm này chỉ là "phím tắt" gọi model rẻ hơn — tái sử dụng Task 1.1,
đừng copy-paste code:
```python
return call_gemini(prompt, model=GEMINI_FLASH_MODEL,
                   temperature=temperature, top_p=top_p, max_tokens=max_tokens)
```
Tái sử dụng nghĩa là: sau này sửa `call_gemini` một chỗ, cả hai model đều
hưởng lợi.

### Task 1.3 — `compare_models` (~15')

**Bước 1.** Gọi lần lượt hai hàm trên với cùng `prompt`:
```python
gemini_text, gemini_latency = call_gemini(prompt)
flash_text, flash_latency = call_gemini_flash(prompt)
```

**Bước 2.** Ước tính chi phí output của model chính. Ở block này ta dùng ước lượng
thô "0.75 từ ≈ 1 token" (Block 2 sẽ tính chính xác bằng tiktoken):
```python
cost = (len(gemini_text.split()) / 0.75) / 1000 \
       * PRICING_PER_1K_TOKENS[GEMINI_MODEL]["output"]
```

**Bước 3.** Ghép dict đúng 5 key như docstring (`gemini_response`,
`flash_response`, `gemini_latency`, `flash_latency`, `gemini_cost_estimate`).
Tên key phải khớp từng ký tự — test so sánh chính xác.

### ✅ CHECKPOINT 1 (10h40)
```bash
pytest tests/test_part1.py -v
```
Kỳ vọng: **10 passed** —
```
tests/test_part1.py::TestCallGemini::test_returns_non_empty_string PASSED
...
========================= 10 passed in ~1s =========================
```
Nếu có API key, chạy thử thật để cảm nhận độ trễ hai model:
```bash
python -c "from template import compare_models; \
           print(compare_models('Việt Nam có bao nhiêu tỉnh?'))"
```
Sau đó trả lời **Câu 1.1 → 1.3** trong `exercises.md`.

**Nếu bạn bị chậm:** tối thiểu Task 1.1 phải pass (`-k CallGemini`) rồi sang
Block 2 — Task 1.2/1.3 quay lại làm trong giờ wrap-up. Block 2 và 3 không
phụ thuộc Task 1.3.

---

# 🕘 10h40–11h20 · BLOCK 2: System Prompt & Token

### Mục tiêu
- Dùng `system_instruction` để định persona cho model
- Ước lượng token bằng `tiktoken` và fallback cho model Gemini
- Tính chi phí tách bạch input / output

### Kiến thức nền (giảng viên demo 10')

Gemini nhận chỉ dẫn hệ thống qua `config.system_instruction`; lịch sử hội
thoại nằm riêng trong `contents`:

```python
contents = "Giải thích blockchain là gì?"
config = {"system_instruction": "Bạn là giáo viên tiểu học..."}
```

Chi phí API tính theo **token**, không theo từ, và giá input khác giá output
(xem `PRICING_PER_1K_TOKENS` trong template). `tiktoken` được giữ lại vì
bộ test của bài lab sử dụng nó; với model Gemini, kết quả chỉ là **ước lượng**
vì tokenizer thực tế của Gemini khác tokenizer mà `tiktoken` sử dụng.

### Task 2.1 — `chat_with_system_prompt` (~15')

**Bước 1.** Copy cấu trúc `call_gemini` của bạn (import trong hàm, đo giờ,
trả tuple) — điểm khác là thêm `system_instruction` vào config:
```python
contents=user_prompt,
config={"system_instruction": system_prompt, ...}
```

**Bước 2.** Chạy `pytest tests/test_part2.py -k SystemPrompt -v`. Test sẽ
kiểm tra cả việc nội dung `system_prompt` thực sự được gửi lên — nếu bạn quên
truyền, test chỉ tên lỗi rất rõ.

### Task 2.2 — `count_tokens` (~10')

**Bước 1.** Viết phần "đường vui" (happy path):
```python
import tiktoken
enc = tiktoken.encoding_for_model(model)
return len(enc.encode(text))
```

**Bước 2.** Bọc try/except. `tiktoken` không nhận diện tên model Gemini nên
sẽ raise; hàm tiện ích phải chuyển sang ước lượng dự phòng:
```python
try:
    import tiktoken
    enc = tiktoken.encoding_for_model(model)
    return len(enc.encode(text))
except Exception:
    return max(1, len(text) // 4)   # ước lượng: 1 token ≈ 4 ký tự
```
Test có một case truyền model không tồn tại — chính là để kiểm tra fallback này.

### Task 2.3 — `estimate_cost` (~15')

**Bước 1.** Đếm token hai chiều bằng hàm vừa viết:
```python
input_tokens = count_tokens(prompt, model)
output_tokens = count_tokens(response, model)
```

**Bước 2.** Tra bảng giá và tính. Lưu ý đơn vị là **USD trên 1000 token**:
```python
pricing = PRICING_PER_1K_TOKENS[model]
input_cost = input_tokens / 1000 * pricing["input"]
output_cost = output_tokens / 1000 * pricing["output"]
```

**Bước 3.** Trả dict 5 key: `input_tokens`, `output_tokens`, `input_cost`,
`output_cost`, `total_cost` (= input + output).

### ✅ CHECKPOINT 2 (11h20)
```bash
pytest tests/test_part2.py -v
```
Kỳ vọng: **10 passed**. Thử nhanh với Python REPL:
```python
>>> from template import count_tokens, estimate_cost
>>> count_tokens("Xin chào Việt Nam")
7        # con số có thể khác chút tùy encoding
>>> estimate_cost("câu hỏi dài...", "câu trả lời dài...")["total_cost"]
0.000123...
```
Trả lời **Câu 2.1 → 2.2** trong `exercises.md` (cần API key để chạy so sánh
persona thật).

**Nếu bạn bị chậm:** Task 2.1 là bắt buộc (Block 4 cần system prompt).
Task 2.2/2.3 có thể tạm dùng bản tối giản (chỉ fallback `len(text) // 4`,
chưa có tiktoken) — vẫn pass phần lớn test — rồi hoàn thiện sau.

---

# ☕ 11h20–11h30 · GIẢI LAO

Đứng dậy, rời màn hình. Block 3 cần não tươi.

---

# 🕘 11h30–12h10 · BLOCK 3: Streaming & Độ Bền

### Mục tiêu
- Stream phản hồi token-by-token cho UX tức thời
- Duy trì lịch sử hội thoại có giới hạn
- Retry với exponential backoff khi API lỗi tạm thời

### Kiến thức nền (giảng viên demo 10')

Với `stream=True`, API trả về **iterator các chunk** thay vì một response
trọn vẹn — in ra đến đâu người dùng đọc đến đó:

```python
stream = client.models.generate_content_stream(model=..., contents=..., config=...)
reply = ""
for chunk in stream:
    delta = chunk.text or ""
    print(delta, end="", flush=True)
    reply += delta
```

API thật thỉnh thoảng lỗi tạm thời (quá tải, mạng chập chờn). Chiến lược
chuẩn: thử lại với thời gian chờ **tăng gấp đôi** sau mỗi lần
(0.1s → 0.2s → 0.4s...) để không dồn dập đánh vào server đang nghẽn.

### Task 3.1 — `streaming_chatbot` (~25')

**Bước 1.** Dựng khung vòng lặp trước, chưa cần API:
```python
from google import genai
client = genai.Client(api_key=os.getenv("GEMINI_API_KEY"))
history = []
while True:
    user_msg = input("Bạn: ")
    if user_msg.strip().lower() in ("quit", "exit"):
        break
```

**Bước 2.** Trong vòng lặp, ghép `contents` = history + tin nhắn mới rồi gọi
API với `stream=True`:
```python
contents = history + [{"role": "user", "parts": [{"text": user_msg}]}]
stream = client.models.generate_content_stream(
    model=GEMINI_MODEL, contents=contents,
)
```

**Bước 3.** In từng chunk và gom lại thành `reply` (dùng mẫu ở phần kiến
thức nền — nhớ `or ""` cho chunk cuối).

**Bước 4.** Cập nhật history sau mỗi lượt và **cắt còn 3 lượt cuối**. Một
lượt = 1 message user + 1 message assistant, nên 3 lượt = 6 message:
```python
history.append({"role": "user", "parts": [{"text": user_msg}]})
history.append({"role": "model", "parts": [{"text": reply}]})
history = history[-6:]
```
Vì sao phải cắt? History dài ra mãi thì mỗi lượt sau càng tốn token input —
chi phí tăng theo thời gian trò chuyện.

### Task 3.2 — `retry_with_backoff` (~15')

**Bước 1.** Viết vòng lặp `max_retries + 1` lần thử (lần đầu + các lần retry):
```python
for attempt in range(max_retries + 1):
    try:
        return fn()
    except Exception:
        if attempt == max_retries:
            raise                          # hết lượt → ném lỗi cuối cùng ra
        time.sleep(base_delay * (2 ** attempt))
```
Lưu ý `raise` trần (không tham số) giữ nguyên exception gốc — người gọi biết
chính xác lỗi gì.

### ✅ CHECKPOINT 3 (12h10)
```bash
pytest tests/test_part3.py -v
```
Kỳ vọng: **6 passed**. Nếu có API key, chạy chatbot thật:
```bash
python -c "from template import streaming_chatbot; streaming_chatbot()"
```
Hỏi 2–3 câu liên tiếp và để ý: câu sau có "nhớ" ngữ cảnh câu trước không?
Trả lời **Câu 3.1 → 3.2** trong `exercises.md`.

**Nếu bạn bị chậm:** ưu tiên Task 3.2 (`retry_with_backoff` — ngắn và Block 4
cần nó), phần streaming trong Task 3.1 có thể hoàn thiện ngay trong Block 4
vì mini-project dùng lại đúng kỹ thuật đó.

---

# 🕘 12h10–12h50 · BLOCK 4: MINI-PROJECT — Trợ Lý CLI Hoàn Chỉnh

### Mục tiêu
Ghép **tất cả** những gì đã xây thành một hàm `run_assistant`: persona qua
system prompt + streaming + history + retry + thống kê token/chi phí.

### Thiết kế trước khi code (5')

Đọc docstring `run_assistant` trong `template.py` — nó có sẵn khung sườn.
Ba điểm khác với `streaming_chatbot`:

1. **Đầu vào tiêm được:** đọc input qua tham số `get_input` (mặc định là
   `input`). Nhờ đó test tự động "gõ phím hộ" bạn được — đây là kỹ thuật
   dependency injection bạn sẽ gặp lại suốt khóa.
2. **System prompt cố định:** mọi lời gọi API đều truyền
   `config={"system_instruction": persona}` — persona không bị trôi mất khi
   history bị cắt.
3. **Trả về thống kê** thay vì None — sản phẩm thật cần đo được chi phí.

### Các bước (25')

**Bước 1.** Khởi tạo trạng thái phiên:
```python
if get_input is None:
    get_input = input
from google import genai
client = genai.Client(api_key=os.getenv("GEMINI_API_KEY"))
history, num_turns, total_tokens, total_cost = [], 0, 0, 0.0
```

**Bước 2.** Vòng lặp chính — kiểm tra `max_turns` **trước khi** đọc input
(để `max_turns=0` thoát ngay không chờ gõ phím):
```python
while True:
    if max_turns is not None and num_turns >= max_turns:
        break
    user_msg = get_input()
    if user_msg.strip().lower() in ("quit", "exit"):
        break
```

**Bước 3.** Ghép `contents`; system prompt sẽ được truyền riêng ở `config`:
```python
contents = history + [
    {"role": "user", "parts": [{"text": user_msg}]}
]
```

**Bước 4.** Gọi API qua retry — bọc lời gọi trong lambda để
`retry_with_backoff` gọi lại được khi lỗi:
```python
stream = retry_with_backoff(
    lambda: client.models.generate_content_stream(
        model=GEMINI_MODEL,
        contents=contents,
        config={"system_instruction": persona},
    )
)
```

**Bước 5.** Gom reply từ stream (như Block 3), rồi cập nhật history + cắt
còn 6 message.

**Bước 6.** Cộng dồn thống kê mỗi lượt:
```python
num_turns += 1
total_tokens += count_tokens(user_msg) + count_tokens(reply)
total_cost += estimate_cost(user_msg, reply)["total_cost"]
```

**Bước 7.** Trả về dict 4 key: `num_turns`, `total_tokens`, `total_cost`,
`history`.

### Kiểm tra & demo (10')

```bash
pytest tests/test_part4.py -v          # cả basic + scenario
python template.py                     # demo thật (cần API key)
```

Nhóm test `Scenario` chính là "demo tự động": nó giả lập một cuộc hội thoại
nhiều lượt và kiểm tra stats, history, stream — đây là 15 điểm demo của bạn.

### ✅ CHECKPOINT 4 (12h50)
```bash
pytest tests/test_part4.py -v
```
Kỳ vọng: **9 passed** (4 Basic + 5 Scenario).
Trả lời **Câu 4.1 → 4.2** trong `exercises.md`.

**Nếu bạn bị chậm:** làm đúng thứ tự Bước 1 → 2 → 7 trước (vòng lặp + thoát
+ trả dict) — chỉ vậy đã pass nhóm Basic (15đ). Phần API/stream (Bước 3–6)
thêm sau để lấy nhóm Scenario.

---

# 🕘 12h50–13h00 · WRAP-UP & NỘP BÀI

**Bước 1.** Rà lại `exercises.md` — đủ 9 câu chưa?

**Bước 2.** Chấm điểm tự động:
```bash
python grade.py
```
Đọc bảng điểm — mục nào chưa tối đa thì biết chính xác cần sửa gì.

**Bước 3.** Đóng gói và nộp theo [README.md](README.md#hướng-dẫn-nộp-bài):
copy vào `solution/`, zip, đổi tên `<mã sinh viên>_lab_1.zip`, upload LMS.

---

## Phụ Lục A — Lỗi Thường Gặp

| Triệu chứng | Nguyên nhân | Cách sửa |
|---|---|---|
| Test fail dù code "chạy thật" được | Import `genai` ở đầu file | Chuyển `from google import genai` vào **trong** hàm |
| `AuthenticationError` khi chạy pytest | Code đang gọi API thật thay vì mock | Cùng nguyên nhân trên — mock không "bắt" được import đầu file |
| `AuthenticationError` khi chạy demo | Gemini key thiếu hoặc sai | Kiểm tra `GEMINI_API_KEY` trong `.env`, không thêm dấu nháy/khoảng trắng |
| `NotFoundError` / model không tồn tại | Sai model ID | Dùng `gemini-2.5-pro` hoặc `gemini-2.5-flash` |
| `KeyError: 'gemini_response'` | Tên key trong dict gõ sai | So từng ký tự với docstring |
| Chunk cuối làm crash (`TypeError: ... NoneType`) | Quên `or ""` khi đọc `chunk.text` | `delta = chunk.text or ""` |
| History phình to, chi phí tăng dần | Quên cắt history | `history = history[-6:]` sau mỗi lượt |
| `StopIteration` trong test scenario | Đọc input nhiều hơn số lượt kịch bản | Kiểm tra `max_turns` **trước** khi `get_input()` |
| tiktoken treo/lỗi khi offline | Lần đầu cần mạng để tải encoding | Fallback `max(1, len(text) // 4)` trong try/except |

---

## Phụ Lục B — Cấu Hình Google Gemini API

Project dùng SDK chính thức `google-genai`; chỉ cần API key, không cần cấu
hình base URL.

### Bước 1 — Tạo API key

1. Mở [Google AI Studio — API Keys](https://aistudio.google.com/app/apikey).
2. Đăng nhập tài khoản Google và chọn **Create API key**.
3. Copy key và không chia sẻ hoặc commit key lên Git.

### Bước 2 — Cấu hình `.env`

Mở `.env` và điền:

```dotenv
GEMINI_API_KEY=AIza...key-cua-ban
LAB_MODEL=gemini-2.5-pro
LAB_MINI_MODEL=gemini-2.5-flash
```

Cặp model trên lần lượt đóng vai model chất lượng cao và model nhanh/tiết
kiệm của Block 1.

### Bước 3 — Kiểm tra key hoạt động

```bash
python -c "
from template import call_gemini
text, latency = call_gemini('Chào bạn, hãy trả lời bằng 1 câu tiếng Việt.')
print(f'[{latency:.2f}s] {text}')
"
```

Thấy câu trả lời tiếng Việt in ra là xong — làm tiếp lab như bình thường.

### Lưu ý khi dùng Gemini

- **pytest và `python grade.py` không cần key** — mọi test đều mock, nên
  điểm số không phụ thuộc API hoặc hạn mức Gemini.
- `count_tokens` không có bảng mã tiktoken cho model Gemini → tự động rơi về ước
  lượng `len(text) // 4` (đúng như thiết kế fallback ở Task 2.2).
- Bảng giá trong `template.py` chỉ phục vụ bài tập ước tính; hãy kiểm tra
  trang giá Gemini chính thức trước khi dùng con số cho sản phẩm thật.
- Nếu gặp lỗi 429 (hết hạn mức tạm thời) — chính là lúc `retry_with_backoff`
  của Task 3.2 tỏa sáng.
