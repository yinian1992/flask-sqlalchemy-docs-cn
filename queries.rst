.. currentmodule:: flask.ext.sqlalchemy

Select, Insert, Delete
======================

现在你已经有了 :ref:`声明好的模型 <models>` ，是时候从数据库查询数据了。
我们会使用 :ref:`quickstart` 章中的模型定义。


插入记录
-----------------

在我们查询之前，我们需要先插入。你的所有模型都应该有一个构造函数，如果你
忘记了，请确保加上一个。只有你自己使用这些构造函数而 SQLAlchemy 在内部不会使用它，
所以如何定义这些构造函数完全取决与你。

向数据库插入数据分为三步:

1.  创建 Python 对象
2.  把它添加到会话
3.  提交会话

这里的会话不是 Flask 会话，而是 Flask-SQLAlchemy 会话。这本质上是一个
数据库事务的加强版本。它是这样工作的:


>>> from yourapp import User
>>> me = User('admin', 'admin@example.com')
>>> db.session.add(me)
>>> db.session.commit()

好吧，这并不难。在哪点发生了什么？在你把对象添加到会话之前， SQLAlchemy
基本不考虑把它加到事务中。这是正确的，因为你仍然可以放弃更改。比如想象
在一个页面上创建文章，但是你只想把文章传递给模板来预览渲染，而不是把它
存进数据库。

:func:`~sqlalchemy.orm.session.Session.add` 会添加对象。它会发出一个
`INSERT` 语句给数据库，但是由于事务仍然没有提交，你不会立即得到返回的
ID 。如果你提交，你的用户会有一个 ID:

>>> me.id
1

删除记录
----------------

删除记录非常类似，用 :func:`~sqlalchemy.orm.session.Session.delete`
替换 :func:`~sqlalchemy.orm.session.Session.add`:

>>> db.session.delete(me)
>>> db.session.commit()

查询记录
----------------

那么我们如何从数据库中取回数据？为此， Flask-SQLAlchemy 在你的
:class:`Model` 类上提供了一个 :attr:`~Model.query` 属性。当你访问它
时，你会得到一个新的针对所有记录的查询对象。在你使用:func:`~sqlalchemy.orm.query.Query.all` 
或:func:`~sqlalchemy.orm.query.Query.first` 发起 select 之前
你可以使用诸如:func:`~sqlalchemy.orm.query.Query.filter` 的方法来过滤记录。
如果你想要用主键查询，你也可以使用:func:`~sqlalchemy.orm.query.Query.get` 。

下面的查询假设数据库中有如下条目:

=========== =========== =====================
`id`        `username`  `email`
1           admin       admin@example.com
2           peter       peter@example.org
3           guest       guest@example.com
=========== =========== =====================

通过用户名反查用户:

>>> peter = User.query.filter_by(username='peter').first()
>>> peter.id
1
>>> peter.email
u'peter@example.org'

与上面的相同，对不存在的用户名返回 `None`:

>>> missing = User.query.filter_by(username='missing').first()
>>> missing is None
True

以更复杂的表达式选取一些用户:

>>> User.query.filter(User.email.endswith('@example.com')).all()
[<User u'admin'>, <User u'guest'>]

以某种规则对用户排序:

>>> User.query.order_by(User.username)
[<User u'admin'>, <User u'guest'>, <User u'peter'>]

限制返回的用户数目:

>>> User.query.limit(1).all()
[<User u'admin'>]

用主键获取用户:

>>> User.query.get(1)
<User u'admin'>


在视图中查询
----------------

当你编写一个 Flask 视图函数，对不存在的条目返回一个 404 错误通常是非常
方便的。因为这个需求过于普遍， Flask-SQLAlchemy 为这个特定需求提供了一个
辅助函数。用  :meth:`~Query.get_or_404` 替代
:meth:`~sqlalchemy.orm.query.Query.get` ,而 :meth:`~Query.first_or_404`
代替 :meth:`~sqlalchemy.orm.query.Query.first` 。
这样会抛出一个异常，而不是返回 `None`::

    @app.route('/user/<username>')
    def show_user(username):
        user = User.query.filter_by(username=username).first_or_404()
        return render_template('show_user.html', user=user)
