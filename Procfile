web: gunicorn --chdir best_apr_backend backend.wsgi:application
worker: celery --app best_apr_backend worker -l INFO