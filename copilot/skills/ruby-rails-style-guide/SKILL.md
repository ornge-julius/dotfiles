---
name: ruby-rails-style-guide
description: Applies the Ruby on Rails style guide to Ruby and Rails code, including Rails conventions for routing, controllers, models, queries, migrations, views, I18n, mailers, time, and testing. Use when writing or reviewing Ruby/Rails code or when the user mentions Rails style or the Rails style guide.
---
# Ruby + Rails Style Guide (Rails RuboStyle)

## Purpose
Use the Rails style guide at https://rails.rubystyle.guide/ as the source of truth for Ruby on Rails conventions. Apply general Ruby style when it supports Rails readability, but prioritize Rails-specific rules from the guide.

## When to Apply
- Writing or reviewing Rails code (controllers, models, routes, migrations, views, mailers, jobs).
- Refactoring existing Rails code for readability or consistency.
- Answering questions about Rails best practices or style.

## Core Principles
- Keep controllers skinny, move business logic to models or service objects.
- Prefer readable, explicit Rails APIs over metaprogramming or implicit behavior.
- Avoid surprising behavior (unsafe SQL, skip validations, ambiguous ordering).
- Favor conventions and defaults unless there is a strong reason to deviate.

## Configuration
- Place custom initialization in config/initializers; one file per gem.
- Keep env-specific configuration in config/environments.
- Keep shared config in config/application.rb.
- Use config.load_defaults matching the Rails version.
- Prefer config_for with YAML files under config/.

## Routing
- Use RESTful resources and member/collection routes instead of ad-hoc routes.
- Prefer shallow nesting when nesting more than one level.
- Use namespaces for admin/segmented areas.
- Never use wild controller routes; avoid match unless multiple verbs are needed.

### Routing Examples
```ruby
# bad
get 'subscriptions/:id/unsubscribe'
resources :subscriptions

# good
resources :subscriptions do
  get 'unsubscribe', on: :member
end

# bad
match 'photos/search', to: 'photos#search'

# good
resources :photos do
  get 'search', on: :collection
end

# good: shallow nesting
resources :posts, shallow: true do
  resources :comments do
    resources :versions
  end
end
```

## Controllers
- Keep actions small; ideally one method besides lookup/build.
- Minimize instance variables shared with views.
- Keep action filters lexically scoped to avoid surprises.
- Prefer templates to inline rendering.
- Use render plain: and HTTP status symbols (e.g., :forbidden).

### Controller Examples
```ruby
# bad: inline rendering
def index
  render inline: "<% products.each do |p| %><p><%= p.name %></p><% end %>", type: :erb
end

# good: render template
def index
  render :index
end

# bad
render text: 'Forbidden', status: 403

# good
render plain: 'Forbidden', status: :forbidden
```

## Models
- Use meaningful, non-abbreviated model names.
- Prefer non-AR models for domain objects; use ActiveModel::Model when needed.
- Keep formatting helpers out of models; use helpers/decorators/presenters.

### Active Record
- Avoid changing AR defaults (table_name, primary_key) unless required.
- Append to ignored_columns rather than overwriting it.
- Use hash syntax for enum values.
- Group macro-style declarations at top (scopes, constants, attrs, enums, associations, validations, callbacks).
- Prefer has_many :through over HABTM.
- Use self[:attr] for read/write of raw attributes.
- Use new-style validations and validate one attribute per line.
- Name custom validation methods clearly; use custom validators for reuse.
- Keep custom validators in app/validators (or extract to shared gem when needed).
- Order callbacks in execution order; use prepend: true for before_destroy validations.
- Define dependent: on has_many/has_one.
- Prefer save!/create!/update! or handle false returns explicitly.
- Use user-friendly URLs (to_param or friendly_id).
- Use find_each for large batches.

### Active Record Examples
```ruby
# bad: implicit enum values
enum status: %i[draft published]

# good: explicit enum values
enum status: { draft: 0, published: 1 }

# bad: old-style validation
validates_presence_of :email

# good: new-style validation
validates :email, presence: true

# bad: HABTM
has_and_belongs_to_many :groups

# good: has_many :through
has_many :memberships
has_many :groups, through: :memberships
```

```ruby
# good: callbacks ordered as executed
class Person < ApplicationRecord
  before_validation :normalize_name
  after_commit :audit_change
end

# good: before_destroy with prepend
before_destroy :ensure_deletable, prepend: true
```

### Active Record Queries
- Avoid SQL interpolation; use bind parameters or hash conditions.
- Prefer named placeholders when multiple parameters exist.
- Use find for primary key lookup; find_by for attribute lookup.
- Prefer where.missing for missing relationships (Rails 6.1+).
- Avoid ordering by id for chronological order; use timestamps.
- Use symbol order (order(created_at: :desc)).
- Use pluck/pick/ids appropriately.
- Use squished heredocs for find_by_sql.
- Prefer size or length over count depending on intent.
- Use range conditions for date/time filtering.
- Avoid where.not with multiple attributes (use SQL string with AND).
- Avoid redundant .all receivers.
- Do not memoize find_by with ||=; use defined? guard.

### Query Examples
```ruby
# bad: SQL interpolation
Client.where("orders_count = #{params[:orders]}")

# good: bind params
Client.where('orders_count = ?', params[:orders])

# good: named placeholders
Client.where(
  'orders_count >= :min AND country_code = :country',
  min: params[:min], country: params[:country]
)

# good: find vs find_by
User.find(id)
User.find_by(email: email)
```

