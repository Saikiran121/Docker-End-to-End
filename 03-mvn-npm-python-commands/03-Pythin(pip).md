# 01. Development Environment Commands
      
      a. python -m venv venv && source venv/bin/activate	
         - Create and activate isolated virtual environment; like setting up a clean workshop.
      
      b. pip install -r requirements.txt
         - Install all project dependencies from requirements file.
      
      c. pip install -e .	
         - Install project in editable/development mode for local testing.
      
      d. pytest -v	
         - Run tests with verbose output during development.
      
      e. python -m flask run --debug or python manage.py runserver	
         - Start development server with hot-reload (Flask/Django).
      
      f. pip list --outdated	
         - Check for outdated packages that need updates.
      
      g. pip-audit	
         - Scan installed packages for known vulnerabilities.
      
      h. black . && flake8 .	
         - Format code and run linting checks.


# 02. CI & Testing Environment Commands
      
      a. python -m venv venv && source venv/bin/activate	
         - Create fresh virtual environment for CI isolation.
      
      b. pip install --no-cache-dir -r requirements.txt	
         - Install dependencies without cache for reproducible CI builds.
      
      c. pytest --cov=. --cov-report=xml --junitxml=test-results.xml	
         - Run tests with coverage reporting for CI dashboards.

      d. pytest -x --tb=short	
         - Stop on first failure with short traceback for faster CI feedback.
      
      e. pip-audit --format=json --output=audit.json	
         - Generate security audit report in JSON for CI processing.
      
      f. safety check -r requirements.txt --json	
         - Alternative security scanning tool for dependencies.
      
      g. black --check . && flake8 . && mypy .	
         - Enforce code quality without modifying files.

      h. bandit -r . -f json -o bandit-report.json	
         - Security linting for common Python security issues.


# 03. Staging Environment Commands
      
      a. pip install --no-deps -r requirements.txt	
         - Install exact dependency versions without resolving transitive deps.
      
      b. python -m pytest --tb=short -q	
         - Quick test validation in staging environment.
      
      c. pip freeze > requirements-staging.txt	
         - Capture exact staging environment for reproducibility.
      
      d. gunicorn app:app --bind 0.0.0.0:8000 --preload	
         - Start production-grade WSGI server for staging validation.
      
      e. python -c "import sys; print(sys.version)"	
         - Verify Python version consistency across environments.

      f. pip check	
         - Verify dependency compatibility in staging.


# 04. Production Environment Commands
      
      a. pip install --no-cache-dir --no-deps -r requirements.txt	
         - Install exact production dependencies without cache or resolution.
      
      b. python -O -m compileall .	
         - Compile Python bytecode with optimizations for faster startup.
      
      c. gunicorn app:app --workers 4 --bind 0.0.0.0:8000	
         - Launch production WSGI server with multiple workers.
      
      d. python -m py_compile *.py	
         - Pre-compile Python files to catch syntax errors before deployment.
      
      e. pip audit --fix	
         - Apply security fixes to production dependencies (with caution).

      f. python manage.py collectstatic --noinput (Django)	
         - Collect static files for production serving.
      
      g. python manage.py migrate --run-syncdb (Django)	
         - Apply database migrations in production.
