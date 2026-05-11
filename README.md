# bksgpx - Fixed GPX Templates

这是从 gospacex/gpx v0.0.15 修复后的模板仓库。

## 修复内容

### 1. 移除私有依赖 `github.com/gospacex/logger`
- 原模板使用了未开源的 `gospacex/logger` 包
- 修复：替换为标准库 `log`

### 2. 修复字段命名不一致
- 原模板中 `m.ID` (大写) 与 proto 生成的 `m.Id` (小写) 不匹配
- 修复：统一使用 `m.Id` 与 proto 生成代码保持一致

### 3. 修复路由双 s 问题
- 原模板：`/api/v1/{{.Name}}s` 会生成 `/api/v1/userss`
- 修复：改为 `/api/v1/{{.Name}}`，由用户提供正确的复数形式

### 4. 移除 `goTools.PartialUpdate`
- 原模板引用了不存在的 `goTools` 包
- 修复：改为手动字段赋值

### 5. 移除 `copier.Copy`
- 原模板使用了未导入的 `copier` 包
- 修复：注释掉相关代码

## 模板结构

```
templates/
└── micro-app/
    ├── srv/                    # 微服务模板
    │   ├── repo/
    │   │   └── repository.go.tmpl
    │   ├── service/
    │   │   └── service.go.tmpl
    │   ├── handler/
    │   │   └── handler.go.tmpl
    │   ├── main/
    │   │   └── main_direct.go.tmpl
    │   ├── model/
    │   │   └── model.go.tmpl
    │   └── interceptor/
    │       └── interceptor.go.tmpl
    └── bff/                    # BFF 模板
        ├── router/
        │   ├── gin_router.go.tmpl
        │   └── gin_router_crud.go.tmpl
        └── handler/
            └── handler.go.tmpl
```

## 使用方法

1. 克隆本仓库
2. 将模板复制到你的 gpx 工具模板目录
3. 重新编译 gpx 工具

## 原始来源

- 原始仓库：https://github.com/gospacex/gpx
- 版本：v0.0.15
