在 MySQL 的 `command` 参数中使用 `--` 有以下作用：

1. 分隔 `command` 参数和 MySQL 服务器的启动参数：`--` 用于分隔 `command` 参数和 MySQL 服务器的启动参数。通过在 `command` 参数的末尾添加 `--`，可以明确指示 Docker Compose 将后续的参数传递给 MySQL 服务器而不是 Docker 容器本身。
    
2. 指定额外的启动参数：通过在 `command` 参数中使用 `--`，你可以将额外的启动参数传递给 MySQL 服务器。例如，在示例中使用的 `--character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci` 就是额外的启动参数，用于设置 MySQL 服务器的字符集和校对规则。
    

请注意，`--` 的使用是可选的，具体取决于你的需求。如果在 `command` 参数中没有额外的启动参数，你可以省略 `--`。