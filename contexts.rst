.. _contexts:

.. currentmodule:: flask.ext.sqlalchemy

介绍上下文
==========================

如果你计划只使用一个应用，你大可跳过这章。只需要把你的应用传递给
:class:`SQLAlchemy` 的构造函数，通常情况下，你就搞定了。而如果你想使用
多于一个的应用或是在函数中动态创建应用，你会想继续阅读本章。

如果你在一个函数中定义定义你的应用，而 :class:`SQLAlchemy` 是全局的，
后者怎么得悉前者？答案是 :meth:`~SQLAlchemy.init_app` 函数::

    from flask import Flask
    from flask.ext.sqlalchemy import SQLAlchemy

    db = SQLAlchemy()

    def create_app():
        app = Flask(__name__)
        db.init_app(app)
        return app

它所做的是准备应用以与 :class:`SQLAlchemy` 共同工作。可是这
现在不把 :class:`SQLAlchemy` 绑定到你的应用。为什么不这么做？
因为那里可能创建不止一个应用。

那么 :class:`SQLAlchemy` 如何获知你的应用？你会需要配置一个请求上下文。
如果你在一个 Flask 视图函数中进行工作，这会自动实现。但如果你在交互式
的 shell 中，你需要手动这么做。

（见 `深入上下文局域变量
<http://flask.pocoo.org/docs/shell/#diving-into-context-locals>`_ ）。

简而言之，像这样做:

>>> from yourapp import create_app
>>> app = create_app()
>>> app.test_request_context().push()

在脚本中，使用 with 声明也同样有意义::

    def my_function():
        with app():
            user = db.User(...)
            db.session.add(user)
            db.session.commit()

Flask-SQLAlchemy 里的一些函数也可以接受要操作的应用作为参数:

>>> from yourapp import db, create_app
>>> db.create_all(app=create_app())
