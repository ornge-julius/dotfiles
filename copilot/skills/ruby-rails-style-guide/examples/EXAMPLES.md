# Ruby + Rails Style Guide - Extended Examples

This file expands on the main skill with more Good/Bad examples. Use it when you need concrete patterns or are reviewing tricky Rails code.

## Routing
```ruby
# bad: ad-hoc route
+get 'reports/:id/archive'
+resources :reports
+
# good: member route
+resources :reports do
+  post 'archive', on: :member
+end
```

```ruby
# bad: deep nesting without shallow
+resources :accounts do
+  resources :users do
+    resources :sessions do
+      resources :events
+    end
+  end
+end
+
# good: shallow nesting
+resources :accounts, shallow: true do
+  resources :users do
+    resources :sessions do
+      resources :events
+    end
+  end
+end
```

```ruby
# bad: match without via
+match 'payments/refund', to: 'payments#refund'
+
# good: explicit verb
+post 'payments/refund', to: 'payments#refund'
```

## Controllers
```ruby
# bad: action with business logic
+def create
+  user = User.new(user_params)
+  user.plan = Plan.find_by(code: params[:plan_code])
+  user.discount = DiscountCalculator.compute(user)
+  user.save!
+  redirect_to user
+end
+
# good: delegate business logic
+def create
+  user = Users::Creator.call(user_params, plan_code: params[:plan_code])
+  redirect_to user
+end
```

```ruby
# bad: too many ivars
+def show
+  @user = User.find(params[:id])
+  @orders = @user.orders
+  @favorites = @user.favorites
+  @addresses = @user.addresses
+end
+
# good: reduce surface area
+def show
+  @user = User.includes(:orders, :favorites, :addresses).find(params[:id])
+end
```

```ruby
# bad: non-lexical filter target
+before_action :require_admin, only: :destroy
+
+# good: keep action and filter close
+before_action :require_admin, only: :destroy
+
+def destroy
+end
```

## Models
```ruby
# bad: abbreviated model name
+class Acct < ApplicationRecord
+end
+
# good
+class Account < ApplicationRecord
+end
```

```ruby
# bad: formatting in model
+def name_html
+  "<strong>#{name}</strong>".html_safe
+end
+
# good: helper or presenter
+module UsersHelper
+  def name_html(user)
+    content_tag(:strong, user.name)
+  end
+end
```

```ruby
# good: ActiveModel for non-AR objects
+class ContactForm
+  include ActiveModel::Model
+  attr_accessor :name, :email, :message
+
+  validates :name, presence: true
+  validates :email, format: { with: /\A[^@\s]+@[^@\s]+\z/ }
+end
```

### Active Record
```ruby
# bad: overwriting ignored_columns
+class LegacyOrder < ApplicationRecord
+  self.ignored_columns = %i[old_field]
+end
+
# good: append
+class LegacyOrder < ApplicationRecord
+  self.ignored_columns += %i[old_field]
+end
```

```ruby
# good: macro order
+class User < ApplicationRecord
+  default_scope { where(active: true) }
+
+  STATUSES = %w[pending active].freeze
+
+  attr_accessor :temporary_token
+
+  enum role: { user: 0, admin: 1 }
+
+  has_many :orders, dependent: :destroy
+
+  validates :email, presence: true
+
+  before_save :normalize_email
+end
```

```ruby
# bad: single validation for many attributes
+validates :email, :name, presence: true
+
# good
+validates :email, presence: true
+validates :name, presence: true
```

```ruby
# bad: unclear validation name
+validate :name_present
+
# good: validation reads like a sentence
+validate :name_must_be_present
```

```ruby
# good: has_many :through
+class Membership < ApplicationRecord
+  belongs_to :user
+  belongs_to :group
+end
+
+class User < ApplicationRecord
+  has_many :memberships
+  has_many :groups, through: :memberships
+end
```

```ruby
# good: to_param
+def to_param
+  "#{id}-#{title}".parameterize
+end
```

