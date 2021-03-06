Problem
=======

Develop an app for testing input datasets of the following format::

 [
   {
     "a": 1,
     "b": 2,
   },
   {
     "a": 3,
     "b": 5,
   },
   ...
   {
     "a": prime number,
     "b": prime number,
   },
 ]

The app consumes datasets, then calculates results by adding up ``a`` and ``b``::

 [
   {
     "result": 3,
   },
   {
     "result": 8,
   },
   ...
   {
     "result": a's prime + b's prime,
   },
 ]

Features
--------

- take in input JSON via user interface
- save input to the DB
- process all uploaded datasets on a button click
- save results to the DB
- save all exceptions raised to the DB
- show the results of the last processing
- show the status of the last processing: True if exceptions were raised, False otheriwse
- show the list of exceptions raised

Implementation
--------------

1. Each datasets submissions invokes a chain of 3 tasks:

  - DB querying, JSON request
  - Processing JSON data, JSON response
  - Saving JSON to the DB

2. Each chain's task handled by a separate worker
3. App must be developed in a git-repository.


Requirements
------------

- App should be written in Python using Django framework
- Dependencies are installed using ``requirements.txt`` and ``pip``
- DB queries handled by ``Django ORM`` with ``PostgreSQL 9.4`` as a backend
- Asynchronous tasks handled by ``Celery`` with ``RabbitMQ`` as a backend
- Tasks monitoring done with ``Celery Flower``
- The App must be deployed on ``Ubuntu 14.04 LTS`` VPS instance (webservers, application server, daemons, etc. are up to developer)


Usage
=====

Submit -> Process -> See Report


Screencast
==========

.. image:: https://pilosus.org/pics/django-primes-interview.gif


Up & Running Locally
====================

Under Django project's directory (where ``manage.py`` is located) do the following:

1. Create a file with environment variables, e.g. ``.env``, for project's settings::


    # Django
    DJANGO_SECRET_KEY=YourSuperSecretString

    # DB
    DATABASE_NAME=dbname
    DATABASE_USER=dbuser
    DATABASE_PASSWORD=dbpasswd
    DATABASE_HOST=127.0.0.1
    DATABASE_PORT=5432

    # Celery
    CELERY_BROKER_URL=amqp://guest@localhost//
    CELERY_RESULT_BACKEND=rpc
    CELERY_QUEUE_FIRST=first
    CELERY_QUEUE_SECOND=second
    CELERY_QUEUE_THIRD=third

    # Flower
    FLOWER_BASIC_AUTH=foo:bar
    FLOWER_PORT=5555

2. Install ``requirements`` (globally, or better off in the ``virtualenv``)::

    # choose either of the following depending on your environment

    # production
    $ pip install -r ../requirements/prod.txt

    # development & testing
    $ pip install -r ../requirements/testing.txt

3. Before executing each of the following steps (in separate shells), export above mentioned variables
   and activate ``virtualenv`` if needed::

    $ set -a
    $ source .env
    $ set +a

    $ source .venv/bin/activate


4. Run three ``Celery`` workers::

    $ celery -A primes worker -Q first -l info --hostname=first-server@%h
    $ celery -A primes worker -Q second -l info --hostname=second-server@%h
    $ celery -A primes worker -Q third -l info --hostname=third-server@%h


5. Run ``Flower``::

    $ flower -A primes --port=5555


6. Run ``Django`` server::

    $ python manage.py runserver

7. Go to `http://127.0.0.1:8000/ <http://127.0.0.1:8000//>`_. Now you are done!


Testing
=======

The app covered with functional tests (using ``Selenium``), as well as with unit-tests (100% coverage). Run tests this way::

    # Unit-tests without coverage
    $ python manage.py test datasets

    # Unit-tests with coverage
    # NB! Install dependencies with pip install -r requirements/testing.txt beforehand!
    $ ./run-tests-with-coverage.sh datasets

    # Functional test
    # NB! Tests get use of TransactionTestCase, so the tests will use your DB by default
    # Also, see discussion in functional_tests/tests.py as for why we don't use LiveServerTestCase
    $ python manage.py test functional_tests


License
========

See `LICENSE` file.
