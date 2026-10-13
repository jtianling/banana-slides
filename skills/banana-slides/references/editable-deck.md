# 分层清单

完整校验规则在 `../scripts/editable-deck.schema.json`. 顶层包含 title, 可选 aspect_ratio (`16:9` 或 `4:3`), 以及按页序排列的 slides. 每页包含 title, 可选 notes 和六位十六进制 background, 以及按从后到前排列的 elements. title 是元数据, 要在页面显示标题必须创建 text 元素.  

所有元素都包含页内唯一 id, type 和 box. box 是 `[x, y, width, height]`, 坐标相对于整页, 取值 0 到 1, 宽高至少 0.001, 不能超出页面. 例如 `[0.1, 0.1, 0.8, 0.2]` 表示距离左上各 10%, 宽 80%, 高 20%. 原始识别结果是像素坐标时, 分别除以原图宽高转换; 不把像素数直接写入清单.  

| type | 必需内容 | 说明 |
| --- | --- | --- |
| text | text | 原生文本框, 换行用 JSON 的 `\n` |
| image | image | 相对清单目录的 PNG, JPEG 或单帧 WebP 路径, 等比居中 |
| shape | shape | rect, ellipse 或 arrow, 可设 fill, stroke, stroke_width |
| table | rows | 矩形二维字符串数组, 每行单元格数相同 |
| chart | chart_type, categories, series | 原生数据图表, 支持 column, line, pie |

text 和 table 可设 font, font_size, color, bold, italic, align. font 默认 Arial, 使用运行环境实际安装且适合目标语言的字体; font_size 单位 pt, 默认 24. align 为 left, center 或 right. color, background, fill, stroke 均为六位十六进制, 不加 `#`. 文本框无内边距, 从顶部排版, 支持换行; 字体不嵌入文件, 打开端缺字库可能替换字体.  

表格所有单元格使用同一组文字样式, 当前不支持合并单元格和逐格配色. 简单图形与标签应分成独立对象. 图表 categories 为字符串数组, series 为 `[{"name":"系列名称","values":[3,7]}]`, 每组数值长度等于类别数. 不接受非有限数或绝对值大于 1e100 的值, 类别和系列名称不能以 `=` 或 `{=` 开头. 饼图只能有一组非负数据且总和大于零. 不通过截图绕过当前图表类型的限制.  

以下是无需生图即可运行的单页示例:  

```json
{
  "title": "可编辑演示",
  "aspect_ratio": "16:9",
  "slides": [
    {
      "title": "交付质量",
      "background": "102038",
      "notes": "数值为演示用示例, 不代表实际业务数据.",
      "elements": [
        {
          "id": "title",
          "type": "text",
          "box": [0.07, 0.08, 0.86, 0.18],
          "text": "交付质量",
          "font_size": 32,
          "color": "FFFFFF",
          "bold": true
        },
        {
          "id": "body",
          "type": "text",
          "box": [0.07, 0.32, 0.40, 0.52],
          "text": "文字可以直接修改.\n图表保留原始数据.",
          "font_size": 22,
          "color": "FFFFFF"
        },
        {
          "id": "chart",
          "type": "chart",
          "box": [0.53, 0.32, 0.40, 0.52],
          "chart_type": "column",
          "categories": ["第一期", "第二期"],
          "series": [{"name": "示例数量", "values": [3, 7]}]
        }
      ]
    }
  ]
}
```

使用整页背景时, 将 `{"id":"background","type":"image","image":"assets/background.png","box":[0,0,1,1]}` 放在 elements 首项, 背景必须不含需要编辑的文字. 独立插画使用单独的 image 元素和对应 box, 不烘焙到文字层.  

程序拒绝只有图片或装饰形状的页面, 至少需要 text, table 或 chart. 这只是最低结构检查, 不保证文字完整或布局无溢出; 交付前仍要逐项对照原始内容, 并在可用时渲染检查.  
