# This is the minimum version number required.
fastlane_version '2.14.2'
default_platform :ios

platform :ios do
  ##
  # Lifecycle hooks
  #
  # Hooks for Fastlane lifecycle used (by default) on all projects unless
  # overriden.
  #

  before_all do
    skip_docs
  end

  after_all do |_lane|
    clean_build_artifacts
  end

  error do |_lane, exception|
    next unless is_in_ci? && ENV['SLACK_URL'] && ENV['SLACK_CHANNEL']

    slack(
      message: "Build error: #{exception.message}",
      success: false,
      channel: ENV['SLACK_CHANNEL'],
    )
  end

  ##
  # Public lanes
  #
  # Below are the list of lanes your are allow to use in your project's Fastfile.
  #

  desc 'Bootstrap project (sync certificates, prepare)'
  private_lane :bootstrap do
    prepare
    sync_certificates(readonly: true)
  end

  desc 'Prepare the project for building'
  lane :prepare do
    run_if_file_exists('Gemfile') { bundle_install }
    run_if_file_exists('Podfile') { cocoapods(repo_update: true) }
  end

  desc 'Clean derived data and build directory'
  lane :clean do
    clear_derived_data
  end

  desc 'Ensure repository is clean'
  private_lane :ensure_clean do
    next if ENV['FASTLANE_DEBUG']

    ensure_git_status_clean
  end

  desc <<-EOF
    Increment iOS build version (CFBundleVersion) using the passed strategy.

    Valid strategies are:
      bitrise      Use Bitrise build number (BITRISE_BUILD_NUMBER env variable), bails out loud if not set.
      testflight   Use lastest TestFlight build number and add 1 to it
  EOF
  private_lane :increment_build_version do |options|
    ensure_option_set(options, :strategy)
    ensure_option_one_of(options, :strategy, [:bitrise, :testflight])

    increment_bitrise_build_version if options[:strategy] == :bitrise
    increment_testflight_build_version if options[:strategy] == :testflight

    get_version_number
  end

  desc 'Run all lints'
  lane :lint do
    lint_ruby
  end

  desc 'Run Ruby linting rules'
  private_lane :lint_ruby do
    sh 'bundle exec rubocop -D'
  end

  desc 'Create build and send to Crashlytics.'
  private_lane :publish_crashlytics do |options|
    overlay_version(version: formatted_version(options[:version_number] || get_version_number))
    build(match_type: 'adhoc', export_method: 'ad-hoc')

    crashlytics(
      crashlytics_path: './Pods/Crashlytics',
      api_token: ENV['CRASHLYTICS_API_TOKEN'],
      build_secret: ENV['CRASHLYTICS_BUILD_SECRET'],
      emails: options[:tester_emails],
      groups: options[:tester_groups] || ['samsao'],
    )
  end

  desc 'Create build and send to Testflight.'
  private_lane :publish_testflight do |options|
    overlay_version(version: formatted_version(options[:version_number] || get_version_number))
    build(match_type: 'appstore', export_method: 'app-store')

    changelog = options[:changelog]
    if changelog.nil? && options[:changelog_file]
      changelog = ::File.read(options[:changelog_file])
    end

    pilot(
      changelog: changelog,
      distribute_external: options[:distribute_external],
      skip_waiting_for_build_processing: options[:skip_waiting_for_build_processing],
    )
  end

  desc 'Create build and send to iTunes'
  private_lane :publish_itunes do
    build(match_type: 'appstore', export_method: 'app-store')

    deliver(force: true)
  end

  desc 'Run tests on the project'
  lane :test do
    destination = ENV['TEST_DESTINATION'] || 'OS=9.3,name=iPhone 5s'
    should_notify_slack = is_in_ci? && ENV['SLACK_URL'] && ENV['SLACK_CHANNEL']

    scan(
      scheme: ENV['SCHEME'],
      destination: destination,
      slack_url: ENV['SLACK_URL'],
      slack_channel: ENV['SLACK_CHANNEL'],
      skip_slack: !should_notify_slack,
    )
  end

  desc 'Sync all required certificates from repository'
  lane :sync_certificates do
    # Implement in the sub fastlane
  end

  ##
  # Private lanes
  #
  # Below are the lanes that are helper lanes used solely inside the
  # `base-ios-fastlane` project. You should not use any of the ones
  # below in your own project.
  #
  # If you do nonetheless, please note that there is no backward
  # compatibility restrictions on those, you've been warned!
  #

  desc 'Increment app build version using Bitrise BITRISE_BUILD_NUMBER, bails if not defined'
  private_lane :increment_bitrise_build_version do
    ensure_env_set('BITRISE_BUILD_NUMBER')

    increment_build_number(build_number: ENV['BITRISE_BUILD_NUMBER'])
  end

  private_lane :increment_testflight_build_version do
    increment_build_number(build_number: latest_testflight_build_number + 1)
  end

  desc 'Build the project'
  private_lane :build do |options|
    ensure_option_set(options, :export_method)
    ensure_option_set(options, :match_type)

    # Load the proper certificates
    run_match(match_type: options[:match_type])

    # Compile the project
    gym(
      scheme: ENV['SCHEME'],
      configuration: ENV['CONFIGURATION'],
      clean: true,
      include_symbols: true,
      export_method: options[:export_method],
    )
  end

  desc 'Overlays version number with `Beta` tag'
  private_lane :overlay_version do |options|
    badge(shield: options[:version], dark: true, shield_no_resize: true)
  end

  desc 'Match helper'
  private_lane :run_match do |options|
    match(
      type: options[:match_type],
      readonly: true,
      force_for_new_devices: true,
    )
  end

  desc 'Returns true if currently running inside Bitrise CI (but not the CLI), false otherwise'
  private_lane :'is_in_ci?' do
    !ENV['BITRISE_IO'].nil?
  end

  ################
  # Success/Error:
  ################

  def ensure_env_set(key)
    raise "The '#{key}' environment variable must be set." unless ENV[key]
  end

  def ensure_option_one_of(options, key, values)
    current_value = options[key]
    return if values.include?(current_value)

    formatted_values = values.map { |value| "'#{value}'" }.join(', ')

    raise "The option '#{key}' (value '#{current_value}') must be one of: #{formatted_values}."
  end

  def ensure_option_set(options, key)
    raise "The option '#{key}' must be set." unless options.key?(key)
  end

  def formatted_version(version)
    "Version-#{version}-orange"
  end

  def run_if_file_exists(file, &block)
    # Check current directory or the one just above it (because Fastline is usually in fastlane directory)
    return unless File.exist?("./#{file}") || File.exist?("../#{file}")

    yield block
  end
end
