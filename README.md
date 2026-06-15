# Django Blog

A full-featured blog application built with Django 6.0, featuring user authentication, profile management with avatars, rich post CRUD operations, pagination, and password reset via email.

**Live demo:** [https://blog-j22t.onrender.com/](https://blog-j22t.onrender.com/)

## Features

- **User authentication** — register, login, logout
- **Profile management** — update username, email, and avatar image (auto-resized to 300x300)
- **Blog posts** — create, read, update, delete with ownership protection
- **Author filtering** — view all posts by a specific user via `/user/<username>`
- **Pagination** — 5 posts per page
- **Rich UI** — Bootstrap 5 via `django-crispy-forms` with a steel-gray theme
- **Password reset** — email-based reset flow using Gmail SMTP
- **Responsive design** — works on desktop and mobile

## Tech Stack

- **Python** 3.14
- **Django** 6.0.6
- **Bootstrap 5** (via `crispy-bootstrap5`)
- **SQLite** (development) / **PostgreSQL** (production)
- **Pillow** — image processing for profile avatars
- **python-decouple** — environment variable management
- **WhiteNoise** — production static file serving
- **Gunicorn** — production WSGI server

## Project Structure

```
blog/
├── app/                    # Django project configuration
│   ├── settings.py         # Settings (dev + production)
│   ├── urls.py             # Root URL configuration
│   ├── wsgi.py             # WSGI entry point
│   └── asgi.py             # ASGI entry point
├── posts/                  # Blog posts app
│   ├── models.py           # Post model
│   ├── views.py            # Class-based views (CRUD + listing)
│   ├── urls.py             # Post URL patterns
│   ├── admin.py            # Admin registration
│   ├── templates/posts/    # Templates (home, detail, form, etc.)
│   └── static/posts/       # CSS styles
├── users/                  # User profiles app
│   ├── models.py           # Profile model (OneToOne with User)
│   ├── forms.py            # Registration + profile update forms
│   ├── views.py            # Register + profile views
│   ├── signals.py          # Auto-create/save profile on User events
│   ├── admin.py            # Profile admin registration
│   └── templates/users/    # Auth templates (login, register, etc.)
├── media/                  # User-uploaded files (avatars)
│   ├── default.jpg
│   └── profile_pics/
├── manage.py               # Django CLI
├── requirements.txt        # Python dependencies
├── Procfile                # Render deployment command
├── runtime.txt             # Python version for Render
└── posts.json              # Sample fixture data (30 posts)
```

## Local Development

### Prerequisites

- Python 3.14+
- pip

### Setup

```bash
# Clone the repository
git clone <repo-url>
cd blog

# Create and activate a virtual environment
python -m venv venv
source venv/bin/activate  # Linux/macOS
# venv\Scripts\activate   # Windows

# Install dependencies
pip install -r requirements.txt

# Apply migrations
python manage.py migrate

# (Optional) Load sample data
python manage.py loaddata posts.json

# Create a superuser
python manage.py createsuperuser

# Run the development server
python manage.py runserver
```

The app will be available at `http://localhost:8000`. The admin panel is at `http://localhost:8000/admin/`.

### Environment Variables

Create a `.env` file in the project root with the following variables (locally, only email credentials are required):

```env
EMAIL_HOST_USER=your-email@gmail.com
EMAIL_HOST_PASSWORD=your-gmail-app-password
```

For production, you will also set:
- `SECRET_KEY` — Django secret key (generated one is the fallback for local dev)
- `DEBUG` — `False` in production
- `ALLOWED_HOSTS` — comma-separated hostnames
- `DATABASE_URL` — PostgreSQL connection string
- `CSRF_TRUSTED_ORIGINS` — comma-separated origins

## Deployment to Render

### Prerequisites

1. A [Render](https://render.com) account
2. Your project pushed to a Git provider (GitHub, GitLab, etc.)

### Steps

1. From the Render Dashboard, click **New > Web Service**
2. Connect your Git repository
3. Configure the service:

   | Setting | Value |
   |---|---|
   | **Name** | `django-blog` (or your choice) |
   | **Region** | Closest to your users |
   | **Branch** | `main` |
   | **Root Directory** | `blog` |
   | **Runtime** | `Python 3` |
   | **Build Command** | `pip install -r requirements.txt && python manage.py collectstatic --noinput && python manage.py migrate` |
   | **Start Command** | `gunicorn app.wsgi --log-file -` |
   | **Plan** | Free or Starter |

4. Add the following **Environment Variables**:

   | Variable | Value |
   |---|---|
   | `SECRET_KEY` | Generate a secure key: `python -c "from django.core.management.utils import get_random_secret_key; print(get_random_secret_key())"` |
   | `DEBUG` | `False` |
   | `ALLOWED_HOSTS` | `your-app.onrender.com` |
   | `CSRF_TRUSTED_ORIGINS` | `https://your-app.onrender.com` |
   | `DATABASE_URL` | Leave blank for Render's SQLite disk, or provision a PostgreSQL database via Render Dashboard and paste the internal connection string |
   | `EMAIL_HOST_USER` | Your Gmail address |
   | `EMAIL_HOST_PASSWORD` | Your Gmail app password |

5. Click **Create Web Service**

Render will build and deploy your app. The first deployment may take a few minutes.

### Adding a Custom Domain

1. Go to your Web Service's **Settings** tab
2. Under **Custom Domain**, click **Add Domain**
3. Follow Render's DNS configuration instructions
4. Update `ALLOWED_HOSTS` and `CSRF_TRUSTED_ORIGINS` to include your domain

### PostgreSQL on Render

For a production database, provision a PostgreSQL instance:

1. From Render Dashboard, click **New > PostgreSQL**
2. Copy the **Internal Database URL**
3. Add it as the `DATABASE_URL` environment variable in your Web Service
4. Render ensures your web service can reach the database internally

## Admin Panel

Access `/admin/` with your superuser credentials to manage:
- **Posts** — view, edit, delete any post
- **Users** — manage user accounts
- **Profiles** — view and edit user profiles

## API / URL Reference

| URL | View | Description |
|---|---|---|
| `/` | `PostListView` | Paginated home page (5 posts per page) |
| `/user/<username>/` | `UserPostListView` | Posts by a specific author |
| `/post/<pk>/` | `PostDetailView` | Single post detail |
| `/post/new/` | `PostCreateView` | Create a new post (login required) |
| `/post/<pk>/update/` | `PostUpdateView` | Edit a post (owner only) |
| `/post/<pk>/delete/` | `PostDeleteView` | Delete a post (owner only) |
| `/about/` | `about` | Static about page |
| `/register/` | `register` | Create an account |
| `/login/` | `LoginView` | Sign in |
| `/logout/` | `LogoutView` | Sign out |
| `/profile/` | `profile` | View/edit profile |
| `/password-reset/` | `PasswordResetView` | Request password reset |
| `/admin/` | — | Django admin panel |

## License

This project is for educational purposes.
