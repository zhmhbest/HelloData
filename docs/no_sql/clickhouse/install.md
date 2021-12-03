
## 安装

### 系统

#### 依赖检查

```bash
# CPU必须支持SSE4.2指令集
grep -q sse4_2 /proc/cpuinfo && echo "Supported"

# 依赖glibc-2.16
yum info glibc | grep '^Version' | uniq
# yum -y install glibc glibc-headers glibc-common glibc-devel
```

#### 关闭防火墙

```bash
systemctl disable firewalld.service
systemctl stop firewalld.service
firewall-cmd --state
```

#### [配置Hosts](../zookeeper/index.html#系统配置)

#### [安装Zookeeper](../zookeeper/index.html)

### 安装ClickHouse

```bash
mkdir clickhouse; pushd clickhouse

# CLICKHOUSE_REPO='https://repo.yandex.ru/clickhouse/rpm/stable/x86_64'
CLICKHOUSE_REPO='http://192.168.19.90/files/clickhouse/rpm'
CLICKHOUSE_VERSION='20.8.3.18-2'
# echo "${CLICKHOUSE_REPO}/clickhouse-common-static-${CLICKHOUSE_VERSION}.x86_64.rpm"
# echo "${CLICKHOUSE_REPO}/clickhouse-server-${CLICKHOUSE_VERSION}.noarch.rpm"
# echo "${CLICKHOUSE_REPO}/clickhouse-client-${CLICKHOUSE_VERSION}.noarch.rpm"
wget "${CLICKHOUSE_REPO}/clickhouse-common-static-${CLICKHOUSE_VERSION}.x86_64.rpm"
wget "${CLICKHOUSE_REPO}/clickhouse-server-${CLICKHOUSE_VERSION}.noarch.rpm"
wget "${CLICKHOUSE_REPO}/clickhouse-client-${CLICKHOUSE_VERSION}.noarch.rpm"

yum -y localinstall ./*.rpm
popd

systemctl stop clickhouse-server
systemctl status clickhouse-server
# systemctl start clickhouse-server
```

### 配置ClickHouse

#### [config.xml](https://clickhouse.tech/docs/zh/operations/configuration-files/)

```bash
vim /etc/clickhouse-server/config.xml
# :?<listen_host>
# <Enter>
# ...
# :?<interserver_http_host>
# <Enter>
# ...
# wq
grep '<listen_host>' /etc/clickhouse-server/config.xml
grep '<interserver_http_host>' /etc/clickhouse-server/config.xml
```

```xml
    <!-- 全域暴露 -->
    <listen_host>::</listen_host>
    <listen_host>0.0.0.0</listen_host>
```

```xml
    <!-- 主机名称（需要定制） -->
    <!-- ↓↓↓`hostname -f`↓↓↓ -->
    <interserver_http_host>clickhouse1</interserver_http_host>
    <!-- ↑↑↑`hostname -f`↑↑↑ -->
    <timezone>Asia/Shanghai</timezone>
    <include_from>/etc/clickhouse-server/metrika.xml</include_from>
```

#### user.xml

```bash
# 生成SHA256密钥
echo -n 'ckpwd' | sha256sum

vim /etc/clickhouse-server/users.xml
# :?<password>
grep '^ *<password' /etc/clickhouse-server/users.xml
```

```xml
<!-- 仅选择以下一个设置 -->

<!-- 明文密钥 -->
<password>ckpwd</password>

<!-- 密文密钥 -->
<password_sha256_hex>0fcde7eb94e529578898ccb51c9cb941e51447e1d45cdc68deba25c38c578967</password_sha256_hex>
```

#### metrica.xml

```bash
rm -f /etc/clickhouse-server/metrika.xml
vim /etc/clickhouse-server/metrika.xml
```

```xml
<yandex>
    <clickhouse_remote_servers>
        <!-- 自定义集群名称 -->
        <ck_cluster>
            <!-- 分片1 -->
            <shard>
                <!-- 分片权重，多少概率写入此分片 -->
                <weight>1</weight>
                <!-- 不只将数据写入其中一个副本 -->
                <internal_replication>true</internal_replication>
                <!-- 副本1 -->
                <replica>
                    <host>clickhouse1</host>
                    <port>9000</port>
                </replica>
                <!-- 副本2 -->
                <replica>
                    <host>clickhouse2</host>
                    <port>9000</port>
                </replica>
            </shard>
            <!-- 分片2 -->
            <shard>
                <weight>1</weight>
                <internal_replication>true</internal_replication>
                <!-- 副本1 -->
                <replica>
                    <host>clickhouse3</host>
                    <port>9000</port>
                </replica>
            </shard>
        </ck_cluster>
    </clickhouse_remote_servers>

    <zookeeper-servers>
        <node index="1">
            <host>zookeeper1</host>
            <port>2181</port>
        </node>
        <node index="2">
            <host>zookeeper2</host>
            <port>2181</port>
        </node>
        <node index="3">
            <host>zookeeper3</host>
            <port>2181</port>
        </node>
    </zookeeper-servers>

    <!-- 当前主机所在集群-分片-副本（需要定制） -->
    <macros>
        <layer>ck_cluster</layer>
        <shard>01</shard>
        <replica>clickhouse1</replica>
    </macros>

    <networks>
        <ip>::/0</ip>
    </networks>

    <clickhouse_compression>
        <case>
            <min_part_size>10000000000</min_part_size>
            <min_part_size_ratio>0.01</min_part_size_ratio>
            <method>lz4</method>
            <!-- 压缩算法: zstd | lz4 -->
        </case>
    </clickhouse_compression>
</yandex>
```

### 调试ClickHouse

#### 服务管理

```bash
# Debug运行
sudo -u clickhouse clickhouse-server --config-file=/etc/clickhouse-server/config.xml
```

```bash
systemctl start clickhouse-server
systemctl restart clickhouse-server
systemctl status clickhouse-server
```

#### 测试集群

```bash
# clickhouse-client --multiline --user <用户名> --password <密码>
clickhouse-client --multiline --user <用户名> --password <密码>
clickhouse-client -m
```
