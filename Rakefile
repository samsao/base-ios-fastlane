# rubocop:disable Style/HashSyntax

require 'rubocop/rake_task'

task default: :check

task :check => [:lint]

task :lint => [:lint_ruby]

desc 'Run RuboCop on the lib/specs directory'
RuboCop::RakeTask.new(:lint_ruby)
