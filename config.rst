配置
=============

下面是 Flask-SQLAlchemy 中存在的配置值。 Flask-SQLAlchemy 从你的 Flask
主配置（可以以多种方式填充）中加载这些值。注意其中的一些在引擎创建后不
能修改，所以确保尽早配置且不在运行时修改它们。

配置键
------------------

一个该扩展当前理解的配制键清单:

.. tabularcolumns:: |p{6.5cm}|p{8.5cm}|

=============================== =========================================
``SQLALCHEMY_DATABASE_URI``     用于连接的数据库 URI 。例如:

                                - ``sqlite:////tmp/test.db``
                                - ``mysql://username:password@server/db``
``SQLALCHEMY_BINDS``            一个映射 binds 到连接 URI 的字典。更多
                                binds 的信息见 :ref:`binds` 。
``SQLALCHEMY_ECHO``             如果设置为 `Ture` ， SQLAlchemy 会记录所有
                                发给 stderr 的语句，这对调试有用。
``SQLALCHEMY_RECORD_QUERIES``   可以用于显式地禁用或启用查询记录。查询记录
                                在调试或测试模式自动启用。更多信息见
                                :func:`get_debug_queries` 。
``SQLALCHEMY_NATIVE_UNICODE``   可以用于显式禁用原生 unicode 支持。当使用
                                不合适的指定无编码的数据库默认值时，这对于
                                一些数据库适配器是必须的（比如 Ubuntu 上
                                某些版本的 PostgreSQL ）。
``SQLALCHEMY_POOL_SIZE``        数据库连接池的大小。默认是引擎默认值（通常
                                是 5 ）
``SQLALCHEMY_POOL_TIMEOUT``     设定连接池的连接超时时间。默认是 10 。
``SQLALCHEMY_POOL_RECYCLE``     多少秒后自动回收连接。这对 MySQL 是必要的，
                                它默认移除闲置多于 8 小时的连接。注意如果
                                使用了 MySQL ， Flask-SQLALchemy 自动设定
                                这个值为 2 小时。
=============================== =========================================

.. versionadded:: 0.8
   添加了 `SQLALCHEMY_NATIVE_UNICODE``, ``SQLALCHEMY_POOL_SIZE``,
   ``SQLALCHEMY_POOL_TIMEOUT`` 和 ``SQLALCHEMY_POOL_RECYCLE``
   配置键。


.. versionadded:: 0.12
   添加了 ``SQLALCHEMY_BINDS`` 配置键。

连接 URI 格式
---------------------

完整连接 URI 列表请跳转到 SQLAlchemy 下面的文档 (`Supported Databases
<http://www.sqlalchemy.org/docs/core/engines.html>`_) 。这里给出一些
常见的连接字符串。

SQLAlchemy 把一个引擎的源表示为一个连同设定引擎选项的可选字符串参数的
URI 。 URI 的形式是::

    dialect+driver://username:password@host:port/database

该字符串中的许多部分是可选的。如果没有指定驱动器，会选择默认的 （确保不
在这种情况下包含 ``+`` ）。

Postgres::

    postgresql://scott:tiger@localhost/mydatabase

MySQL::

    mysql://scott:tiger@localhost/mydatabase

Oracle::

    oracle://scott:tiger@127.0.0.1:1521/sidname

SQLite （注意开头的四个斜线）::

    sqlite:////absolute/path/to/foo.db
