# This is the minimum version number required.
fastlane_version '2.14.2'
default_platform :ios

platform :ios do
  before_all do
    skip_docs

    prepare
  end

  desc 'Prepare the project for building'
  lane :prepare do
    run_if_file_exists('Gemfile') { bundle_install }
    run_if_file_exists('Podfile') { cocoapods }
  end

  desc 'Clean derived data and build directory'
  lane :clean do
    clear_derived_data
  end

  desc 'Bump build number, use build_number to set a specific build number'
  private_lane :bump_build_number do |options|
    increment_build_number(
      build_number: latest_testflight_build_number + 1 || options[:build_number],
    )

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

  desc 'Overlays version number with `Beta` tag'
  private_lane :overlay_version do |options|
    badge(shield: options[:version], dark: true, shield_no_resize: true)
  end

  desc 'Match helper'
  private_lane :run_match do |options|
    match(
      type: options[:matchtype],
      readonly: true,
      force_for_new_devices: true,
    )
  end

  desc 'Create build and send to Testflight. Use changelog as an option to put the changelog'
  private_lane :beta_build do |options|
    clean

    # Get the proper certificates
    run_match(matchtype: 'appstore')

    version = formatted_version('0.0.3')
    overlay_version(version: version)

    gym(
      scheme: ENV['SCHEME'],
      configuration: ENV['CONFIGURATION'],
      clean: true,
      include_symbols: true,
      export_method: 'app-store',
    )

    pilot(changelog: options[:changelog])
  end

  desc 'Create build and send to iTunes'
  private_lane :release_build do
    clean

    # Get the proper certificates
    run_match(matchtype: 'appstore')

    gym(
      scheme: ENV['SCHEME'],
      configuration: ENV['CONFIGURATION'],
      clean: true,
      include_symbols: true,
      export_method: 'app-store',
    )

    deliver(force: true)
  end

  desc 'Create build and send to Crashlytics, use testers to set testers groups'
  private_lane :crashlytics_build do |options|
    tester_emails = options[:tester_emails]
    tester_groups = options[:tester_groups] || ['samsao']

    clean

    # Get the proper certificates
    run_match(matchtype: 'adhoc')

    # Build
    gym(
      scheme: ENV['SCHEME'],
      configuration: ENV['CONFIGURATION'],
      clean: true,
      include_symbols: true,
      export_method: 'ad-hoc',
    )

    crashlytics(
      crashlytics_path: './Pods/Crashlytics',
      api_token: ENV['CRASHLYTICS_API_TOKEN'],
      build_secret: ENV['CRASHLYTICS_BUILD_SECRET'],
      emails: tester_emails,
      groups: tester_groups,
    )
  end

  desc 'Returns true if currently running inside Bitrise CI (but not the CLI), false otherwise'
  private_lane :'is_in_ci?' do
    !ENV['BITRISE_BUILD_NUMBER'].nil?
  end

  desc 'Sync certificates'
  lane :certificates do
    # Implement in the sub fastlane
  end

  ################
  # Success/Error:
  ################

  after_all do |_lane|
    clean_build_artifacts
  end

  error do |_lane, exception|
    return unless is_in_ci? && ENV['SLACK_URL'] && ENV['SLACK_CHANNEL']

    slack(
      message: "Build error: #{exception.message}",
      success: false,
      channel: ENV['SLACK_CHANNEL'],
    )
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
