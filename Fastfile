#update_fastlane

# This is the minimum version number required.
fastlane_version "1.95.0"
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
    scan(scheme: ENV["SCHEME"])
  end

  desc "Create build and send to Testflight"
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

  desc "Create build and send to Crashlytics, use testers to set testers groups"
  private_lane :crashlyticsBuild do |options|
    testers = ""
    #TODO: Add the external testers here
    if options[:testers]
      testers = options[:testers]
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
      groups: testers,
    )
  end

  desc "Send a message to a slack channel, SLACK_URL and SLACK_CHANNEL have to be set before"
  private_lane :slack do |options|
    if ENV["SLACK_URL"] && ENV["SLACK_CHANNEL"]
      if options[:message]
        slack({
        message: "foo",
        success: true,
        channel: ENV["SLACK_CHANNEL"]
      })
      else
        slack({
        success: true,
        channel: ENV["SLACK_CHANNEL"]
      })
      end
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