# Smart QA System with Neo4j & BERT

本项目是一个 **水电气领域知识库转化与智能问答系统** 的复现版本，核心目标是：
1. 将原始资料（PDF、Word、图片等）转化为结构化数据，构建 **Neo4j 知识图谱**。
2. 基于 **BERT 模型** 实现面向知识图谱的智能问答，支持模糊匹配、上下文理解与实指/异指消解。
3. 提供 **FastAPI 服务接口**，支持查询图数据库、实时数据联通（如冷冻水泵运行参数查询）。

---

## 🚀 技术栈
- **Graph Database**: Neo4j  
- **NLP Model**: BERT (transformers)  
- **Framework**: FastAPI  
- **Data Processing**: PyPDF2 / docx / OCR (PaddleOCR / Tesseract)  
- **Vector Search**: Sentence-BERT embeddings + cosine similarity  
- **Deployment**: Docker (可选)

---

## ⚡ 技术难点与解决方案

### 1. 原始资料的多模态转化
- **难点**: 项目中资料类型复杂（PDF、Word、图片扫描件），文本质量参差不齐。  
- **解决**:
  - 对 PDF/Word 使用 `pdfplumber`、`python-docx` 提取文本。  
  - 对图片资料使用 OCR（PaddleOCR/Tesseract）转文字。  
  - 对提取文本进行 **NLP 分句 + 实体识别（NER）**，筛选出关键设备、参数、运行逻辑。

---

### 2. 知识打标签与图谱转化
- **难点**: 如何将“冷冻水泵运行参数”这类非结构化描述转化为图数据库节点与关系。  
- **解决**:
  - 使用规则 + NLP Pipeline 自动给资料打标签（设备名、部件、工艺参数）。  
  - 定义领域本体（Ontology），统一节点和关系类型，如：
    - 节点：设备 (Pump)、能源 (Electricity)、参数 (Pressure)  
    - 关系：`HAS_PARAM`, `CONNECTS_TO`, `BELONGS_TO_SYSTEM`  
  - 最终用 Neo4j 承载图谱。

---

### 3. 智能问答（BERT + 图数据库检索）
- **难点**:  
  - 用户问题语义复杂，如“这台水泵的电流情况如何？” → 需要解析“这台”指代哪台水泵。  
  - 实际查询需要同时考虑 **图数据库实体匹配 + BERT 语义检索**。  
- **解决**:
  - 使用 BERT embeddings 计算用户问题与知识库片段的相似度，排序候选答案。  
  - 实现 **上下文管理**（实指/异指消解）：结合历史提问推断“这台”“它”等代词指代的对象。  
  - 结合 Neo4j 查询语句（Cypher），获取结构化结果。

---

### 4. 实时数据与 API 联通
- **难点**: 部分问题需要实时数据（如冷冻水泵当前负载）。  
- **解决**:
  - FastAPI 提供统一入口。  
  - 静态知识问答 → Neo4j 查询。  
  - 动态参数查询 → 调用实时监控系统接口（或模拟数据）。  
  - 结果合并后返回给用户。

---

## 📂 项目结构

smart-qa-neo4j-bert/
├── data/                # 示例原始资料 (pdf/doc/image)
├── preprocessing/       # 文档解析 & OCR & NER
├── graph/               # Neo4j 数据导入脚本
├── qa/                  # BERT-based QA 模块
├── api/                 # FastAPI 接口
├── tests/               # 单元测试
└── README.md

---

## 🔧 快速开始
```bash
# 1. 启动 Neo4j (需本地或Docker)
docker run -d --name neo4j \
  -p7474:7474 -p7687:7687 \
  -e NEO4J_AUTH=neo4j/test neo4j

# 2. 安装依赖
pip install -r requirements.txt

# 3. 运行 FastAPI
uvicorn api.main:app --reload
```

## 🧩 示例

问: “冷冻水泵的当前运行状态如何？”
答:
	•	来自 Neo4j 知识图谱：冷冻水泵参数节点 → 功率/电流信息。
	•	来自实时数据接口：电流 10.2A，温度 35℃。

