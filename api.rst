API
---

.. module:: flask.ext.sqlalchemy

这部分文档叙述了所有 Flask-SQLAlchemy 中的公共类和函数。

配置
`````````````

.. autoclass:: SQLAlchemy
   :members:

   .. attribute:: Query

      :class:`BaseQuery` 类。

模型
``````

.. autoclass:: Model
   :members:

   .. attribute:: __bind_key__

      可选地声明要使用的 bind 。 `None` 为默认的 bind 。更多信息见
      :ref:`binds 。

.. autoclass:: BaseQuery
   :members: get, get_or_404, paginate, first_or_404

   .. method:: all()

      把此查询的结果返回为一个列表。这会执行底层查询。

   .. method:: order_by(*criterion)

      对查询应用一个或更多 ORDER BY 约束，并返回新的结果查询。

   .. method:: limit(limit)

      对应用一个 LIMIT ，并返回新的结果查询。

   .. method:: offset(offset)

      对查询应用一个 OFFSET ，并返回新的结果查询。

   .. method:: first()

      返回此查询中的第一个结果，如果结果不包含任何行，返回 `None` 。
      这会执行底层查询。

实用工具
`````````

.. autoclass:: Pagination
   :members:

.. autofunction:: get_debug_queries
