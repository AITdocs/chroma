# 更新 Chroma 集合中的数据

可以使用 `.update` 方法更新集合中记录的任何属性：

{% TabbedCodeBlock %}

{% Tab label="python" %}
```python
collection.update(
    ids=["id1", "id2", "id3", ...],
    embeddings=[[1.1, 2.3, 3.2], [4.5, 6.9, 4.4], [1.1, 2.3, 3.2], ...],
    metadatas=[{"chapter": 3, "verse": 16}, {"chapter": 3, "verse": 5}, {"chapter": 29, "verse": 11}, ...],
    documents=["doc1", "doc2", "doc3", ...],
)
```
{% /Tab %}

{% Tab label="typescript" %}
```typescript
await collection.update({
    ids: ["id1", "id2", "id3", ...], 
    embeddings: [[1.1, 2.3, 3.2], [4.5, 6.9, 4.4], [1.1, 2.3, 3.2], ...], 
    metadatas: [{"chapter": 3, "verse": 16}, {"chapter": 3, "verse": 5}, {"chapter": 29, "verse": 11}, ...], 
    documents: ["doc1", "doc2", "doc3", ...]
})
```
{% /Tab %}

{% /TabbedCodeBlock %}

如果集合中找不到某个 `id`，则会记录一个错误，并忽略该更新。如果提供了 `documents` 而没有对应的 `embeddings`，则会使用集合的嵌入函数重新计算嵌入。

如果提供的 `embeddings` 的维度与集合的维度不一致，则会抛出异常。

Chroma 还支持 `upsert` 操作，该操作会更新已存在的条目，如果条目不存在则会添加它们。

{% TabbedCodeBlock %}

{% Tab label="python" %}
```python
collection.upsert(
    ids=["id1", "id2", "id3", ...],
    embeddings=[[1.1, 2.3, 3.2], [4.5, 6.9, 4.4], [1.1, 2.3, 3.2], ...],
    metadatas=[{"chapter": 3, "verse": 16}, {"chapter": 3, "verse": 5}, {"chapter": 29, "verse": 11}, ...],
    documents=["doc1", "doc2", "doc3", ...],
)
```
{% /Tab %}

{% Tab label="typescript" %}
```typescript
await collection.upsert({
    ids: ["id1", "id2", "id3"],
    embeddings: [
        [1.1, 2.3, 3.2],
        [4.5, 6.9, 4.4],
        [1.1, 2.3, 3.2],
    ],
    metadatas: [
        { chapter: "3", verse: "16" },
        { chapter: "3", verse: "5" },
        { chapter: "29", verse: "11" },
    ],
    documents: ["doc1", "doc2", "doc3"],
});
```
{% /Tab %}

{% /TabbedCodeBlock %}

如果某个 `id` 在集合中不存在，则会按照 `add` 操作创建对应的条目。具有已有 `id` 的条目则会按照 `update` 操作进行更新。