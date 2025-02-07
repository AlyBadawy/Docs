# Overcommit

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
