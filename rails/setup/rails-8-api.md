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
  - Modified Rails 8 Authentication (JWT-pased)
  - Session invalidation and control
  - User roles

---

### Create a new rails application

Run the following command to create and setup the minimum requirements for the application

```bash
rails new my_app --api -T
cd my_app
```

This will create a new rails application named `my_app`, and will be API only, and will not generate any tests (as we will be using RSPEC as shown later)

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

Cleanup the `Gemfile` and uncomment the `rack-cors` gem, then run

```bash
bundle
git add --all
git commit -m "[chore] Gem cleanup"
```

---

### Rubocop

Configure Rubocop using the documentation at [Rubocop](./rubocop.md)

---

### OverCommit

Configure Overcommit using the documentation at [OverCommit](./overcommit.md)

---

### Testing

For testing we will be installing

- Rspec
- Database cleaner
- FactoryBot
- Faker
- Shoulda Matchers

To setup testing, use the documentation found at [Testing](testing.md)

---
