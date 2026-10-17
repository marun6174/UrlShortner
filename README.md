# URL Shortener

A URL shortener built with Python and Django. It provides a simple way to turn a long URL into a short link that can be shared and redirected to the original destination.

## Features

- Create a short link from a destination URL
- Redirect visitors from a short link to its original URL

## Tech stack

- Python
- Django

## Getting started

### Prerequisites

- Python 3.10 or later
- pip

### Set up the project

1. Clone this repository and move into the project directory.
2. Create and activate a virtual environment:

   ```bash
   python -m venv .venv
   source .venv/bin/activate
   ```

   On Windows, activate it with:

   ```powershell
   .venv\Scripts\Activate.ps1
   ```

3. Install Django:

   ```bash
   python -m pip install Django
   ```

4. Apply database migrations and start the development server:

   ```bash
   python manage.py migrate
   python manage.py runserver
   ```

5. Open <http://127.0.0.1:8000/> in your browser.

## Configuration

For local development, use Django's development settings. Before deploying, set a secure `SECRET_KEY`, configure `DEBUG = False`, define the allowed hosts, and use a production-ready database and web server.

## Development

Run the Django test suite with:

```bash
python manage.py test
```

## License

No license has been specified yet.
