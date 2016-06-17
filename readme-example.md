# the project title

## Code Status

master: 

production: 

## Codestyle

* https://github.com/bbatsov/ruby-style-guide
* https://github.com/bbatsov/rails-style-guide

Use `rubocop` to check codestyle.

## Local codestyle conventions

**Controller**

* Don't leave controller view without corresponding method.

```ruby
# Example

# app/views/home/index.html.haml
%p Some content

# app/controllers/home_controller.rb
def index
  render
end

```

* Don't leave empty method. Add `render` at least.

## First Touch

System requirements:

* ruby (Check out ruby version in .ruby-version)
* postgresql
* redis (Used by sidekiq)
* imagemagick (Thumbnails generation. Check app/uploaders)
* phantomjs (Integration tests)

Configure database in config/database.yml. If you don't need password for
postgresql (e.g. have superadmin role in pg), you can do
`cp config/database.yml.example config/databse.yml`.

Run

```
cp config/auth.yml.example config/auth.yml
bundle install
bundle exec rake db:create
bundle exec rake db:migrate
bundle exec rake test
bundle exec rake test:integration
bundle exec rails server
```

If these commands work, then you have installed project successfully.

## Authentication

In development mode you can disable One Time Password (OTP) validation.
To do this - change `require_otp_validation` to false in config/auth.yml.

## Azure

To disable azure in `development` environment, add this to your `~/.bashrc`

```
export FAKE_AZURE=1
```

Then you should expect to see `Faking azure uploader` messages this when starting server

```
=> Booting Thin
=> Rails 4.2.1 application starting in development on http://0.0.0.0:3000
=> Run `rails server -h` for more startup options
=> Ctrl-C to shutdown server
Faking azure uploader for LoanPdfUploader!
Faking azure uploader for ReportFileUploader!
Faking azure uploader for LogoUploader!
Thin web server (v1.6.4 codename Gob Bluth)
Maximum connections set to 1024
Listening on 0.0.0.0:3000, CTRL+C to stop
```

## Mailer Testing

This is available only in development mode

1. Run
```
mailcatcher --http-ip 0.0.0.0
```

2. Send mails

3. See mails in the browser on `http://your-ip:1080/`

## Tests

Prefer tests over specs. Specs will be removed.

Running specs
```
bundle exec rspec spec
```

Running tests. It excludes integration tests because they require "truncation"
mode for database cleaner. By default database cleaner is in transaction mode,
which is considerably faster.
```
bundle exec rake test
```

Running integration tests
```
bundle exec rake test:integration
```

## Integration tests

When phantomjs tests failing, you can include `::Capybara::Screenshot::MiniTestPlugin`
to your test, and it will create screenshot + page.html file in tmp/capybara
for failing tests. **Don't commit it!**

Example:

```ruby
# test/integration/employees/csv_upload_test.rb
require 'test_helper'

module Employees
  class CSVUploadTest < IntegrationTest
    include ::Capybara::Screenshot::MiniTestPlugin
```

## Development

Don't make refactoring issues unless it will fix bugs or make easier to
integrate new functionality.

Always include tests with your code.

*Flow:*

* Don't work more than on 2 issues simultaneously
* Merge PR only when local tests green, CI green & reviewed
* Don't make more than 10 simultaneous PR's
* Never commit to master/production directly. Use only PR's
* Use `hub` utility for converting issue to pull-request, e.g. `hub pull-request -i 163`
* Making tasks:

1. Choose the next issue from the closest release by next priority:
  * `hotfix`
  * `bugfix`
  * `high_priority`
  * `low_priority`
  * everything else

2. Assign the issue.

3. Attach `in-progress` label.

4. Create new branch from fresh "master" (or from "production" if label is "hotfix").
   Branch name should start from issue number (e.g. `194-fix-sign-in`)

5. Make commits. Each commit description should start from issue number. (e.g. `#194 Fix user password`)

6. When ready - open pull-request on github.

7. Remove `in-progress` label, and attach `to-verify`

8. Reassign PR to reviewer.

9. When reviewed - rebase with base branch, squash commits and merge.

10. Delete github branch.


## "hotfix" issues

**hotfix** issue has the biggest priority and should be shipped to production ASAP.
Working on "hotfix" issue is almost same as on regular issue, except that you should create branch
from "production", not from "master".

In `hub` utility this will look like this

```
hub pull-request -i 1234 -b production
```

Before deploying hotfixes, one should create corresponding release and git tag for
including these changes. Prefer to increment minor version, e.g. `v1.0` -> `v1.0.1`.

After merging "hotfix" to production, don't forget cherry-pick commits to master!

## Releases

1. Create milestone with planned version name.

2. Assign priotirized issues to the milestone.

3. When releasing - clear this version milestone. Resolve tasks or move them to
   the next release.

4. Close the milestone

5. Create git tag.

6. Create release on github.

7. Checkout to recent release and deploy.

Versions including "hotfix" may not be assigned to milestones.

## Deployment

Ensure your public key is in `/home/deploy/.ssh/authorized_keys.` on
corresponding server.

Staging:

```
bundle exec mina staging deploy
```

Production

```
bundle exec mina production deploy
```

## Terms

* "OTP" means One Time Password. (https://en.wikipedia.org/wiki/Time-based_One-time_Password_Algorithm)
* "EE ID", "ER ID" - Employee ID, Employer ID
* "vesting requirement" in "repayment plan" model means "months after employee's hiring date"
