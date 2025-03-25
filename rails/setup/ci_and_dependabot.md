

### Add testing to CI:

- To add testing to CI, we will edit the `.github/workflows/ci.yml` file that was generated when we created the Rails 8 application, to include the following job:

  ```yml
  test:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      checks: write
    services:
      postgres:
        image: postgres:latest
        ports:
          - 5432:5432
        env:
          POSTGRES_USER: postgres
          POSTGRES_PASSWORD: postgres
          POSTGRES_DB: test_db
        options: >-
          --health-cmd "pg_isready -U postgres"
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
    steps:
      - name: Checkout code
        uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683
      - name: Set up Ruby
        uses: ruby/setup-ruby@277ba2a127aba66d45bad0fa2dc56f80dbfedffa
        with:
          ruby-version: .ruby-version
          bundler-cache: true
      - name: Install dependencies
        run: bundle install
      - name: Wait for PostgreSQL to be ready
        run: |
          while ! pg_isready -h localhost -p 5432 -U postgres; do
            sleep 1
          done
      - name: Create and setup the database
        run: |
          bundle exec rails db:create
          bundle exec rails db:schema:load
      - name: Run SimpleCov to check test coverage
        run: bundle exec rspec
        env:
          SIMPLECOV_MINIMUM_COVERAGE: 95
  ```

  - Commit and push your code.

### Configure Dependabot:
  - Make sure you enable dependabot in the Github repository settings. then update (or create if not exist) the `.github/dependabot.yml` file to look like:
  ```yml
  version: 2
  updates:
    - package-ecosystem: bundler
      directory: '/'
      schedule:
        interval: weekly
      open-pull-requests-limit: 15
      groups:
        prod-dependencies:
          dependency-type: 'production'
          update-types:
            - 'minor'
            - 'patch'
        dev-dependencies:
          dependency-type: 'development'
          update-types:
            - 'minor'
            - 'patch'
    - package-ecosystem: github-actions
      directory: '/'
      schedule:
        interval: weekly
      open-pull-requests-limit: 15
  ```
  - Commit and push your code.
