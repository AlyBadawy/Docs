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

Change the contents of `.rubocop.yml` to match the following:

```yaml
inherit_gem:
  rubocop-rails-omakase: rubocop.yml
  rubocop-config-prettier: config/rubocop.yml

require:
  - rubocop-performance
  - rubocop-rails
  - rubocop-rspec

AllCops:
  NewCops: enable
  Exclude:
    - bin/yarn
    - db/schema.rb
    - vendor/**/*
    - node_modules/**/*
    - tmp/**/*
    - public/**/*
    - bin/**/*
    - data/**/*

Bundler/DuplicatedGem:
  Enabled: true
  Include:
    - '**/*.gemfile'
    - '**/Gemfile'
    - '**/gems.rb'
Bundler/GemComment:
  Enabled: false
Bundler/GemVersion:
  Enabled: false
  EnforcedStyle: 'required'
  Include:
    - '**/*.gemfile'
    - '**/Gemfile'
    - '**/gems.rb'
  AllowedGems: []
Bundler/OrderedGems:
  Enabled: true
  TreatCommentsAsGroupSeparators: true
  ConsiderPunctuation: false
  Include:
    - '**/*.gemfile'
    - '**/Gemfile'
    - '**/gems.rb'

Layout/AccessModifierIndentation:
  Enabled: true
  EnforcedStyle: indent
Layout/ArgumentAlignment:
  Enabled: true
  EnforcedStyle: with_first_argument
Layout/ArrayAlignment:
  Enabled: true
  EnforcedStyle: with_first_element
Layout/CaseIndentation:
  Enabled: true
  EnforcedStyle: end
Layout/ClassStructure:
  Enabled: true
  Categories:
    module_inclusion:
      - include
      - prepend
      - extend
  ExpectedOrder:
    - module_inclusion
    - constants
    - public_class_methods
    - initializer
    - public_methods
    - protected_methods
    - private_methods
Layout/CommentIndentation:
  Enabled: true
  AllowForAlignment: true
Layout/DotPosition:
  Enabled: true
  EnforcedStyle: leading
Layout/EmptyLineBetweenDefs:
  Enabled: true
  EmptyLineBetweenMethodDefs: true
  EmptyLineBetweenClassDefs: true
  EmptyLineBetweenModuleDefs: true
  AllowAdjacentOneLineDefs: true
  NumberOfEmptyLines: 1
Layout/EmptyLinesAroundAccessModifier:
  Enabled: true
  EnforcedStyle: around
Layout/HashAlignment:
  Enabled: true
  AllowMultipleStyles: true
  EnforcedHashRocketStyle: key
  EnforcedColonStyle: key
Layout/IndentationStyle:
  Enabled: true
  EnforcedStyle: spaces
Layout/IndentationWidth:
  Width: 2
Layout/MultilineMethodCallIndentation:
  Enabled: true
  EnforcedStyle: indented
Layout/SpaceAroundEqualsInParameterDefault:
  Enabled: true

Lint/Debugger:
  Enabled: true
  Severity: error

Metrics/AbcSize:
  Max: 50
Metrics/BlockLength:
  CountComments: false
  Max: 50
  AllowedMethods:
    - context
    - describe
    - it
    - shared_examples
    - shared_examples_for
    - namespace
    - draw
    - configure
    - group
  Exclude:
    - config/routes.rb
    - config/seeds.rb
Metrics/ClassLength:
  Enabled: true
  Max: 200
Metrics/ModuleLength:
  Enabled: true
  Max: 250
Metrics/MethodLength:
  Enabled: true
  Max: 20
  Exclude:
    - 'db/migrate/*.rb'

Rails:
  Enabled: true
Rails/ApplicationController:
  Enabled: true
Rails/FilePath:
  Enabled: false
Rails/ResponseParsedBody:
  Enabled: false

RSpec:
  Enabled: true
RSpec/DescribeClass:
  Enabled: false
RSpec/ExampleLength:
  Enabled: true
  Max: 25
RSpec/HookArgument:
  Enabled: false
RSpec/InstanceVariable:
  Enabled: false
RSpec/NestedGroups:
  Enabled: true
  Max: 5
RSpec/MultipleExpectations:
  Enabled: true
  Max: 5

Style/Documentation:
  Enabled: false
Style/FrozenStringLiteralComment:
  Enabled: false
Style/StringLiterals:
  Enabled: true
  EnforcedStyle: double_quotes
Style/QuotedSymbols:
  Enabled: true
  EnforcedStyle: single_quotes
```

Now, install the plugins used by `rubocop` as gem by modifying the `Gemfile` to include the following group:

```rb
group :development do
  gem "rubocop"
  gem "rubocop-config-prettier"
  gem "rubocop-performance"
  gem "rubocop-rails"
  gem "rubocop-rspec"
end
```

Run rubocop and commit:

```bash
bundle
rubocop -A
git add --all
git commit -m "[chore] Rubocop setup"
```

---

### Overcommit

`Overcommit` is a popular gem for managing Git hooks in a Rails application. Git hooks are scripts that run automatically at certain points in the Git workflow, such as before a commit is made or after a pull request is merged. Overcommit provides an easy way to manage and configure these hooks, allowing you to enforce coding standards, run tests, and perform other checks before changes are committed to the repository. Overcommit can be customized to fit the needs of your project, with a wide variety of pre-built hooks available out of the box. By using Overcommit, you can improve the quality and consistency of your codebase, while also catching errors and potential issues earlier in the development process.

We use Overcommit to make sure the code quality and meeting specific standards before committing a change to the git repository

Start by adding overcommit to the GemFile as:

```rb
group :development do
  gem 'overcommit'
end
```

then run `bundle install` and create a file named `.overcommit.yml` with the contents of:

