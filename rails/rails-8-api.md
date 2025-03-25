# Setup rails 8 Api Application

In this documentation, I will document setting up a rails 8 application (API only) with the usual templates that I start all rails apps with.

This setup will include the following

- Database:
  - PostgreSQL for the main application database.
  - SQLite for `Solid Queue`, `Solid Cable`, and `Solid Cache`
- Testing:
  - Rspec
  - FactoryBot
  - Faker
- Linting:
  - Rubocop
- Authentication:
  - Modified Rails 8 Authentication (JWT-Based)
  - Session invalidation and control
  - User roles

---

### Create a new rails application

Run the following command to create and setup the minimum requirements for the application

```bash
rails new . --api -T
cd my_app
```

This will create a new rails application in the current directory (and will be named based on the directory name), will be API only, and will not generate any tests (as we will be using RSPEC as shown later).

**Note** \
We didn't specify the database here as we will be using a mix of Postgresql and SQLite (discussed in [databases](./setup/database.md)), and it's easier to start with the default SQLite and then add PostgreSQL.

#### Git

It's a good practice to use source control for your project, and we will start by adding `.DS_Store` to our `.gitignore` file, and commit the initial commit.

To do so, issue the following commands:

```bash
echo "" >> .gitignore
echo ".DS_Store" >> .gitignore
git add --all
git commit -m "Initial commit"
```

At this point, you may want to create a repository on github and push this commit to it.

---

### Gemfile

- Cleanup the `Gemfile` (Optional, but recommended).
- Uncomment the `rack-cors` gem
- Uncomment the `jbuilder` gem

- Run the following command:

  ```bash
  bundle
  git add --all
  git commit -m "[chore] Gem cleanup"
  ```

---

### Testing

For testing we will be installing

- Rspec
- Database cleaner
- FactoryBot
- Faker
- Shoulda Matchers

To setup testing, use the documentation found at [Testing](./setup/testing.md)

---

### Rubocop

Configure Rubocop using the documentation at [Rubocop](./setup/rubocop.md)

---

### OverCommit

Configure Overcommit using the documentation at [OverCommit](./setup/overcommit.md)

---

### Database

Use the documentation in [Database](./setup/database.md) to setup the database

---

### CI and Dependabot

Use the documentation in [CI and Dependabot](./setup/ci_and_dependabot.md) to configure both for better code quality

---

### Authentication

We will be building the authentication for the application to have user roles, authentication using JWT tokens, Refresh and invalidate token mechanisms, account management and more.

Now, head to the [Authentication Setup](./auth/setup-auth.md) Docummentation.
