.. _binds:

.. currentmodule:: flask.ext.sqlalchemy

用 Binds 操作多个数据库
=============================

从 0.12 开始， Flask-SQLAlchemy 可以容易地连接到多个数据库。为了实现
这个功能，预配置了 SQLAlchemy 来支持多个“binds”。

什么是 binds ？在 SQLAlchemy 中，一个 bind 是可以执行 SQL 语句 且通常是一
个连接或引擎的东西。在 Flask-SQLAlchemy 中， bind 总是背后自动为你创建好
的引擎。这些引擎中的每个之后都会关联一个短键（bind key）。这个键会在模型
声明时使用来把一个模型关联到一个特定引擎。

如果没有为模型指定 bind key ，会默认连接（即 ``SQLALCHEMY_DATABASE_URI``
配置的值）。


示例配置
---------------------

下面的配置声明了三个数据库连接：特殊的默认值和另外两个分别名为 `users`
（用于用户）和 `appmeat` （连接到一个提供只读访问应用内部数据的 sqlite
数据库）::

    SQLALCHEMY_DATABASE_URI = 'postgres://localhost/main'
    SQLALCHEMY_BINDS = {
        'users':        'mysqldb://localhost/users',
        'appmeta':      'sqlite:////path/to/appmeta.db'
    }

创建和删除表
----------------------------

:meth:`~SQLAlchemy.create_all` 和 :meth:`~SQLAlchemy.drop_all` 方法默认作用
于声明的所有 bind ，包括默认的。这个行为可以通过提供 `bind` 参数来定制。它
可以是单个 bind 名 、``'__all__'`` 指向所有 binds 或一个 bind 名的列表。默
认的 bind （ ``SQLALCHEMY_DATABASE_URI`` ）名为 `None`:

>>> db.create_all()
>>> db.create_all(bind=['users'])
>>> db.create_all(bind='appmeta')
>>> db.drop_all(bind=None)

引用 Binds
------------------

当你声明模型时，你可以用 :attr:`~Model.__bind_key__` 属性指定 bind::

    class User(db.Model):
        __bind_key__ = 'users'
        id = db.Column(db.Integer, primary_key=True)
        username = db.Column(db.String(80), unique=True)

bind key 内部存储在表的 `info` 字典中，作为 ``'bind_key'`` 键值。了解这个
很重要，因为当你想要直接创建一个表对象时，你会需要把它放在那::

    user_favorites = db.Table('user_favorites',
        db.Column('user_id', db.Integer, db.ForeignKey('user.id')),
        db.Column('message_id', db.Integer, db.ForeignKey('message.id')),
        info={'bind_key': 'users'}
    )

如果你在模型上指定了 `__bind_key__` ，你可以用它们准确地做你想要的。模型会自行连
接到指定的数据库连接。
