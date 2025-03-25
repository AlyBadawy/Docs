  # Database

For the application database, we will be using:

- PostgreSql as the primary database
- SQLite as the database for cache, queue, and cable.

### Setup

- Include the `pg` gem in your `Gemfile` by adding the following line

  ```rb
    gem "pg"
  ```

- Run `bundle` to install the `pg` gem.

- update your `config/database.yml` file to look like:

  ```yaml
  default: &default
    pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>
    timeout: 5000

  # SQLite3 configuration
  sqlite3: &sqlite3
    <<: *default
    adapter: sqlite3

  # PostgreSQL configuration
  postgres: &postgres
    <<: *default
    adapter: postgresql
    encoding: unicode
    username: <%= ENV.fetch('DATABASE_USERNAME') {'postgres'} %>
    password: <%= ENV.fetch('DATABASE_PASSWORD') {'postgres'} %>
    host: <%= ENV.fetch('DATABASE_HOST') {'localhost'} %>

  development:
    <<: *postgres
    database: <%= "#{ENV.fetch('APP_NAME') {'anubis_backend_engine'}}_development" %>

  test:
    <<: *postgres
    database: <%= "#{ENV.fetch('APP_NAME') {'anubis_backend_engine'}}_test" %>

  production:
    primary:
      <<: *postgres
      database: storage/production.sqlite3
    cache:
      <<: *sqlite3
      database: storage/production_cache.sqlite3
      migrations_paths: db/cache_migrate
    queue:
      <<: *sqlite3
      database: storage/production_queue.sqlite3
      migrations_paths: db/queue_migrate
    cable:
      <<: *sqlite3
      database: storage/production_cable.sqlite3
      migrations_paths: db/cable_migrate
  ```

- Run the following commands:

  ```bash
  bundle
  bundle exec rails db:create
  ```

- Commit and push your changes.

### Setup UUID as default ID

To enable and use UUID as the default ID for models in your project while using Postgresql:

#### Create a migration to enable the UUID extensions:

- Issue the following command to create a migration:

  ```bash
  rails g migration EnableUUID
  ```

- This will create a migration file, change the content of the migration file to:

  ```rb
  class EnableUuid < ActiveRecord::Migration[7.0]
    def change
      enable_extension 'pgcrypto'
    end
  end
  ```

#### Use UUID by default:

- Change the migrations generator to add UUID as the default ID type by creating the file config/initializers/generators.rb with the following content:

  ```rb
  Rails.application.config.generators do |g|
    g.orm :active_record, primary_key_type: :uuid
  end
  ```

#### Bring back rails .first and .last methods:

Because we now use UUID instead of an incremental number as the idea, rails may get confused when trying to sort the objects of a given model, and hence we lose the .first and .last functionality.

To bring that back, and to sort objects correctly, we will use created_at as the column to sort instead of id. We also need to generate the UUID on new records creation

- Change `app/models/application_record.rb` to be line:

  ```rb
  class ApplicationRecord < ActiveRecord::Base
    primary_abstract_class

    self.implicit_order_column = :created_at

    before_create :generate_uuid_v7

    private

    def generate_uuid_v7
      return if self.class.attribute_types["id"].type != :uuid

      self.id ||= Random.uuid_v7
    end
  end
  ```

- Run the following command:

  ```rb
  bundle exec rails db:drop db:create db:migrate
  ```
