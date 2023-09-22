---
title: "配置存储层"
weight: 1
---
Clusterpedia 的`默认存储层`支持 **MySQL** 和 **PostgreSQL** 两种存储组件。

用户在安装 Clusterpedia 时，可以使用已存在的存储组件，
不过需要创建相应的[`默认存储层`配置（ConfigMap）](#默认存储层配置)和[存储组件密码 Secret](#配置存储组件密码-secret)。

## 默认存储层配置
用户需要在 `clusterpedia-system` 命名空间下创建 `clusterpedia-internalstorage` ConfigMap。
```yaml
# internalstorage configmap example
apiVersion: v1
kind: ConfigMap
metadata:
  name: clusterpedia-internalstorage
  namespace: clusterpedia-system
data:
  internalstorage-config.yaml: |
    type: "mysql"
    host: "clusterpedia-internalstorage-mysql"
    port: 3306
    user: root
    database: "clusterpedia"
    connPool:
      maxIdleConns: 10
      maxOpenConns: 100
      connMaxLifetime: 1h
    log:
      slowThreshold: "100ms"
      logger:
        filename: /var/log/clusterpedia/internalstorage.log
        maxbackups: 3
```

internalstorage config 支持以下基本字段:
|field|description|
|-----|-----------|
|`type`|存储组件的类型，支持 "postgres" 和 "mysql" |
|`host`|存储组件地址，可以使用 IP 或者 Service Name|
|`port`|存储组件端口|
|`user`|存储组件用户|
|`password`|存储组件密码|
|`database`|Clusterpedia 所使用的 database|

**存储组件的访问密码，最好存放在 Secret，参考 [配置存储组件密码 Secret](#配置存储组件密码-secret)**

### 数据库连接池配置
|field|description|
|-----|-----------|
|`connPool.maxIdleConns`|空闲连接池中的最大数量，默认为 10|
|`connPool.maxOpenConns`|打开的数据库连接的最大数量，默认为 100|
|`connPool.connMaxLifetime`|连接可以复用的最大时间，默认为 1h |

根据用户的当前环境，合理设置数据库连接池

### 日志配置
支持配置存储层日志，通过 `log` 字段来开启日志打印慢 SQL 和错误
|field|description|
|-----|-----------|
|`log.stdout`|打印日志到标准输出|
|`log.colorful`|是否开启彩色打印|
|`log.slowThreshold`|设置慢 SQL 阀值，例如 "100ms"|
|`log.level`|设置日志级别，支持 Slient, Error, Warn, Info|
|`log.logger`|日志轮滚配置|

开启日志打印后，如果 `log.stdout` 不为 true，则将日志输出到 */var/log/clusterpedia/internalstorage.log* 文件中

#### 日志轮滚配置
将存储层的日志保存到文件中，并且可以配置日志文件的轮滚
|field|description|
|-----|-----------|
|`log.logger.filename`|日志文件路径, 默认为 */var/log/clusterpedia/internalstorage.log* |
|`log.logger.maxsize`|触发日志轮滚的最大文件大小，单位为 MB|
|`log.logger.maxage`|轮滚的旧日志的最大存活时间|
|`log.logger.maxbackups`|轮滚的旧日志的最大数量|
|`log.logger.localtime`|是否为本地时间，默认为 UTC |
|`log.logger.compress`|是否将轮滚的日志文件进行压缩，默认不进行压缩|

#### 关闭日志打印
在 internalstorage config 不填写 `log` 字段，便会忽略日志打印，例如：
```yaml
type: "mysql"
host: "clusterpedia-internalstorage-mysql"
port: 3306
user: root
database: "clusterpedia"
```

### 更多配置
默认存储层还提供了有关 MySQL 和 PostgreSQL 的更多配置，可以参考 [internalstorage/config.go](https://github.com/clusterpedia-io/clusterpedia/blob/main/pkg/storage/internalstorage/config.go)

## 配置存储组件密码 Secret
Clusterpedia 的安装 yaml 会从 `internalstorage-password` 的 Secret 中获取密码。

将存储组件的密码配置到 Secret 中
```bash
kubectl -n clusterpedia-system create secret generic \
    internalstorage-password --from-literal=password=<存储组件访问密码>
```
