# Coreseek配置示例

### 1. 配置用的数据库表
```
创建搜索统计表
CREATE TABLE `sph_search_counter` (
  `counter_id` int(11) NOT NULL,
  `max_update_time` int(11) NOT NULL DEFAULT '0',
  PRIMARY KEY (`counter_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

创建实体表略
```
### 2. 配置文件示例

```
# 全量数据源配置
source src_search_9312
{
    type		    = mysql
    sql_host		= 127.0.0.1
    sql_user		= root
    sql_pass		= 12345
    sql_db		    = db_search
    sql_port		= 3306	

    sql_query_pre   = SET NAMES utf8
    sql_query_pre   = REPLACE INTO sph_search_counter SELECT 9312, MAX(update_time) FROM fm_search
    sql_query       = SELECT id,name,alias,channel,director,actor,category,tag,score,disable FROM fm_search \
    				WHERE update_time<=(SELECT max_update_time FROM sph_search_counter WHERE counter_id=9312)
	#声明过滤属性
    sql_attr_uint   = disable
    sql_attr_uint   = score
    sql_attr_uint   = channel
}

# 主索引
index search_9312
{
    source		    = src_search_9312
    path		    = /usr/local/coreseek/var/data/search_9312
    docinfo		    = extern
    charset_dictpath   = /usr/local/mmseg3/etc/
    min_prefix_len  = 1
    charset_type    = zh_cn.utf-8
}

# 增量数据源
source src_search_delta_9312:src_search_9312
{
    sql_query_pre   = SET NAMES utf8
    sql_query       = SELECT id,name,alias,channel,director,actor,category,tag,score,disable FROM fm_search \
                    WHERE update_time>(SELECT max_update_time FROM sph_search_counter WHERE counter_id=9312)
}

# 增量索引
index search_delta_9312:search_9312
{
    source          = src_search_delta_9312
    path            = /usr/local/coreseek/var/data/search_delta_9312
    docinfo         = extern
    charset_dictpath = /usr/local/mmseg3/etc/
    min_prefix_len  = 1
    charset_type    = zh_cn.utf-8
}

# 建立索引允许使用的最大内存
indexer
{
	mem_limit		= 512M
}

# 配置搜索服务
searchd
{
	port			= 9312
	log				= /usr/local/coreseek/var/log/search_searchd_9312.log
	query_log		= /usr/local/coreseek/var/log/search_query_9312.log
	read_timeout	= 5
	max_children	= 30
	pid_file		= /usr/local/coreseek/var/log/search_searchd_9312.pid
	max_matches		= 2000
	seamless_rotate	= 1
	preopen_indexes	= 0
	unlink_old		= 1
}

```
以上配置仅供参考