```ruby
# bad: ordering by id
scope :chronological, -> { order(id: :asc) }

# good: order by timestamp
scope :chronological, -> { order(created_at: :asc) }

# good: ranges
User.where(created_at: 30.days.ago..7.days.ago)
```

```ruby
# bad: where.not with multiple attrs
User.where.not(status: 'active', plan: 'basic')

# good
User.where.not('status = ? AND plan = ?', 'active', 'basic')
```

## Migrations
- Keep schema.rb or structure.sql in version control.
- Use db:schema:load for fresh DB setup.
- Enforce defaults in migrations, not in model getters.
- Avoid 3-state booleans: set default and null: false.
- Add foreign keys explicitly and name them meaningfully.
- Use change for reversible migrations; use up/down when needed.
- Define migration-local model classes if using models in migrations.

### Migration Examples
```ruby
# good: boolean with default and null constraint
add_column :users, :active, :boolean, default: true, null: false

# good: foreign keys
create_table :comments do |t|
  t.references :article, foreign_key: true
  t.belongs_to :user, foreign_key: true
end
```

```ruby
# good: change method for reversible migration
class AddNameToPeople < ActiveRecord::Migration
  def change
    add_column :people, :name, :string
  end
end

# good: up/down for non-reversible
class DropUsers < ActiveRecord::Migration
  def up
    drop_table :users
  end

  def down
    create_table :users do |t|
      t.string :name
    end
  end
end
```

## Views
- Do not call models directly from views.
- Keep formatting simple in views; use helpers/decorators for complex logic.
- Use partials; pass locals to partials, not instance variables.

### View Examples
```erb
<!-- bad: instance variable in partial -->
<%= render 'course_description' %>
<!-- _course_description.html.erb -->
<%= @course.description %>

<!-- good: locals in partial -->
<%= render 'course_description', course: @course %>
<!-- _course_description.html.erb -->
<%= course.description %>
```

## I18n
- Do not hardcode locale strings in views/models/controllers.
- Use activerecord.* scope for model/attribute translations.
- Organize locale files by domains (models vs views) and add load_path.
- Use I18n.t and I18n.l short forms.
- Use lazy lookup in views (t '.title').
- Prefer dot-separated keys over scope arrays.

### I18n Examples
```ruby
# bad
I18n.t :record_invalid, scope: [:activerecord, :errors, :messages]

# good
I18n.t 'activerecord.errors.messages.record_invalid'
```

```erb
<!-- good: lazy lookup in views -->
<%= t '.title' %>
```

## Assets
- app/assets for app-specific assets.
- lib/assets for internal libraries.
- vendor/assets for third-party assets.
- Prefer gemified assets when possible.

## Mailers
- Name mailers SomethingMailer.
- Provide both HTML and plain-text templates.
- Enable delivery errors in development.
- Use local SMTP in development; set default host name.
- Format email addresses with name + email.
- Use smtp delivery in dev/prod and test in test env.
- Inline CSS for HTML emails; use premailer/roadie if needed.
- Send emails in background jobs.

## Active Support Core Extensions
- Prefer &. over try!.
- Prefer Ruby stdlib methods over AS aliases and uncommon extensions.
- Prefer exclude? over !include?.
- Prefer squiggly heredoc (<<~) over strip_heredoc.
- Prefer to_fs over to_formatted_s (Rails 7+).

## Time and Duration
- Configure time_zone and prefer Time.zone.parse, Time.current.
- Avoid Time.parse, String#to_time, Time.now.
- Prefer date.all_day/all_week/all_month/all_quarter/all_year over begin/end ranges.
- Use from_now/ago without args; since/after/before/until with args.
- Prefer duration arithmetic (2.days.from_now) over Time.current +/-.

### Time Examples
```ruby
# bad
Time.now
Time.parse('2015-03-02 19:05:37')

# good
Time.current
Time.zone.parse('2015-03-02 19:05:37')

# good
date.all_day
```

### Duration Examples
```ruby
# good
2.days.from_now
5.hours.ago

# bad
Time.current + 2.days
Time.current - 5.hours
```

## Bundler
- Group dev/test gems in appropriate groups.
- Use established gems; review small/uncommon gems.
- Keep Gemfile.lock committed.

## Testing
- Prefer integration tests (ActionDispatch::IntegrationTest) over controller tests.
- Prefer freeze_time over travel_to(Time.current).

### Testing Examples
```ruby
# bad
class UsersControllerTest < ActionController::TestCase
end

# good
class UsersControllerTest < ActionDispatch::IntegrationTest
end

# good
freeze_time
```

## Tooling Guidance
- When asked about enforcing the style guide, suggest rubocop-rails as it aligns with the guide.

## Extended Examples
For a larger set of Good/Bad examples, see [references/EXAMPLES.md](references/EXAMPLES.md).

## Quick Examples
```ruby
# Good: controller action stays slim
class UsersController < ApplicationController
  def show
    @user = User.find(params[:id])
  end
end

# Good: range query, safe placeholder, order symbol
User.where(created_at: 30.days.ago..7.days.ago)
    .where(status: :active)
    .order(created_at: :desc)

# Good: enum with explicit values
enum status: { draft: 0, published: 1 }
```

## References
- Rails style guide: https://rails.rubystyle.guide/
