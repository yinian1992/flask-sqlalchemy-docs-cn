.. _quickstart:

快速入门
==========

.. currentmodule:: flask.ext.sqlalchemy

Flask-SQLAlchemy 的使用是有趣的，对于基本应用异常的简单，并且为大型应用扩展也
是没有困难的。为了完整的指导，签出 API 文档中的 :class:`SQLAlchemy` 类部分。


一个最小的应用
---------------------

对于通常情况，只有一个 Flask 应用，你需要做的全部就是创建你的 Flask 应用，
选择加载你的配置，然后在创建 :class:`SQLALchemy` 时把应用传递给它。

一旦创建，这个对象会包含 :mod:`sqlalchemy` 和 :mod:`sqlalchemy.orm` 中
的所有函数和助手。此外，它还提供了一个名为 `Model` 的类，用于作为声明
模型时的 delarative 基类::

    from flask import Flask
    from flask.ext.sqlalchemy import SQLAlchemy

    app = Flask(__name__)
    app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:////tmp/test.db'
    db = SQLAlchemy(app)


    class User(db.Model):
        id = db.Column(db.Integer, primary_key=True)
        username = db.Column(db.String(80), unique=True)
        email = db.Column(db.String(120), unique=True)

        def __init__(self, username, email):
            self.username = username
            self.email = email

        def __repr__(self):
            return '<User %r>' % self.username

要创建初始的数据库，只需要在交互式 Python shell 中导入 `db` 对象并
调用 :meth:`SQLAlchemy.create_all` 方法来创建表和数据库:

>>> from yourapplication import db
>>> db.create_all()

砰，你的数据库就好了。现在来创建一些用户:

>>> from yourapplication import User
>>> admin = User('admin', 'admin@example.com')
>>> guest = User('guest', 'guest@example.com')

但是它们还不在数据库中，所以让我们确保它们被添加进去:

>>> db.session.add(admin)
>>> db.session.add(guest)
>>> db.session.commit()

访问数据库中的内容易如反掌:

>>> users = User.query.all()
[<User u'admin'>, <User u'guest'>]
>>> admin = User.query.filter_by(username='admin').first()
<User u'admin'>

简单关系
--------------------

SQLAlchemy 聚合关系型数据库和关系型数据库善于处理关系的方面。如此，我们
可以创建一个有两张互相关联的表的应用作为例子::

    from datetime import datetime


    class Post(db.Model):
        id = db.Column(db.Integer, primary_key=True)
        title = db.Column(db.String(80))
        body = db.Column(db.Text)
        pub_date = db.Column(db.DateTime)

        category_id = db.Column(db.Integer, db.ForeignKey('category.id'))
        category = db.relationship('Category',
            backref=db.backref('posts', lazy='dynamic'))

        def __init__(self, title, body, category, pub_date=None):
            self.title = title
            self.body = body
            if pub_date is None:
                pub_date = datetime.utcnow()
            self.pub_date = pub_date
            self.category = category

        def __repr__(self):
            return '<Post %r>' % self.title


    class Category(db.Model):
        id = db.Column(db.Integer, primary_key=True)
        name = db.Column(db.String(50))

        def __init__(self, name):
            self.name = name

        def __repr__(self):
            return '<Category %r>' % self.name

首先让我们创建一些对象:

>>> py = Category('Python')
>>> p = Post('Hello Python!', 'Python is pretty cool', py)
>>> db.session.add(py)
>>> db.session.add(p)

现在因为我们把 `posts` 声明为动态关系，并在反向引用中，它作为查询出现:

>>> py.posts
<sqlalchemy.orm.dynamic.AppenderBaseQuery object at 0x1027d37d0>

它的行为与通常的查询对象类似，所以我们可以用它获取与我们测试的“Python”分类
关联的所有帖子:

>>> py.posts.all()
[<Post 'Hello Python!'>]


启蒙之路
---------------------

你仅需要知道的与普通 SQLAlchemy 的区别是:

1.  :class:`SQLAlchemy` 允许你访问下面的东西:
    -   :mod:`sqlalchemy` 和 :mod:`sqlalchemy.orm` 模块中的所有函数
        和类
    -   一个名为 `session` 的预配置范围会话
    -   :attr:`~SQLAlchemy.metadata` 属性
    -   :attr:`~SQLAlchemy.engine` 属性
    -   :meth:`SQLAlchemy.create_all` 和 :meth:`SQLAlchemy.drop_all`
        方法，根据模型分别创建和删除表
    -   一个 :class:`Model` 基类，其实是一个配置好的 delarative 基类

2.  declarativ 基类 :class:`Model` 的表现得像一个正规的 Python 类，而
    附加了一个 `query` 属性用于查询模型
    （ :class:`Model` 和 :class:`BaseQuery` ）

3.  你必须提交会话，但你不需要在请求的最后移除它， Flask-SQLAlchemy 已
    经为你这么做了。
