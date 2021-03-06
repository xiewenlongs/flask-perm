.. Flask-Perm documentation master file, created by
   sphinx-quickstart on Thu Jan 28 17:29:15 2016.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

Flask-Perm
==========

.. module:: flask.ext.perm

`Flask-Perm` is a Flask extension that can protect your view or function to be
accessed by person who owns proper permission. There is a default dashboard
to add/create/authorize/revoke permission to a person or a group, which is
convenient for you to bootstrap your permission management..

Installation
------------

Install the extension with one of the following commands::

    $ easy_install Flask-Perm

or alternatively if you have pip installed::

    $ pip install Flask-Perm

Usage
-----

Initialize
``````````

To use the extension simply import the class wrapper and pass the Flask app
object back to here. Do so like this::

    from flask import Flask
    from flask.ext.perm import Perm

    app = Flask(__name__)
    perm = Perm(app)

Or initialize perm in factory function::

    perm = Perm(app)

    def create_app():
        app = Flask(__name__)
        perm.init_app(app)
        return app

Register
````````

To have Flask-Perm works, you have some works to finish yet.

You must register current user loader before using `require_permission` 
and `require_group`. These two methods can not works well if they don't
know whom to require permission or group::

    @perm.current_user_loader
    def load_current_user():
        if 'user_id' in session:
            return User.query.get(session['user_id'])

If you have used `Flask-Login`, register can be like this::

    from flask_login import current_user
    @perm.current_user_loader(lambda: current_user)

You must register user loader, users loader, users count loader before 
using admin dashboard::

    @perm.user_loader
    def load_user(user_id):
        return User.query.get(user_id)

    @perm.users_count_loader
    def load_users_count():
        return User.query.all()

    @perm.users_loader
    def load_users(filter_by={}, sort_field='created_at', sort_dir='desc', offset=0, limit=20):
        sort = getattr(getattr(User, sort_field), sort_dir)()
        return User.query.filter_by(**filter_by).order_by(sort).offset(offset).limit(limit).all()


Configurations
````````````````

* `PERM_ADMIN_URL`, default `/perm-admin`. This url_prefix determins which
  url your dashboard will be visited.

Quick Start
-----------

Require Permission
``````````````````

code::

    @perm.require_permission('post.publish')
    def publish_post():
        Post.publish()

code::

    @perm.require_permission('post.publish', 'post.schedule')
    def schedule_post_publish():
        Post.schedule_publish()

In template, you can protect a block by writing code::

    {% if require_permission('post.publish') %}
    <a class="btn btn-primary" href="{{ url_for('publish_post') }}">
      Publish Post
    </a>
    {% endif %}

Require Group
`````````````

code::

    @perm.require_group('editor')
    def publish_post():
        Post.publish()

In template, you can protect a block by writing code::

    {% if require_group('editor') %}
    <a class="btn btn-primary" href="{{ url_for('publish_post') }}">
      Publish Post
    </a>
    {% endif %}

Low Level API
`````````````

Validate user's permission::

    perm.has_permission(user_id, 'post.publish')

Validate user's membership::

    perm.is_user_in_groups(user_id, 'editor')

Script
``````

Initialize Script Manager::

    from flask.ext.manager import Manager

    manager = Manager(app)
    perm.register_commands(manager)

Create super admin account::

    $ python manage.py perm create_superadmin admin@example.org
    Please input password:
    Please input password again:
    Success!

Reseting password shares same command with creating super admin.

List all super admins::

    $ python manage.py perm list_superadmin
    admin@example.org
    editor@example.org

Delete a super admin account::

    $ python example.py perm delete_superadmin admin@example.org
    Do you really want to delete this account? [y/n] [n]: y
    Success!

Dashboard
``````````

Before using dashboard, please create superadmin, which is described above.
Visit http://SERVER_NAME:PORT/PERM_ADMIN_URL, login and manage permissions.
There are several pages:

* Dashboard Index
* Users
* User Groups
* Permissions
* User Permissions
* User Group Permissions
* User Group Members

Other Library
--------------

Django Auth Contrib
````````````````````

Flask-Perm is a subset of django.contrib.auth.
Flask-Perm have no assumption of your user module, while django.contrib.auth
have builtin user support.
Flask-Perm is model-agnostic, while django.contrib.auth relate permission with
a specific model.


Flask-Principle
```````````````

The permission of Flask-Perm is similar to the ActionNeed of Flask-Principle;
and the group of Flask-Perm is similar to the RoleNeed of Flask-Principle.
All permisions and groups are created or deleted by superadmin's account;
while Need in Flask-Principle is hard coded.

Flask-Perm is not designed to system involved a large number of users;
Flask-Principle may be a good choice in this case.

Flask-Permissions
``````````````````

The basic concept in Flask-Permissions is `role`, `ability`.
In Flask-Permissions, user has several roles, and each role has several abibities.
The use case of Flask-Perm is a little bit more complex than Flask-Permissions.
In Flask-Perm, user gain permissions via both group and directly authorize permission.

Notice
------

Is it safe to store superadmin's password?
``````````````````````````````````````````

Superadmins have great power to control access permission.
Their password has encrypted in Bcrypt algorithm, which is considered very hard to be
cracked.
Use HTTPS to protect your site.
Don't use simple password for superadmin.
Keep changing password for superadmin.

How did Flask-Perm implement it?
````````````````````````````````

Flask-Perm manipulates data in several database tables with helps of Flask-SQLAlchemy.

Thanks to Ng-Admin, an administration dashboard application built in Angular.
Flask-Perm is allowed to supply RESTful APIs and has complete GUI to be used.

API
---
.. autoclass:: flask_perm.Perm
   :members:

Indices and tables
==================

* :ref:`genindex`
* :ref:`modindex`
* :ref:`search`

