# python-docx 简历定制陷阱

## 1. 中文引号导致run-level替换失败

**问题**：中文简历中常有中文引号（""''），与Python字符串引号冲突。且中文文本常被拆分到多个`<w:r>` run中（粗体/普通/颜色各一个run），导致run-level的 `old[:25] in run.text` 匹配失败。

**解决方案**：paragraph-level fallback
```python
# 先尝试run-level
for run in p.runs:
    if old[:20] in run.text:
        run.text = run.text.replace(old, new)
        break
else:
    # fallback: 整段替换
    full = p.text
    if old in full:
        new_full = full.replace(old, new)
        p.runs[0].text = new_full
        for r in p.runs[1:]:
            r.text = ""
```

**验证**：替换后必须用 `Document(dst)` 重新读取文件验证，不能只看替换计数。

## 2. ATS高危词必须中英文双语清洗

**高危词清单**（中文）：慕课、内训师、培训体系、人才发展
**高危词清单**（英文）：e-learning、internal trainer、talent development、Training Ecosystem

**安全替换**：
- 培训体系 → 业务赋能体系
- 慕课平台 → 标准化营运SOP
- 内训师体系 → 一线业务赋能机制
- 大型培训 → 大型业务落地宣导
- e-learning platform → operational SOP platform
- Training Ecosystem → Business Enablement System

## 3. 必须验证替换生效

每次定制简历后必须做完整验证：
```python
doc2 = Document(dst)
ats_bad = []
for p in doc2.paragraphs:
    t = p.text.lower()
    for w in ["慕课","内训师","培训体系","人才发展","e-learning","internal trainer","talent development"]:
        if w in t and "业务赋能" not in p.text:
            ats_bad.append(w)
assert not ats_bad, f"ATS words remaining: {ats_bad}"
```

## 4. 文件被锁定时的处理

Word打开文件时python-docx无法写入（PermissionError）。解决方案：存为新文件名（如加后缀 `-内推版`），不要尝试删除旧文件。
