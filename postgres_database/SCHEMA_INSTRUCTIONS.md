PostgreSQL schema bootstrap guide

Use the connection string from db_connection.txt. Execute each SQL as a single -c statement:

-- Create tables
psql postgresql://appuser:dbuser123@localhost:5000/myapp -c "CREATE TABLE IF NOT EXISTS users (id SERIAL PRIMARY KEY, email VARCHAR(255) UNIQUE NOT NULL, password_hash VARCHAR(255) NOT NULL, created_at TIMESTAMP DEFAULT NOW(), is_active BOOLEAN DEFAULT TRUE);"

psql postgresql://appuser:dbuser123@localhost:5000/myapp -c "CREATE TABLE IF NOT EXISTS categories (id SERIAL PRIMARY KEY, name VARCHAR(100) UNIQUE NOT NULL, slug VARCHAR(120) UNIQUE NOT NULL);"

psql postgresql://appuser:dbuser123@localhost:5000/myapp -c "CREATE TABLE IF NOT EXISTS recipes (id SERIAL PRIMARY KEY, title VARCHAR(200) NOT NULL, description TEXT, instructions TEXT, image_url VARCHAR(500), ingredients TEXT, created_at TIMESTAMP DEFAULT NOW());"

psql postgresql://appuser:dbuser123@localhost:5000/myapp -c "CREATE TABLE IF NOT EXISTS recipe_categories (recipe_id INT NOT NULL REFERENCES recipes(id) ON DELETE CASCADE, category_id INT NOT NULL REFERENCES categories(id) ON DELETE CASCADE, CONSTRAINT uix_recipe_category UNIQUE (recipe_id, category_id));"

psql postgresql://appuser:dbuser123@localhost:5000/myapp -c "CREATE TABLE IF NOT EXISTS favorites (id SERIAL PRIMARY KEY, user_id INT NOT NULL REFERENCES users(id) ON DELETE CASCADE, recipe_id INT NOT NULL REFERENCES recipes(id) ON DELETE CASCADE, created_at TIMESTAMP DEFAULT NOW(), CONSTRAINT uix_user_recipe_fav UNIQUE (user_id, recipe_id));"

-- Seed categories
psql postgresql://appuser:dbuser123@localhost:5000/myapp -c "INSERT INTO categories (name, slug) VALUES ('Breakfast','breakfast') ON CONFLICT (slug) DO NOTHING;"
psql postgresql://appuser:dbuser123@localhost:5000/myapp -c "INSERT INTO categories (name, slug) VALUES ('Lunch','lunch') ON CONFLICT (slug) DO NOTHING;"
psql postgresql://appuser:dbuser123@localhost:5000/myapp -c "INSERT INTO categories (name, slug) VALUES ('Dinner','dinner') ON CONFLICT (slug) DO NOTHING;"
psql postgresql://appuser:dbuser123@localhost:5000/myapp -c "INSERT INTO categories (name, slug) VALUES ('Dessert','dessert') ON CONFLICT (slug) DO NOTHING;"
psql postgresql://appuser:dbuser123@localhost:5000/myapp -c "INSERT INTO categories (name, slug) VALUES ('Vegan','vegan') ON CONFLICT (slug) DO NOTHING;"

-- Seed demo user (password hash must match backend hashing; use app to signup in development)
-- Optionally insert directly if needed:
-- psql ... -c \"INSERT INTO users (email, password_hash) VALUES ('demo@example.com', '<hash>') ON CONFLICT (email) DO NOTHING;\"

Notes:
- The backend also auto-creates tables via SQLAlchemy on startup. Prefer running through backend for dev. This guide helps for DB-only bootstrap.
