# redmine_acts_as_taggable_on

`redmine_acts_as_taggable_on` is a gem which allows multiple Redmine plugins to
use the tables provided by `acts-as-taggable-on` without stepping on each
others' toes.

## Why?

The problem we ran into when we discovered that both `redmine_tags` and
`redmine_knowledgebase` want to use the `acts-as-taggable-on` gem is that after
either is installed, the migration for the other fails, since the database
tables already exist.

Additionally, the plugins must choose between two less than ideal options when
uninstalling:

* Drop the tables, destroying data which may still be in use by another plugin
* Leave the tables there, violating the user's expectation that the database
  should be back to how it was before the plugin was installed

`redmine_acts_as_taggable_on` solves this issue by giving Redmine plugins a
mechanism to declare that they require these tables, and providing intelligent
migrations which only drop the tables when no other plugins are using them.

`redmine_acts_as_taggable_on` also provides a limited defence against plugins
which directly depend on `acts-as-taggable-on`. It does this by grepping
through their Gemfiles for the string `acts-as-taggable-on` -- if found, it
will treat the plugin as requiring the `acts-as-taggable-on` tables, and will
print a gentle(-ish) suggestion to use this gem instead.

## Status

**Not quite ready for prime-time.**

* No automated tests, yet (I'm working on it!)
* No known bugs
* Tested with Redmine trunk (as of 2013-05-09) only
* Not yet used by any Redmine plugins in the wild

## Limitations

This gem cannot currently protect against situations where a plugin directly
using `acts-as-taggable-on` has put the generated migration into its db/migrate
directory. If a user tries to migrate the plugin down by executing `rake
redmine:plugins:migrate VERSION=0 NAME=redmine_foo_plugin`, the tables will
still be dropped, regardless of whether other plugins are still using them.

I'm in two minds about whether to fix this: on the one hand, it would require
some nasty monkey-patching. On the other hand, data loss is no fun at all.

## Setup

Add it to your plugin's Gemfile:

    gem 'redmine_acts_as_taggable_on', '~> 0.1'

Add the migration:

    echo 'class AddTagsAndTaggings < RedmineActsAsTaggableOn::Migration; end' \
        > db/migrate/001_add_tags_and_taggings.rb

Declare that your plugin needs `redmine_acts_as_taggable_on` inside init.rb:

    require 'redmine_acts_as_taggable_on/initialize'

    Redmine::Plugin.register :my_plugin do
      requires_acts_as_taggable_on
      ...

That's it. Your plugin should now migrate up and down intelligently.
