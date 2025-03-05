这段代码定义了一个名为 `EmbeddingService` 的类，其主要功能是将 PDF 文件内容转换为向量表示，并使用 FAISS 库构建向量索引，以便后续进行高效的相似性搜索。以下是对代码逻辑和功能的详细总结：

### 类的初始化
```python
def __init__(self):
    self.model = SentenceTransformer('paraphrase-multilingual-MiniLM-L12-v2')
    self.index_dir = Path("indexes")
    self.index_dir.mkdir(exist_ok=True)
    self.dimension = 384
    self.current_index = None
    self.current_documents = {}
```
- 使用 `SentenceTransformer` 加载多语言模型 `paraphrase-multilingual-MiniLM-L12-v2`，以支持中文文本处理。
- 创建一个名为 `indexes` 的目录，用于存储索引和文档。
- 初始化索引维度为 384，与模型输出维度一致。
- 初始化索引为空，初始化文档为空。

### 辅助方法
1. **`_generate_safe_id`**:
```python
def _generate_safe_id(self, metadata: dict) -> str:
    timestamp = str(int(time.time()))
    file_info = f"{metadata.get('filename', '')}_{timestamp}"
    return hashlib.md5(file_info.encode()).hexdigest()
```
    - 根据文件名和当前时间生成哈希值，用作文件的唯一ID。

2. **`_create_index`**:
```python
def _create_index(self) -> faiss.IndexFlatL2:
    return faiss.IndexFlatL2(self.dimension)
```
    - 创建一个新的 FAISS 索引对象，使用欧几里得距离（L2 距离）。

3. **`_get_index_path`**:
```python
def _get_index_path(self, file_path: str) -> str:
    file_hash = hashlib.md5(file_path.encode()).hexdigest()
    return f"indexes/index_{file_hash}.bin"
```
    - 将`file_path`的哈希值生作为二进制索引文件的文件名。

### 核心方法
1. **`create_embeddings`**:
```python
async def create_embeddings(self, file_path: str, index_dir: str) -> Dict:
    text_chunks = []
    with open(file_path, 'rb') as f:
        pdf_reader = PyPDF2.PdfReader(f)
        for page in pdf_reader.pages:
            text_chunks.append(page.extract_text())
    index = self._create_index()
    vectors = self.model.encode(text_chunks)
    vectors = vectors.astype('float32')
    index.add(vectors)
    file_hash = hashlib.md5(file_path.encode()).hexdigest()
    index_id = f"index_{file_hash}"
    documents = {}
    for i, text in enumerate(text_chunks):
        documents[str(i)] = {
            "text": text,
            "metadata": {
                "page": i + 1,
                "source": file_path
            }
        }
    self._save_index(file_hash, index, documents)
    return {
        "status": "success",
        "index_id": index_id,
        "chunks": len(text_chunks)
    }
```
    - 从指定的 PDF 文件中提取文本内容，按页分割成文本块。
    - 使用 `SentenceTransformer` 模型将文本块转换为向量表示。
    - 创建一个新的 FAISS 索引，并将向量添加到索引中。
    - 生成文件 ID 和文档数据，包括文本内容和元数据。
    - 调用 `_save_index` 方法保存索引和文档数据。
    - 返回操作结果，包括状态、索引 ID 和文本块数量。

2. **`_save_index`**:
```python
def _save_index(self, file_id: str, index: faiss.Index, documents: dict):
    index_path = self.index_dir / f"index_{file_id}.bin"
    docs_path = self.index_dir / f"docs_{file_id}.json"
    faiss.write_index(index, str(index_path))
    with open(docs_path, 'w', encoding='utf-8') as f:
        json.dump(documents, f, ensure_ascii=False, indent=2)
```
    - 将 FAISS 索引保存为二进制文件。
    - 将文档数据保存为 JSON 文件。

3. **`_load_index`**:
```python
def _load_index(self, index_id: str):
    index_path = self.index_dir / f"{index_id}.bin"
    docs_path = self.index_dir / f"docs_{index_id.replace('index_', '')}.json"
    if not index_path.exists() or not docs_path.exists():
        old_index_path = self.index_dir / f"index_{index_id}.bin"
        old_docs_path = self.index_dir / f"docs_{index_id}.json"
        if old_index_path.exists() and old_docs_path.exists():
            index_path = old_index_path
            docs_path = old_docs_path
        else:
            raise FileNotFoundError(f"找不到索引文件: {index_id}")
    self.current_index = faiss.read_index(str(index_path))
    if self.current_index.d != self.dimension:
        raise ValueError(f"索引维度不匹配: 期望 {self.dimension}, 实际 {self.current_index.d}")
    with open(docs_path, 'r', encoding='utf-8') as f:
        self.current_documents = json.load(f)
    if not self.current_documents:
        raise ValueError("文档数据为空")
    print(f"成功加载索引 {index_id}: {self.current_index.ntotal} 个向量, {len(self.current_documents)} 个文档")
```
    - 尝试加载指定索引 ID 的索引文件和文档数据。
    - 支持旧的文件名格式，以确保兼容性。
    - 验证索引维度和文档数据是否有效。
    - 打印加载成功的信息。

4. **`search`**:
```python
async def search(self, query: str, top_k: int = 3) -> List[dict]:
    if not self.current_index:
        raise Exception("未加载索引")
    query_vector = self.model.encode([query], convert_to_tensor=False)
    query_vector = query_vector.astype('float32')
    distances, indices = self.current_index.search(query_vector, top_k)
    results = []
    for i in range(len(indices[0])):
        idx_str = str(int(indices[0][i]))
        if idx_str in self.current_documents:
            results.append({
                "score": float(distances[0][i]),
                "content": self.current_documents[idx_str]["text"],
                "metadata": self.current_documents[idx_str]["metadata"]
            })
    return results
```
    - 检查当前是否加载了索引。
    - 将查询文本转换为向量表示。
    - 在 FAISS 索引中搜索最相似的向量，返回前 `top_k` 个结果。
    - 根据搜索结果从文档数据中提取相关信息，包括得分、文本内容和元数据。
    - 返回搜索结果列表。

### 总结
`EmbeddingService` 类提供了一个完整的解决方案，用于将 PDF 文件内容转换为向量索引，并支持高效的相似性搜索。通过使用 `SentenceTransformer` 模型和 FAISS 库，该类可以处理多语言文本，并提供了保存、加载和搜索索引的功能。
