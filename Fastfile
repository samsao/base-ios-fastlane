# This is the minimum version number required.
fastlane_version "2.14.2"
default_platform :ios

platform :ios do

  before_all do
    ensure_git_status_clean
    clear_derived_data

    #Bundler
    bundle_install

    #Pod install
    cocoapods
  end

  desc "Bump build number, use build_number to set a specific build number"
  private_lane :bumpBuildNumber do |options|
    if options[:build_number]
      increment_build_number(
        build_number: options[:build_number]
      )
    else
      increment_build_number(
        build_number: 0
      )
    end
    #Get version number
    get_version_number
  end

  desc "Run tests on the project"
  lane :test do
    scan(
      scheme: ENV["SCHEME"],
      destination: "OS=9.3,name=iPhone 5s"
    )
  end

  desc "Overlays version number with 'Beta' tag"
  private_lane :overlayVersion do |options|
    badge(shield: options[:version], dark: true, shield_no_resize: true)
  end

  desc "Match helper"
  private_lane :run_match do |options|
    match(type: options[:matchtype],
      readonly: true,
      force_for_new_devices: true)
  end

  desc "Create build and send to Testflight. Use changelog as an option to put the changelog"
  private_lane :betaBuild do |options|
    # Get the proper certificates
    run_match(matchtype: "appstore")

    version = formattedVersion("0.0.3")
    overlayVersion(version: version)

    gym(scheme: ENV["SCHEME"],
      configuration: ENV["CONFIGURATION"],
      clean: true,
      include_symbols: true,
    )
    pilot(changelog: options[:changelog])
  end

  desc "Create build and send to iTunes."
  private_lane :releaseBuild do
    # Get the proper certificates
    run_match(matchtype: "appstore")

    gym(scheme: ENV["SCHEME"],
      configuration: ENV["CONFIGURATION"],
      clean: true,
      include_symbols: true,
    )
    deliver(force: true)
  end

  desc "Create build and send to Crashlytics, use testers to set testers groups"
  private_lane :crashlyticsBuild do |options|
    testerEmails = options[:testers] ? options[:testers] : nil
    testerGroups = options[:testerGroups] ? options[:testerGroups] : ['samsao']

    # Get the proper certificates
    run_match(matchtype: "adhoc")

    # Build
    gym(scheme: ENV["SCHEME"],
        configuration: ENV["CONFIGURATION"],
        export_method: "ad-hoc",
        use_legacy_build_api: true,
        clean: true,
        include_symbols: true,
    )
    crashlytics(
      crashlytics_path: './Pods/Crashlytics',
      api_token: ENV["CRASHLYTICS_API_TOKEN"],
      build_secret: ENV["CRASHLYTICS_BUILD_SECRET"],
      emails: testerEmails,
      groups: testerGroups,
    )
  end

  desc "Sync certificates"
  lane :certificates do
    # implement in the sub fastlane
  end

  ################
  # Success/Error:
  ################

  after_all do |lane|
    clean_build_artifacts
  end

  error do |lane, exception|
    if ENV["SLACK_URL"] && ENV["SLACK_CHANNEL"]
       slack({
          message: "Build error: #{exception.message}",
          success: false,
          channel: ENV["SLACK_CHANNEL"]
        })
     end
  end

  def formattedVersion(version)
    "Version-#{version}-orange"
  end

end
