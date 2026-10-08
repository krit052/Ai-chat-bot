# 🎓 MFU Research Grant Disbursement Advisor

แชทบอทที่ปรึกษาด้านการเบิกจ่ายเงินทุนสนับสนุนการวิจัย มหาวิทยาลัยแม่ฟ้าหลวง (MFU) พัฒนาด้วยเทคนิค **RAG (Retrieval-Augmented Generation)** โดยค้นหาข้อมูลจากเอกสารระเบียบการเบิกจ่ายของมหาวิทยาลัย แล้วให้ Large Language Model ตอบคำถามโดยอ้างอิงจากเอกสารเท่านั้น

> โปรเจคนี้เป็นส่วนหนึ่งของรายวิชา Big Data Analytics (BDA) — Project 2, Group 9

## สมาชิกกลุ่ม

| รหัสนักศึกษา | ชื่อ |
|---|---|
| 6631501028 | CHAT JAISAN |
| 6631501041 | TANAKRIT SOMBOON |
| 6631501047 | TANAWAT KEUNKAEW |
| 6631501065 | NANTAWUT PANAN |

## ความสามารถของระบบ

- 💬 หน้าต่างแชทถาม-ตอบภาษาไทย พร้อมจำประวัติการสนทนาภายใน session
- 📄 ตอบคำถามโดยอ้างอิงจากเอกสาร PDF ระเบียบการเบิกจ่ายเงินทุนวิจัยของ มฟล. (ห้ามเดาข้อมูลเอง)
- 💡 ปุ่มคำถามแนะนำ เช่น "อาจารย์จะตั้งงบวิจัยอย่างไร" และ "ผู้วิจัยจะได้รับเงินเมื่อไหร่"
- ⚡ แคชฐานข้อมูลเวกเตอร์ด้วย `st.cache_resource` ทำให้สร้าง index เพียงครั้งเดียวต่อการรันแอป

## สถาปัตยกรรม (RAG Pipeline)

```
PDF documents ──► PyPDFLoader ──► RecursiveCharacterTextSplitter
                                   (chunk_size=1000, overlap=150)
                                              │
                                              ▼
                              HuggingFaceEmbeddings (BAAI/bge-m3)
                                              │
                                              ▼
                                     FAISS Vector Store
                                              │
User question ──► Retriever (top-k = 3) ◄─────┘
                         │
                         ▼
        System prompt + Context + Chat history
                         │
                         ▼
     Hugging Face Inference API (Qwen/Qwen2.5-72B-Instruct)
                         │
                         ▼
                 Answer shown in Streamlit
```

| องค์ประกอบ | เทคโนโลยีที่ใช้ |
|---|---|
| Web UI | [Streamlit](https://streamlit.io/) |
| Document loader / text splitter | LangChain (`langchain-community`, `langchain-text-splitters`) |
| Embedding model | `BAAI/bge-m3` (รองรับหลายภาษา รวมถึงภาษาไทย) — ใช้ GPU อัตโนมัติถ้ามี CUDA |
| Vector database | FAISS (`faiss-cpu`) |
| LLM | `Qwen/Qwen2.5-72B-Instruct` ผ่าน `huggingface_hub.InferenceClient` (`temperature=0.3`, `max_tokens=512`) |

## โครงสร้างโปรเจค

```
.
├── .devcontainer/
│   └── devcontainer.json        # ตั้งค่า Dev Container / GitHub Codespaces
├── app.py                       # แอป Streamlit หลัก (RAG chatbot)
├── requirements.txt             # Python dependencies
├── fund_service_steps.pdf       # เอกสารอ้างอิง: ขั้นตอนการให้บริการทุน
├── learning_dev_fund_2023.pdf   # เอกสารอ้างอิง: ระเบียบทุน ฉบับ 2023
├── learning_dev_fund_eng.pdf    # เอกสารอ้างอิง: ระเบียบทุน ฉบับภาษาอังกฤษ
└── README.md
```

## การติดตั้งและใช้งาน

### สิ่งที่ต้องมี

- Python 3.11 (แนะนำ)
- Hugging Face Access Token ที่ใช้งาน Inference API ได้ — สร้างได้ที่ <https://huggingface.co/settings/tokens>

### 1. Clone และติดตั้ง dependencies

```bash
git clone https://github.com/Tanawat-ak/-BDA-Project2-Group9-.git
cd -- -BDA-Project2-Group9-

python -m venv .venv
# Windows
.venv\Scripts\activate
# macOS / Linux
source .venv/bin/activate

pip install -r requirements.txt
pip install streamlit   # streamlit ไม่ได้อยู่ใน requirements.txt
```

### 2. ตั้งค่า Hugging Face Token

แอปอ่าน token จาก Streamlit secrets ให้สร้างไฟล์ `.streamlit/secrets.toml`:

```toml
HF_TOKEN = "hf_xxxxxxxxxxxxxxxxxxxxxxxx"
```

> ⚠️ อย่า commit ไฟล์ `secrets.toml` ขึ้น Git หากนำขึ้น Streamlit Community Cloud ให้ตั้งค่า `HF_TOKEN` ในเมนู **App settings → Secrets** แทน

### 3. รันแอป

```bash
streamlit run app.py
```

เปิดเบราว์เซอร์ที่ <http://localhost:8501> — การรันครั้งแรกจะใช้เวลาสักครู่เพื่อดาวน์โหลดโมเดล `BAAI/bge-m3` และสร้าง FAISS index

### ใช้งานผ่าน GitHub Codespaces / Dev Container

เปิดโปรเจคใน Codespaces หรือ VS Code Dev Container ระบบจะติดตั้ง dependencies และรัน `streamlit run app.py` ให้อัตโนมัติ พร้อม forward พอร์ต `8501` (ยังต้องตั้งค่า `HF_TOKEN` ตามขั้นตอนที่ 2)

## การเพิ่ม/เปลี่ยนเอกสารอ้างอิง

วางไฟล์ PDF ใหม่ไว้ในโฟลเดอร์โปรเจค แล้วเพิ่มชื่อไฟล์ในรายการ `pdf_files` ภายในฟังก์ชัน `initialize_vector_store()` ใน [app.py](app.py) จากนั้นรีสตาร์ทแอปเพื่อสร้าง index ใหม่

## ข้อจำกัด

- คุณภาพคำตอบขึ้นกับเนื้อหาในเอกสาร PDF และผลการค้นคืน 3 chunk ที่ใกล้เคียงที่สุด
- ต้องเชื่อมต่ออินเทอร์เน็ตเพื่อเรียก Hugging Face Inference API และอาจติด rate limit ตามสิทธิ์ของบัญชี
- คำตอบจากแชทบอทเป็นข้อมูลประกอบเท่านั้น ควรตรวจสอบกับระเบียบฉบับจริงหรือหน่วยงานที่เกี่ยวข้องก่อนดำเนินการ
