=====================================================================
 Celery Result Backends using the Django ORM/Cache framework.
=====================================================================

|build-status| |coverage| |license| |wheel| |pyversion| |pyimp|

:Version: 1.1.1.celery3
:Web: http://django-celery-results.readthedocs.io/
:Download: http://pypi.python.org/pypi/django-celery-results
:Source: http://github.com/celery/django-celery-results
:Keywords: django, celery, database, results

About
=====

This extension enables you to store Celery task results using the Django ORM.

It defines a single model (``django_celery_results.models.TaskResult``)
used to store task results, and you can query this database table like
any other Django model.

About the fork
==============

This fork backported the main package (``django-celery-results v1.1.1``) to bring support
with Django 1.7+ and Celery 3.1.17+.
Note, unit tests have not been backported yet.

How to integrate this fork with Django-Celery application
---------------------------------------------------------

To integrate Django-Celery based application with this fork you need to have:
- Django v1.7+
- Celery v3.1.17+

In the package, update ``setup.py``:
  - extends the ``install_requires`` list with 'django-celery-results==1.1.1.celery3' item,
  - extends the ``dependency_links`` list with
    'git+https://github.com/essence-tech/django-celery-results.git@master#egg=django-celery-results-1.1.1.celery3'
    item.
  - extend ``INSTALLED_APPS`` list of Django's `settings.py` with ``'django_celery_results'`` item,::
    >>> INSTALLED_APPS = (
    ...     ...,
    ...     'django_celery_results',
    ... )
  - run migrations,::
    $ python manage.py migrate celery_results

Setup Django application to have an alternative persistent SQL result backend
-----------------------------------------------------------------------------
Here is the case - we have a Django application and for most of the cases we don't need a persistent result backend,
but for some feature having persistent result is critical.

Celery does not support multiple result backends in a single application.
But you still can set a different result backend on a Celery task level. To achieve this you need to redefine the
``backend`` task property as shown below. In such case, you also need to pass the same result backend to ``AsynResult``
constructor to get a proper result.

Here is an example of the approach:

1. Base django configuration, `settings.py`,::
    >>> CELERY_RESULT_BACKEND = 'redis://localhost:6379/0'
    >>> INSTALLED_APPS = (
    ...     ...,
    ...     'django_celery_results',
    ... )

2. Create an alternative persistent backend object, `celery.py`,::
    >>> from celery import Celery
    >>> from django.conf import settings
    >>> from django_celery_results.backends.database import DatabaseBackend
    >>> app = Celery('project')
    >>> app.config_from_object('django.conf:settings')
    >>> app.autodiscover_tasks(lambda: settings.INSTALLED_APPS, force=True)
    >>> persistent_backend = DatabaseBackend(app=app)

3. Bind the alternative persistent result backend in a celery task,::
    >>> from celery.task import Task
    >>> class TaskWithPersistentResults(Task):
    ...     @property
    ...     def backend(cls):
    ...         from celery import persistent_backend
    ...         return persistent_backend

     NOTE: here persistent backend is hidden in the method to resolve cyclic import issue, since from the 2nd (above)
     excerpt you can see that there is ``app.autodiscover_tasks(..)`` call which causes the issue.

4. Retrieve results from the celery task with the alternative persistent result backend,::
    >>> from celery.result import AsyncResult
    >>> from celery import app, persistent_backend
    >>> res = AsyncResult('86e225f4-c662-4795-933d-69d8fac1b056', app=app, backend=persistent_backend)

Build package to upload to the PyPi repository
----------------------------------------------

1. Checkout the package from the github repository::
   $ git clone https://github.com/essence-tech/django-celery-results.git
   $ cd django-celery-results

2. Build source distribution::
   $ python setup.py sdist

3. Install twine (helper package to upload package to PyPi repo)::
   $ pip install twine

4. Upload package to the PyPi::
   $ twine upload -r nexus-production dist/*

NOTE: `nexus-production` configured separately. Please look for detailed instructions related to upload to Nexus
on the wiki page: https://essencedigital.atlassian.net/wiki/spaces/IO/pages/139577933/Pypi+repository

Installing
==========

The installation instructions for this extension is available
from the `Celery documentation`_:

http://docs.celeryproject.org/en/latest/django/first-steps-with-django.html#django-celery-results-using-the-django-orm-cache-as-a-result-backend


.. _`Celery documentation`:
    http://docs.celeryproject.org/en/latest/django/first-steps-with-django.html#django-celery-results-using-the-django-orm-cache-as-a-result-backend

.. _installation:

Installation
============

You can install django-celery-results either via the Python Package Index (PyPI)
or from source.

To install using `pip`,::

    $ pip install -U django-celery-results

.. _installing-from-source:

Downloading and installing from source
--------------------------------------

Download the latest version of django-celery-results from
http://pypi.python.org/pypi/django-celery-results

You can install it by doing the following,::

    $ tar xvfz django-celery-results-0.0.0.tar.gz
    $ cd django-celery-results-0.0.0
    $ python setup.py build
    # python setup.py install

The last command must be executed as a privileged user if
you are not currently using a virtualenv.

.. _installing-from-git:

Using the development version
-----------------------------

With pip
~~~~~~~~

You can install the latest snapshot of django-celery-results using the following
pip command::

    $ pip install https://github.com/celery/django-celery-results/zipball/master#egg=django-celery-results


Issues with mysql
-----------------

If you want to run ``django-celery-results`` with MySQL, you might run into some issues.

One such issue is when you try to run ``python manage.py migrate django_celery_results``, you might get the following error::

    django.db.utils.OperationalError: (1071, 'Specified key was too long; max key length is 767 bytes')

To get around this issue, you can set::

    DJANGO_CELERY_RESULTS_TASK_ID_MAX_LENGTH=191

(or any other value if any other db other than MySQL is causing similar issues.)

max_length of **191** seems to work for MySQL.


.. |build-status| image:: https://secure.travis-ci.org/celery/django-celery-results.svg?branch=master
    :alt: Build status
    :target: https://travis-ci.org/celery/django-celery-results

.. |coverage| image:: https://codecov.io/github/celery/django-celery-results/coverage.svg?branch=master
    :target: https://codecov.io/github/celery/django-celery-results?branch=master

.. |license| image:: https://img.shields.io/pypi/l/django-celery-results.svg
    :alt: BSD License
    :target: https://opensource.org/licenses/BSD-3-Clause

.. |wheel| image:: https://img.shields.io/pypi/wheel/django-celery-results.svg
    :alt: django-celery-results can be installed via wheel
    :target: http://pypi.python.org/pypi/django-celery-results/

.. |pyversion| image:: https://img.shields.io/pypi/pyversions/django-celery-results.svg
    :alt: Supported Python versions.
    :target: http://pypi.python.org/pypi/django-celery-results/

.. |pyimp| image:: https://img.shields.io/pypi/implementation/django-celery-results.svg
    :alt: Support Python implementations.
    :target: http://pypi.python.org/pypi/django-celery-results/

.. test::
