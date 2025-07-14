### 环境

<img width="548" height="397" alt="Image" src="https://github.com/user-attachments/assets/9b152fb4-d751-4b1d-96ac-3d10c8cad502" />



```python
python -m venv venv #建立虚拟环境
venv\Scripts\activate #激活虚拟环境

pip install tabula-py
pip install pandas
#安装java，jdk，添加环境变量
pip install tabula-py
pip install pandas openpyxl
```

```python
# pdf_to_excel.py
import tabula
import pandas as pd

# PDF文件路径
pdf_path = "E:/Users/93914/Desktop/123.pdf"

# 从PDF中提取表格
tables = tabula.read_pdf(pdf_path, pages="all", multiple_tables=True)

# 检查是否提取到表格
if tables:
    # 将所有表格合并为一个DataFrame（如果有多个表格）
    df = pd.concat(tables, ignore_index=True)

    # 输出Excel文件路径
    excel_path = "E:/Users/93914/Desktop/ddddd.xlsx"

    # 将DataFrame导出为Excel
    df.to_excel(excel_path, index=False)
    print(f"成功将 {pdf_path} 转换为 {excel_path}")
else:
    print("PDF中未找到表格。")
```