```yaml
PostCheckout:
  BundleInstall:
    enabled: true

PreCommit:
  ALL:
    problem_on_unmodified_line: warn
    requires_files: true
    required: false
    quiet: false
  AuthorName:
    enabled: false
  BundleAudit:
    enabled: true
    flags: ['--update']
  BundleCheck:
    enabled: true
  EsLint:
    enabled: true
    # https://github.com/sds/overcommit/issues/338
    required_executable: 'yarn'
    command: ['yarn', 'eslint']
    flags: []
    include:
      [
        'app/javascript/**/*.ts',
        'app/javascript/**/*.tsx',
        'app/javascript/**/*.js',
        'app/javascript/**/*.js',
      ]

  Fasterer:
    enabled: true
    exclude:
      - 'vendor/**/*.rb'
      - 'db/schema.rb'
  ForbiddenBranches:
    enabled: true
    branch_patterns: ['main']
  HamlLint:
    enabled: true
  MergeConflicts:
    enabled: true
    exclude:
      - '**/conflict/file_spec.rb'
      - '**/git/conflict/parser_spec.rb'
  # prettier? https://github.com/sds/overcommit/issues/614 https://github.com/sds/overcommit/issues/390#issuecomment-495703284
  Prettier:
    enabled: true
    command: ['npx', 'prettier']
    flags: ['-c']
    description: 'Ensure Prettier is used to format JS'
    include:
      - '**/*.js*'
      - '**/*.ts*'
      - '**/*.json'
  RuboCop:
    enabled: true
    command: ['bundle', 'exec', 'rubocop']
  #    on_warn: fail # Treat all warnings as failures
  ScssLint:
    enabled: true
  MarkdownLint:
    enabled: true
    description: 'Lint documentation for Markdown errors'
    required_executable: 'node_modules/.bin/markdownlint'
    flags: ['--config', '.markdownlint.yml', 'doc/**/*.md']
    install_command: 'yarn install'
    include:
      - 'doc/**/*.md'
  Vale:
    enabled: true
    description: 'Lint documentation for grammatical and formatting errors'
    required_executable: 'vale'
    flags: ['--config', '.vale.ini', '--minAlertLevel', 'error', 'doc']
    install_command: 'brew install vale # (or use another package manager)'
    include:
      - 'doc/**/*.md'
PrePush:
  # Unit & Integration TEST
  RSpec:
    enabled: true
    command: ['bundle', 'exec', 'rspec', 'spec']
    on_warn: fail
  BundleInstall:
    enabled: true
    on_warn: fail

CommitMsg:
  TextWidth:
    enabled: true
    min_subject_width: 8 # three 2-letter words with 2 spaces
    max_subject_width: 72
    quiet: false

  EmptyMessage:
    enabled: true
    required: true
    description: 'Checking for empty commit message'
```

then run the command:

```bash
git checkout -b overcommit
bundle
overcommit --install
overcommit --sign pre-commit
git add --all
git commit -m "[chore] setup and install Overcommit"
git checkout main
```

---

### Testing

For testing we will be installing

- Rspec
- Database cleaner
- FactoryBot
- Faker
- Shoulda Matchers

Start by updating your `Gemfile` to include the following:

```rb
group :development, :test do
  gem "capybara"
  gem 'database_cleaner'
  gem 'dotenv-rails'
  gem 'factory_bot_rails'
  gem 'faker'
  gem 'pry-byebug'
  gem 'pry-rails'
end
group :test do
  gem 'rspec-rails'
  gem 'shoulda-matchers'
  gem "simplecov"
end
```

Run `bundle` and then:

- Edit `spec/rails_helper.rb` to become as:

```rb

require "spec_helper"
ENV["RAILS_ENV"] ||= "test"
require_relative "../config/environment"

abort("The Rails environment is running in production mode!") if Rails.env.production?
require "rspec/rails"

require "database_cleaner"
require "capybara/rspec"

Dir[Rails.root.join("spec", "support", "**", "*.rb")].each { |f| require f }

begin
  ActiveRecord::Migration.maintain_test_schema!
rescue ActiveRecord::PendingMigrationError => e
  abort e.to_s.strip
end
RSpec.configure do |config|
  config.include ActiveSupport::Testing::TimeHelpers

  config.fixture_path = "#{Rails.root}/spec/fixtures"
  config.use_transactional_fixtures = true
  config.before(:suite) { DatabaseCleaner.clean_with(:truncation) }
  config.before(:each) { DatabaseCleaner.strategy = :transaction }
  config.before(:each, js: true) { DatabaseCleaner.strategy = :truncation }
  config.before(:each) { DatabaseCleaner.start }
  config.after(:each) { DatabaseCleaner.clean }
  config.infer_spec_type_from_file_location!
  config.filter_rails_from_backtrace!

  Shoulda::Matchers.configure do |conf|
    conf.integrate do |with|
      # Choose a test framework:
      with.test_framework :rspec
      with.library :rails
    end
  end
end
```

- Edit `spec/spec_helper.rb` to become as:

```rb

require "simplecov"
SimpleCov.start

RSpec.configure do |config|
  config.expect_with :rspec do |expectations|
    expectations.include_chain_clauses_in_custom_matcher_descriptions = true
  end

  config.mock_with :rspec do |mocks|
    mocks.verify_partial_doubles = true
  end
  config.shared_context_metadata_behavior = :apply_to_host_groups
end
```

- Create a `spec/support/factory_bot.rb` file with this content:

```rb
require "rails_helper"

RSpec.describe "Test" do
  it "Passes" do
    expect(1 + 1).to eq(2)
  end
end
```

- Run `bundle exec rspec` and make sure the test passes.
- Commit and push your code.

---
