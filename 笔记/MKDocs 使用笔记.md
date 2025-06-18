# MKDocs 使用笔记

## 环境准备
- **依赖**：需安装Python 3.7+环境（推荐使用虚拟环境）
- **安装命令**：
  ```bash
  pip install mkdocs
  ```
  （若提示权限问题，可尝试加 `--user` 参数或使用虚拟环境）

---

## 快速开始
```bash
# 创建新项目
mkdocs new my-project

# 进入项目目录
cd my-project

# 启动本地服务器
mkdocs serve
# 访问 http://localhost:8000 查看效果
# 按 Ctrl+C 停止服务
```

---

## 核心文件结构
```
my-project/
├── docs/          # 文档源文件目录
│   ├── index.md   # 首页文件（默认）
│   └── ...        # 其他Markdown文件
├── mkdocs.yml     # 配置文件
└── site/          # 生成的静态网站文件（由mkdocs build生成）
```

---

## 配置文件详解（`mkdocs.yml`）

### 基础配置
```yaml
site_name: My Documentation  # 站点名称（必填）
site_url: https://your-site.com  # 站点基础URL（用于SEO）
repo_url: https://github.com/your/repo  # 仓库链接（用于显示编辑按钮）
```

### 导航栏配置
```yaml
nav:
  - 首页: index.md
  - 指南: guide/
    - 安装: installation.md
    - 使用: usage.md
  - 参考: reference.md
  - 关于我们: about.md
```

### 主题配置
```yaml
theme:
  name: material  # 使用mkdocs-material主题
  custom_dir: docs/styles  # 自定义CSS/JS目录
  language: zh-CN  # 语言设置
```

### 插件配置
```yaml
plugins:
  - search  # 启用搜索功能
  - markdownextradata  # 增强Markdown功能
  - git-revision-date-localized  # 显示最后更新时间
```

---

## 主题推荐
### 1. mkdocs-material（推荐）
- **特点**：现代响应式设计，支持暗黑模式，文档导航更直观
- **安装**：
  ```bash
  pip install mkdocs-material
  ```
- **配置示例**：
  ```yaml
  theme:
    name: material
    palette:
      primary: 'indigo'  # 主色调
      accent: 'pink'     # 强调色
  ```

### 2. readthedocs
- **特点**：模仿Read the Docs经典风格，适合技术文档
- **配置**：
  ```yaml
  theme: readthedocs
  ```

### 3. 其他主题
- [mkdocs](https://www.mkdocs.org/user-guide/custom-themes/) 官方主题
- [mkdocs-jacman](https://github.com/tleyden/mkdocs-jacman) 极简主题
- [mkdocs-awesome-pages](https://github.com/revolunet/mkdocs-awesome-pages-plugin) 自动生成导航插件

---

## 高级功能

### 1. 静态资源管理
- 将图片/文件放入 `docs/assets/` 目录
- 调用方式：`![图片](assets/image.png)`

### 2. 代码高亮
- 支持的语言：Python, JavaScript, Go 等
- 配置代码块：
  ```markdown
  ```python
  def hello():
      print("Hello MKDocs!")
  ```

### 3. 数学公式支持（需插件）
- **安装**：
  ```bash
  pip install mkdocs-macros-plugin
  ```
- **配置**：
  ```yaml
  markdown_extensions:
    - pymdownx.math: {}  # 启用数学公式
  ```
- **使用LaTeX语法**：
  ```markdown
  $$ E = mc^2 $$
  ```

---

## 部署发布

### 1. GitHub Pages
```bash
# 配置mkdocs.yml
site_url: https://yourname.github.io/your-repo/

# 部署命令（需安装ghp-publish插件）
mkdocs gh-deploy --force
```

### 2. 其他平台
- **GitLab Pages**：
  ```yaml
  site_dir: public
  ```
  推送到 `master` 分支即可

- **Vercel/Netlify**：
  将 `mkdocs.yml` 和 `docs/` 文件上传，设置构建命令：
  ```bash
  mkdocs build
  ```

---

## 常见问题
1. **主题加载失败**  
   检查是否正确安装主题包：`pip install mkdocs-material`

2. **图片无法显示**  
   确保图片路径正确，使用相对路径或 `assets/` 目录

3. **配置报错**  
   运行 `mkdocs validate` 检查配置文件语法

---

## 推荐资源
1. [官方文档](https://www.mkdocs.org)
2. [Awesome MkDocs](https://github.com/tleyden/awesome-mkdocs) 插件/主题精选
3. [Markdown语法指南](https://www.mkdocs.org/user-guide/writing-your-docs/#markdown-syntax)
4. [Material主题文档](https://squidfunk.github.io/mkdocs-material/)

---

通过以上配置，您可以快速搭建专业级文档网站。建议结合项目需求选择合适的主题，并善用插件扩展功能。