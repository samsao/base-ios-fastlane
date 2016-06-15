#update_fastlane

# This is the minimum version number required.
fastlane_version "1.94.1"
default_platform :ios

platform :ios do

  before_all do
    ensure_git_status_clean
    clear_derived_data
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
      increment_build_number # automatically increment by one
    end
    #Get version number
    get_version_number
  end

  desc "Create build and send to testflight"
  private_lane :betaBuild do
    # Get the proper certificates
    match(type: "appstore", 
      readonly: true,
      force_for_new_devices: true)
    gym(scheme: ENV["SCHEME"], 
      configuration: ENV["CONFIGURATION"],
      clean: true,
      include_symbols: true,
    )
    deliver(force: true, skip_metadata: true)
  end

  private_lane :crashlyticsBuild do |options|
    bumpBuildNumber
    testers = "internal-samsao-ios"
    #TODO: Add the external testers here
    if options[:testers]
      testers += ""
    end
    # Get the proper certificates
    match(type: "adhoc", 
      readonly: true,
      force_for_new_devices: true)
    # Build
    gym(scheme: ENV["SCHEME"],
        configuration: ENV["CONFIGURATION"],
        export_method: "ad-hoc",
        clean: true,
        include_symbols: true,
    )
    crashlytics(
      crashlytics_path: './Pods/Crashlytics',
      api_token: ENV["CRASHLYTICS_API_TOKEN"],
      build_secret: ENV["CRASHLYTICS_BUILD_SECRET"],
      groups: options[:testers],
    )
  end


  private_lane :slack do |options|
    if ENV["SLACK_URL"] && ENV["SLACK_CHANNEL"]
      slack({
        message: options["message"],
        success: true,
        channel: ENV["SLACK_CHANNEL"]
      })
    end
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

end