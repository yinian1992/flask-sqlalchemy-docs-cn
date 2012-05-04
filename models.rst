.. _models:

.. currentmodule:: flask.ext.sqlalchemy

模型声明
================

通常， Flask-SQLALchemy 表现得像来自 :mod:`~sqlalchemy.ext.declarative`
扩展的一个妥善配置的 declarative 基类。如此，我们推荐阅读 SQLAlchemy 文档
来获取完整的参考。尽管如此，这里会给出最通用的用法。

需要记住的事情:

-   你所有模型的基类叫做 `db.Model` 。它存储在你必须创建的 SQLAlchemy 实
    例上。更多细节见 :ref:`quickstart` 。
-   一些部分在 SQLAlchemy 中是必须的，而在 Flask-SQLAlchemy 中是可选的。
    比如表明已经为你自动设置好，除非你重载它。它由小写的类名导出，即
    “CamelCase” 转换为 “camel_case”。

简单的例子
--------------

一个非常简单的例子::

    class User(db.Model):
        id = db.Column(db.Integer, primary_key=True)
        username = db.Column(db.String(80), unique=True)
        email = db.Column(db.String(120), unique=True)

        def __init__(self, username, email):
            self.username = username
            self.email = email

        def __repr__(self):
            return '<User %r>' % self.username

用 :class:`~sqlalchemy.Column` 来定义一个列。类名就是你把赋给的那个变量的
名字。如果你想要在表中使用不同的名字，你可以提供一个想要的列名的字符串作为
可选第一个参数。主键用 ``primary_key=Ture`` 标记。可以把多个键标记为主键，
此时它们作为复合主键。

列的类型是 :class:`~sqlalchemy.Column` 的第一个参数。你可以直接提供它们或
进一步规定（比如提供一个长度）。下面的类型是最常用的:


=================== =====================================
`Integer`           整数
`String` (size)     有最大长度的字符串
`Text`              长 unicode 文本
`DateTime`          表示为 :mod:`~datetime.datetime` 对象
                    的时间和日期
`Float`             存储浮点值
`Boolean`           存储布尔值
`PickleType`        存储一个持久化 Python 对象
`LargeBinary`       存储任意大的二进制数据
=================== =====================================

一对多关系
-------------------------

最常用的关系就是一对多关系。因为关系在它们建立之前就已经声明，你可以使用
字符串来参考还没有创建的类（比如如果 `Person` 定义了一个到 `Article` 的
关系，而这个关系在文件的后面才会声明）。

关系用 :func:`~sqlalchemy.orm.relationship` 函数来表示。而外键必须用
:class:`sqlalchemy.schema.ForeignKey` 来单独声明::

    class Person(db.Model):
        id = db.Column(db.Integer, primary_key=True)
        name = db.Column(db.String(50))
        addresses = db.relationship('Address', backref='person',
                                    lazy='dynamic')

    class Address(db.Model):
        id = db.Column(db.Integer, primary_key=True)
        email = db.Column(db.String(50))
        person_id = db.Column(db.Integer, db.ForeignKey('person.id'))

``db.relationship()`` 做了什么？这个函数返回一个可以做许多事情的属性。
在本案例中，我们让它指向 `Address` 类并加载那些中的多个。它如何知道这
会返回至少一个地址？因为 SQLALchemy 从你的声明中猜测了一个有用的默认值。
如果你想要一对一联系，你可以把 ``uselist=False`` 传给
:func:`~sqlalchemy.orm.relationship` 。

那么 `backref` 和 `lazy` 意味着什么？ `backref` 是一个同样在 `Address` 类
上声明新属性的简单方法。你之后也可以用 `my_address.person` 来获取这个地址
的人。 `lazy` 决定了 SQLAlchemy 什么时候从数据库中加载数据:

-   ``'select'`` （默认值）意味着 SQLAlchemy 会在使用一个标准 select 语句
    时一气呵成加载那些数据
-   ``'joined'`` 让 SQLAlchemy 当父级使用 `JOIN` 语句是，在相同的查询中加
    载关系。
-   ``'subquery'`` 类似 ``'joined'`` ，但是 SQLAlchemy 会使用子查询。
-   ``'dynamic'`` 在你有可能条目时是特别有用的。 SQLAlchemy 会返回另一个查
    询对象，你可以在加载这些条目时进一步提取。如果不仅想要关系下的少量条目
    时，这通常是你想要的。

你如何为反向引用（backrefs）定义惰性（lazy）状态？使用
:func:`~sqlalchemy.orm.backref` 函数::

    class User(db.Model):
        id = db.Column(db.Integer, primary_key=True)
        name = db.Column(db.String(50))
        addresses = db.relationship('Address',
            backref=db.backref('person', lazy='joined'), lazy='dynamic')

多对多关系
--------------------------

如果你想要用多对多关系，你需要定义一个用于关系的辅助表。对于这个辅助表，
强烈建议 *不* 使用模型，而是采用一个实际的表::

    tags = db.Table('tags',
        db.Column('tag_id', db.Integer, db.ForeignKey('tag.id')),
        db.Column('page_id', db.Integer, db.ForeignKey('page.id'))
    )

    class Page(db.Model):
        id = db.Column(db.Integer, primary_key=True)
        tags = db.relationship('Tag', secondary=tags,
            backref=db.backref('pages', lazy='dynamic'))

    class Tag(db.Model):
        id = db.Column(db.Integer, primary_key=True)

这里我们配置 `Page.tags` 加载后作为标签的列表，因为我们并不期望每页出现
太多的标签。而每个 tag 的页面列表（ `Tag.pages` ）是一个动态的反向引用。
正如上面提到的，这意味着你会得到一个可以发起 select 的查询对象。
