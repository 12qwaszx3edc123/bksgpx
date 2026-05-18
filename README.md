# bksgpx

Go 微服务脚手架生成工具 - 一键生成完整的微服务项目结构

## 安装

```bash
go install github.com/12qwaszx3edc123/bksgpx/cmd/mygen@latest
```

## 使用方法

```bash
mygen --name <项目名> --modules <模块列表> [其他参数]
```

### 参数说明

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `--name` | `bks` | 项目名称 |
| `--modules` | - | 模块列表，逗号分隔，如 `user,order,doctor` |
| `--bff` | `h5` | BFF 类型: `h5`, `web`, `applet`, `app` |
| `--db-host` | `127.0.0.1` | 数据库主机 |
| `--db-port` | `3306` | 数据库端口 |
| `--db-user` | `root` | 数据库用户名 |
| `--db-password` | `root` | 数据库密码 |
| `--db-name` | - | 数据库名称 |
| `--template` | 内嵌模板 | 外部模板目录路径 |

### 示例

```bash
# 生成名为 myshop 的项目，包含 user 和 order 模块
mygen --name myshop --modules user,order --db-name myshop_db

# 使用外部模板
mygen --name myproject --modules user --template /path/to/templates
```

## 生成的项目结构

```
<项目名>/
├── common/
│   ├── config/      # 配置管理
│   ├── init/        # 初始化
│   └── model/       # 数据模型
├── proto/           # Protobuf 定义
├── <module>-srv/    # 各模块的 gRPC 服务
├── bff<type>/       # BFF 层 (h5/web/applet/app)
└── pkg/             # 公共工具包
```

## 模板

默认使用内嵌模板，无需额外配置。如需自定义模板，可使用 `--template` 参数指定外部模板目录。

## License

MIT License