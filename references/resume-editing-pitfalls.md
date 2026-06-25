# python-docx 简历编辑 Pitfalls

> 2026-06-24 验证

## Pitfall 1: 中文引号导致run-level匹配失败

**现象**：包含中文引号的段落，run-level `old_text[:25] in run.text` 匹配失败。

**原因**：python-docx将粗体/普通/颜色各拆为独立run，中文引号常被拆到不同run中。

**正确做法**：当run-level匹配失败时，用段落级全文替换：
```python
for p in doc.paragraphs:
    if "目标文本片段" in p.text:
        old_full = p.text
        new_full = old_full.replace("旧文本", "新文本")
        p.runs[0].text = new_full
        for r in p.runs[1:]:
            r.text = ""
```

## Pitfall 2: Word锁定文件

`shutil.copy2()` 报 PermissionError 时，存为新文件名让用户手动替换。

## Pitfall 3: 英文逐词替换兜底

英文run也可能拆分。用逐词替换兜底：
```python
for p in doc.paragraphs:
    for run in p.runs:
        for old_word, new_word in word_fixes:
            if old_word in run.text:
                run.text = run.text.replace(old_word, new_word)
```

## 安全替换顺序

1. run-level匹配（快速，保留格式）
2. 段落级全文替换（中文引号场景）
3. 逐词替换兜底（英文单词残留）
4. 必须验证零ATS高危词残留
