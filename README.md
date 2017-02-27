# how to use rails structure
# How to deal with fat models
# Hash key and value naming
# Config section
# how to deal with env variables
# naming custom validation methods and how to deal with them
# naming variables and methods
# Gemfile and locales
# Null objects
# Making use of the rails CLI
# When to use constants
# FRIDA: mention obvious examples of vulnerability

Boundless Drop Ruby/Rails Style Guide:
======================================
This is Boundless Drop's Ruby Style Guide.

This was inspired by [bbatsov's](https://github.com/bbatsov/rails-style-guide) and [Airbnb's](https://github.com/airbnb/ruby) guides.

This is always going to be a work in progress and it is going to adapt to whatever is better suiting our team in terms of what will achieve better code quality and cleaner code.

Clarity, Readability & Convention:
----------------------------------
- Always use a Linter such as [rubocop](https://github.com/bbatsov/rubocop).
- Use the Syntastic plugin for vim to work with Rubocop.
- Method SHOULD NOT be long than 8-9 lines. If they do, then you are doing something wrong.
- Classes should not be longer than 100-120 lines. If they do, then you are doing something wrong.
- Use comments only if absolutely necessary and indent them with one space after the pound sign.
# integer keys
- Hashes should always use symbols rather than strings for their keys
- For active record where querying for an attribute in array pass it a hash instead of writing a string where query
- Prefer single-quoted strings when you don't need string interpolation or special symbols.
- Defining  and calling methods should use parenthesis
  ```ruby
  def hello(name)
    puts "#{name}"
  end

  hello('Sabri')
  ```
- Always use &&, ||, ! instead of language words like and, or, not
- Use CamelCase for classes and modules, snake_case for variables and methods, SCREAMING_SNAKE_CASE for constants.
- Break down long chain method calls for readability
- Methods that return boolean should end with a “?”
- Methods that perform some permanent or potentially dangerous change should ending in "!"
- Use Hash params instead of splat operator (*args)
  ```ruby
  def hello(options = {})
    first_name  = options[:first_name]
    middle_name = options[:middle_name]
    last_name   = options[:last_name]

    puts "Hello #{first_name} #{middle_name} #{last_name}"
  end

  hello(first_name: 'Sabri', middle_name: 'Saaber', last_name: 'Sabri')
  ```
- Use `each`, not `for`, for iteration.
- Prefer `private` over `protected` for non-public `attr_reader`s, `attr_writer`s, and `attr_accessor`s.

Database level
--------------
- Never use #{variable} in any SQL queries for obvious security vulnerability
- While using Active Record methods such as `where` do not use `?`. Instead, use named arguements as follows:
  ```ruby
    # bad
    branch.taxes.where('start_date <= ? AND (end_date > ? OR end_date IS NULL)', date, date)

    # good
    branch.taxes.where('start_date <= :date AND (end_date > :date OR end_date IS NULL)', { date: date })
  ```

Models:
---------------------------
- Models are for business logic and data-persistence ONLY!
- Name models after nouns and not verbs
- Meaningful but short names
- Feel free to introduce non ActiveRecord models. If validations are needed, use [ActiveAttr](https://github.com/cgriego/active_attr) gem.

The following model structure is to be used in all our models:
# scopes

  ```ruby
  class User < ActiveRecord::Base
    include Something::Something

    # Constants
    ERRORS = { warning: 'Warning', danger: 'Danger' }.freeze

    # Associations
    has_one :life
    has_many :friends
    has_many :families
    belongs_to :mentor
    belongs_to :team

    # ActiveRecord Validations
    validates :name, presence: true
    validates :mentor_id, presence: true, if: :has_contact_with_mentor?
    validates :username, uniqueness: true, presence: true

    # Custom Validations
    validate :username_is_registered

    # Rails Delegations
    delegate :email, to: mentor, prefix: true, allow_nil: true
    delegate :first_name, to: mentor, prefix: true, allow_nil: true
    delegate :name, to: mentor, prefix: true, allow_nil: true

    # attr methods
    attr_accessor :something
    attr_writer :something_else
    attr_reader :something_different

    # ActiveRecord callbacks
    before_save :clean_username
    after_save :send_welcome_pack

    # class methods
    def self.i_am_a_class_method
      puts 'Hello, I am a class method!'
    end

    # Instance methods
    def i_am_an_instance_method
      puts 'Hello, I am an instance method!'
    end

    protected

    def i_am_a_protected_instance_method
      puts 'Hello, I am a protected instance method!'
    end

    private

    def i_am_a_private_instance_method
      puts 'Hello, I am a private instance method!'
    end

    # Custom validation methods
    def username_is_registered; end
  end
  ```

Routing
-------
- Avoid the `:except` option in routes.
- Use the `:only` option to explicitly state exposed routes.
- Order resourceful routes alphabetically by name.
- Use nested routes to explain relation between models
- If you need to nest routes more than 1 level deep then use the shallow: true option. This will save user from long urls posts/1/comments/5/versions/7/edit and you from long url helpers edit_post_comment_version.

  ```ruby
  resources :posts, shallow: true do
    resources :comments do
      resources :versions
    end
  end
  ```

I18n
----
- All strings that will be used in views MUST be written as a translation key using I18n's translate method.
- All translation keys should be in snake_case.
- We use I18n translate as follows:
  ```ruby
  # In Models
  I18n.t('translation_key')

  # Everywhere else
  t('translation_key')
  ```
- All translation keys should be added to both en.yml and ar.yml which are located in config/locales/ .
- Custom validation errors should be written as follows:
  ```ruby
  I18n.t('model_name_errors.translation_key')
  ```
- Translations files should be structured as follows:
  ```yaml
  # Indentations are very important so be careful
  lang:
      # Normal Translations
      he_is_very_sabri: He is very Sabri
      i_like_trains: I like trains

      # Errors Translations
      model_name_errors:
          cant_be_this: "Can't be this"

      # Lists Translations
      people_categories:
          bad: Bad
          good: Good

      # Responses Translations
      some_responses:
          has_been_saved: Has been saved!

      # ActiveRecord Translations
      activerecord:

        # Models Translations
        models:
            another_model: Another Model
            model_name: Model Name

        # Attributes Translations
        attributes:
            model_name:
                another_one: Another One
                attribute: Attribute
  ```
- Translations inside each group should be sorted alphabetically.
- Gems translations should be in their own separate files in the locales directory and should be named as follows: gemname.lang.yml .
- Directory structure example:

  ```
  .
  ├── ar.yml
  ├── en.yml
  ├── paperclip.ar.yml
  └── paperclip.en.yml
  ```
