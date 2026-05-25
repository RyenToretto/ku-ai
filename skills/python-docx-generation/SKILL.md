# python-docx 文档生成最佳实践

## 触发条件

以下场景应使用本 Skill：
- 用 Python 从 Markdown / 结构化数据生成 Word 文档（.docx）
- 需要批量修改现有 docx 的字体、行距、页眉页脚
- 遇到中文字体乱码、字上字下、字扁等排版问题

---

## 核心原则

### 1. 中文字体必须同时设四个属性

**最常见错误**：`run.font.name = '宋体'` 只设了 `w:ascii`，中文字符仍 fallback 到系统默认字体，导致字扁、字上字下。

**正确做法**：

```python
from docx.oxml.ns import qn
from docx.oxml import OxmlElement

def set_run_font(run, zh_font='宋体', en_font='Times New Roman', size_pt=12):
    run.font.size = Pt(size_pt)
    rPr = run._r.get_or_add_rPr()
    rFonts = rPr.find(qn('w:rFonts'))
    if rFonts is None:
        rFonts = OxmlElement('w:rFonts')
        rPr.insert(0, rFonts)
    rFonts.set(qn('w:ascii'),    en_font)   # 英文
    rFonts.set(qn('w:hAnsi'),    en_font)   # 英文（高级）
    rFonts.set(qn('w:eastAsia'), zh_font)   # 中文
    rFonts.set(qn('w:cs'),       zh_font)   # 复杂脚本（重要！）
    # 同步 sz / szCs，防止行高抖动
    sz_val = str(int(size_pt * 2))
    for tag in ('w:sz', 'w:szCs'):
        el = rPr.find(qn(tag))
        if el is None:
            el = OxmlElement(tag); rPr.append(el)
        el.set(qn('w:val'), sz_val)
```

**必须设的四个属性说明**：

| 属性 | 作用 | 遗漏后果 |
|---|---|---|
| `w:ascii` | 英文字符字体 | 英文用系统默认 |
| `w:hAnsi` | 英文高级字符 | 部分符号显示异常 |
| `w:eastAsia` | 中日韩字符字体 | 中文 fallback，字扁 |
| `w:cs` | 复杂脚本（含汉字） | 行基线错位，字上字下 |

### 2. sz 和 szCs 必须同步

`sz` 控制显示字号，`szCs` 控制行高基线计算。两者不一致会导致同行内有的字偏上、有的字偏下。

```python
# 错误：只设 font.size，szCs 保持旧值
run.font.size = Pt(12)

# 正确：同步设两个
sz_val = str(int(12 * 2))  # half-points
for tag in ('w:sz', 'w:szCs'):
    el = rPr.find(qn(tag)) or OxmlElement(tag)
    el.set(qn('w:val'), sz_val)
    if rPr.find(qn(tag)) is None: rPr.append(el)
```

### 3. Normal 样式也要设 eastAsia

样式继承链：Normal → 段落样式 → run。如果 Normal 的 eastAsia 没设，所有未显式指定字体的 run 都会用错误字体。

```python
s = doc.styles['Normal']
rPr = s.element.get_or_add_rPr()
rFonts = rPr.find(qn('w:rFonts')) or OxmlElement('w:rFonts')
rFonts.set(qn('w:eastAsia'), '宋体')
rFonts.set(qn('w:cs'),       '宋体')
```

### 4. 行距用 auto + 1.4 倍，不用固定行距

固定行距（`exactOrAtLeast`）在字号不一致时会截断字符。

```python
def fix_para_spacing(para, size_pt=12, before=0, after=120):
    pPr = para._p.get_or_add_pPr()
    sp  = pPr.find(qn('w:spacing'))
    if sp is None:
        sp = OxmlElement('w:spacing'); pPr.append(sp)
    line_val = str(int(size_pt * 2 * 20 * 1.4))  # 1.4 倍，单位 twentieths of a point
    sp.set(qn('w:line'),      line_val)
    sp.set(qn('w:lineRule'), 'auto')   # 关键：auto 而非 exact
    sp.set(qn('w:before'),    str(before))
    sp.set(qn('w:after'),     str(after))
```

### 5. 封面页使用分节符隔离

封面和正文必须在不同 section，否则页眉页脚会互相覆盖。

