当然，以下是一个**完整的 Python 3.8 脚本文件**，用于将 PostgreSQL（lastfm 音乐播放记录）同步到 SQLite（sellite）数据库。该脚本包含了配置、批量同步、断点续传、异常处理和日志记录，结构清晰，易于维护。

你只需根据实际数据库连接信息和表名进行简单调整即可直接使用。

---

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-
"""
lastfm_pg_to_sellite_sync.py

将 PostgreSQL 中的 lastfm 音乐播放记录同步到 SQLite（sellite）数据库。
支持断点续传、批量同步、异常处理和日志记录。

作者：专业编程专家
"""

import os
import sys
import logging
import psycopg2
import sqlite3
from typing import List, Tuple

# =========================
# 配置区
# =========================

# PostgreSQL 配置
PG_CONFIG = {
    'host': 'localhost',
    'port': 5432,
    'dbname': 'your_pg_db',
    'user': 'your_pg_user',
    'password': 'your_pg_password'
}

# SQLite 文件路径
SQLITE_PATH = 'sellite.db'

# 批量同步大小
BATCH_SIZE = 1000

# 日志配置
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s %(levelname)s %(message)s',
    handlers=[
        logging.StreamHandler(sys.stdout),
        logging.FileHandler('sync.log', encoding='utf-8')
    ]
)
logger = logging.getLogger(__name__)

# =========================
# 工具函数
# =========================

def get_pg_connection():
    return psycopg2.connect(**PG_CONFIG)

def get_sqlite_connection():
    return sqlite3.connect(SQLITE_PATH)

def ensure_sqlite_table(conn: sqlite3.Connection):
    """
    确保目标表存在。根据实际 sellite 表结构调整。
    """
    with conn:
        conn.execute("""
        CREATE TABLE IF NOT EXISTS sellite_music_records (
            id INTEGER PRIMARY KEY,
            name TEXT NOT NULL,
            artist TEXT NOT NULL,
            album TEXT,
            image TEXT,
            mbid TEXT NOT NULL,
            url TEXT NOT NULL,
            date TEXT NOT NULL
        )
        """)
        # 用于记录上次同步的时间
        conn.execute("""
        CREATE TABLE IF NOT EXISTS sync_state (
            key TEXT PRIMARY KEY,
            value TEXT
        )
        """)

def get_last_sync_time(conn: sqlite3.Connection) -> str:
    """
    获取上次同步的最大 date，初次同步返回 '1970-01-01 00:00:00'
    """
    cur = conn.execute("SELECT value FROM sync_state WHERE key = 'last_sync_time'")
    row = cur.fetchone()
    return row[0] if row else '1970-01-01 00:00:00'

def update_last_sync_time(conn: sqlite3.Connection, new_time: str):
    with conn:
        conn.execute("""
            INSERT INTO sync_state(key, value)
            VALUES ('last_sync_time', ?)
            ON CONFLICT(key) DO UPDATE SET value=excluded.value
        """, (new_time,))

def fetch_pg_records(pg_conn, last_sync_time: str, batch_size: int) -> List[Tuple]:
    """
    从 PostgreSQL 批量读取数据
    """
    with pg_conn.cursor() as cur:
        cur.execute("""
            SELECT id, name, artist, album, image, mbid, url, date
            FROM music_scrobble_records
            WHERE date > %s
            ORDER BY date ASC
            LIMIT %s
        """, (last_sync_time, batch_size))
        return cur.fetchall()

def insert_to_sqlite(sqlite_conn: sqlite3.Connection, records: List[Tuple]):
    """
    批量插入到 SQLite，忽略已存在的记录
    """
    with sqlite_conn:
        sqlite_conn.executemany("""
            INSERT OR IGNORE INTO sellite_music_records
            (id, name, artist, album, image, mbid, url, date)
            VALUES (?, ?, ?, ?, ?, ?, ?, ?)
        """, records)

# =========================
# 主同步流程
# =========================

def sync():
    logger.info("同步任务启动")
    pg_conn = get_pg_connection()
    sqlite_conn = get_sqlite_connection()
    ensure_sqlite_table(sqlite_conn)

    try:
        last_sync_time = get_last_sync_time(sqlite_conn)
        logger.info(f"上次同步时间: {last_sync_time}")

        total_synced = 0
        while True:
            records = fetch_pg_records(pg_conn, last_sync_time, BATCH_SIZE)
            if not records:
                logger.info("无新增数据，任务完成")
                break

            insert_to_sqlite(sqlite_conn, records)
            total_synced += len(records)
            # 假设 date 字段在最后一列
            last_sync_time = max(r[-1] for r in records)
            update_last_sync_time(sqlite_conn, last_sync_time)
            logger.info(f"已同步 {total_synced} 条，最新同步时间: {last_sync_time}")

    except Exception as e:
        logger.exception(f"同步过程中发生异常: {e}")
    finally:
        pg_conn.close()
        sqlite_conn.close()
        logger.info("数据库连接已关闭")

# =========================
# 程序入口
# =========================

if __name__ == '__main__':
    sync()
```

---

### 使用说明

1. **安装依赖**  
   ```bash
   pip install psycopg2
   ```

2. **修改配置**  
   - 填写你的 PostgreSQL 连接信息（`PG_CONFIG`）。
   - 如有需要，调整 SQLite 文件路径和表名。

3. **运行脚本**  
   ```bash
   python lastfm_pg_to_sellite_sync.py
   ```

---

如需进一步定制（如字段映射、增量同步策略优化等），请补充你的 sellite 表结构或特殊需求。