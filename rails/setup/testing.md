# Testing

For testing we will be installing

- Rspec
- Database cleaner
- FactoryBot
- Faker
- Shoulda Matchers

### Setup

- Start by updating your `Gemfile` to include the following:
  ```rb
  group :development, :test do
    gem "capybara"
    gem 'database_cleaner'
    gem 'dotenv-rails'
    gem 'factory_bot_rails'
    gem 'faker'
    gem 'pry-byebug'
    gem 'pry-rails'
    gem 'rspec-rails'
  end
  group :test do
    gem 'shoulda-matchers'
    gem "simplecov"
  end
  ```

- Run `bundle` and then run the following command:
  ```bash
  rails generate rspec:install
  ```


- Edit `spec/rails_helper.rb` to become as:
  ```rb
  require "spec_helper"
  ENV["RAILS_ENV"] ||= "test"
  require_relative "../config/environment"

  abort("The Rails environment is running in production mode!") if Rails.env.production?
  require "rspec/rails"

  require "database_cleaner"
  require "capybara/rspec"

  Rails.root.glob("spec/support/**/*.rb").each { |f| require f }

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
    config.before(:each, :js) { DatabaseCleaner.strategy = :truncation }
    config.before(:each) { DatabaseCleaner.start }
    config.after(:each) { DatabaseCleaner.clean }
    config.infer_spec_type_from_file_location!
    config.filter_rails_from_backtrace!

    Shoulda::Matchers.configure do |conf|
      conf.integrate do |with|
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
  RSpec.configure do |config|
    config.include FactoryBot::Syntax::Methods
  end
  ```

- Now, let's test our configuration with a simple test that we know should always pass. \
  Create a file named `spec/system/pass_spec.rb` with the following content:
  

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