```ruby
# good: find_each for batch processing
+User.where(active: true).find_each do |user|
+  user.refresh_stats!
+end
```

## Active Record Queries
```ruby
# bad: SQL fragment for simple hash
+User.where("status = ?", status)
+
# good
+User.where(status: status)
```

```ruby
# good: named placeholder for multiple params
+Order.where(
+  'total >= :min AND currency = :currency',
+  min: params[:min_total],
+  currency: params[:currency]
+)
```

```ruby
# bad: manual join for missing relationship
+Post.left_joins(:author).where(authors: { id: nil })
+
# good (Rails 6.1+)
+Post.where.missing(:author)
```

```ruby
# bad: ordering by id
+scope :recent, -> { order(id: :desc) }
+
# good
+scope :recent, -> { order(created_at: :desc) }
```

```ruby
# good: prefer pick
+User.pick(:email)
```

```ruby
# good: squished heredoc
+Invoice.find_by_sql(<<-SQL.squish)
+  SELECT invoices.id, customers.name
+  FROM invoices
+  INNER JOIN customers ON customers.id = invoices.customer_id
+SQL
```

```ruby
# bad: redundant all
+User.all.where(id: ids)
+
# good
+User.where(id: ids)
```

```ruby
# bad: size/count misuse
+User.count
+
# good
+User.all.size
```

## Migrations
```ruby
# good: add foreign key with explicit name
+add_foreign_key :orders, :users, name: :orders_user_id_fk
```

```ruby
# good: boolean defaults
+add_column :users, :admin, :boolean, default: false, null: false
```

```ruby
# bad: change with non-reversible
+def change
+  drop_table :legacy_records
+end
+
# good
+def up
+  drop_table :legacy_records
+end
+
+def down
+  create_table :legacy_records do |t|
+    t.string :name
+  end
+end
```

## Views
```erb
<!-- bad: complex formatting in view -->
<%= number_to_currency(@order.total) if @order.total > 0 && @order.currency == 'USD' %>

<!-- good: helper -->
<%= order_total(@order) %>
```

```erb
<!-- good: partial with locals -->
<%= render 'line_item', line_item: item %>
```

## I18n
```yaml
# config/locales/models/user.en.yml
+en:
+  activerecord:
+    models:
+      user: Member
+    attributes:
+      user:
+        name: Full name
```

```erb
<!-- good: lazy lookup -->
<%= t '.title' %>
```

```ruby
# bad: scope array
+I18n.t :invalid, scope: [:errors, :messages]
+
# good
+I18n.t 'errors.messages.invalid'
```

## Mailers
```ruby
# good: mailer naming and defaults
+class NotificationMailer < ApplicationMailer
+  default from: 'Example <info@example.com>'
+
+  def digest(user)
+    @user = user
+    mail(to: user.email, subject: 'Daily Digest')
+  end
+end
```

```ruby
# good: background delivery
+NotificationMailer.digest(user).deliver_later
```

## Active Support Core Extensions
```ruby
# bad
+string = 'abc'
+string.ends_with?('c')
+
# good
+string.end_with?('c')
```

```ruby
# good
+array.exclude?(value)
```

```ruby
# good: squiggly heredoc
+message = <<~TEXT
+  Hello,
+  World
+TEXT
```

## Time and Duration
```ruby
# bad
+Time.now
+'2020-01-01'.to_time
+
# good
+Time.current
+Time.zone.parse('2020-01-01 12:00:00')
```

```ruby
# good: date ranges
+date.all_week
+date.all_month
```

```ruby
# good: duration helpers
+3.hours.from_now
+2.days.ago
```

## Bundler
```ruby
# good: group dev/test gems
+group :development, :test do
+  gem 'rspec-rails'
+  gem 'rubocop-rails'
+end
```

## Testing
```ruby
# bad
+class UsersControllerTest < ActionController::TestCase
+end
+
# good
+class UsersControllerTest < ActionDispatch::IntegrationTest
+end
```

```ruby
# bad
+travel_to(Time.current)
+
# good
+freeze_time
```