```python
def page_break_section():
    p_el  = OxmlElement('w:p')
    pPr   = OxmlElement('w:pPr')
    sectPr= OxmlElement('w:sectPr')
    pgT   = OxmlElement('w:type')
    pgT.set(qn('w:val'), 'nextPage')
    sectPr.append(pgT); pPr.append(sectPr); p_el.append(pPr)
    return p_el
# 插入到 body 最前面的封面元素末尾
body.insert(0, page_break_section())
```

### 6. 页脚页码用域代码

不要硬编码页码，要用 Word 域（field）：

```python
def add_page_number_to_run(run):
    def fld(type_): fc=OxmlElement('w:fldChar'); fc.set(qn('w:fldCharType'),type_); return fc
    def instr(t):   el=OxmlElement('w:instrText'); el.set(qn('xml:space'),'preserve'); el.text=t; return el
    run._r.append(fld('begin'))
    run._r.append(instr(' PAGE '))
    run._r.append(fld('end'))
```

### 7. 推荐字体组合（法律/正式文书）

| 用途 | 中文字体 | 英文字体 | 字号 |
|---|---|---|---|
| 正文 | 宋体 | Times New Roman | 12pt |
| 标题 h1/h2 | 黑体 | Arial | 18pt / 15pt |
| 标题 h3/h4 | 黑体 | Arial | 13pt / 12pt |
| 代码块 | Courier New | Courier New | 10pt |
| 页眉/页脚 | 宋体 | Times New Roman | 9pt |
| 表头 | 黑体 | Arial | 10pt |
| 表格正文 | 宋体 | Times New Roman | 10pt |

---

## 批量修复现有 docx 字体

不重建文档、只修复字体的最简方法：

```python
def fix_all_runs(docx_path, old_zh='仿宋', new_zh='宋体'):
    doc = Document(str(docx_path))
    all_paras = list(doc.paragraphs)
    for tbl in doc.tables:
        for row in tbl.rows:
            for cell in row.cells:
                all_paras.extend(cell.paragraphs)
    for sec in doc.sections:
        for part in (sec.header, sec.footer):
            try: all_paras.extend(part.paragraphs)
            except: pass

    for para in all_paras:
        if para.style and 'Heading' in para.style.name:
            continue  # 标题不改
        for r_el in para._p.findall(qn('w:r')):
            rPr = r_el.find(qn('w:rPr'))
            if rPr is None: continue
            rFonts = rPr.find(qn('w:rFonts'))
            if rFonts is None: continue
            for attr in (qn('w:ascii'), qn('w:hAnsi'),
                         qn('w:eastAsia'), qn('w:cs')):
                if rFonts.get(attr,'') == old_zh:
                    rFonts.set(attr, new_zh)
    doc.save(str(docx_path))
```

---

## 常见问题速查

| 问题现象 | 根因 | 修复方式 |
|---|---|---|
| 中文字体扁/压缩 | `eastAsia` 未设，fallback 到 Helvetica Condensed | 设 `w:eastAsia` |
| 字一上一下 | `sz` ≠ `szCs`，或 run 字号不一致 | 同步 `sz` 和 `szCs` |
| 表格线条消失 | `tbl.style = 'Table Grid'` 但 cell 没设边框 | 逐 cell 设 `w:tcBorders` |
| 页眉页脚被覆盖 | 封面和正文在同一 section | 封面末尾插入 `nextPage` 分节符 |
| 页码不刷新 | 硬编码数字 | 改用 `PAGE` / `NUMPAGES` 域代码 |
| 代码块字体继承正文 | 只设 `font.name` 未设 `eastAsia` | 明确设四个属性为 `Courier New` |

---

## 完整可复用模板

参考实现见：`/Users/koujianfeng/Desktop/kjf/temp/李井法仲裁/` 项目中的 Python 脚本（对话记录中生成的 MD→DOCX 转换器），包含：
- `set_run_fmt()` — 设字体+字号+颜色
- `fix_para_spacing()` — 段落行距
- `insert_cover()` — 封面页（XML 级操作）
- `add_header()` / `add_footer()` — 页眉页脚+页码域
- `md_to_docx()` — Markdown 解析器（标题/列表/表格/代码块/引用）
