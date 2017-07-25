## 0.4.0 (July 25, 2017)

* Added the ability to pass `testflight_groups` parameter to the `publish_testflight` lane.

## 0.3.0 (July 4, 2017)

 * The `prepare` lane now performs a `pod repo update` before doing the
   `pod install` lane to avoid problem of an out of sync pods repository.

 * Added a `bootstrap` lane used to setup the project which does, `prepare` and
   `sync_certificates`.

 * Added possibility in `publish_crashlytics` and `publish_testflight` to pass
   a custom version value (via `:version_number`).

 * Fixed `overlay_version` to correctly retrieve current version number.

 * Split lanes in two sets, a public set and a private one. This is artificial
   but you should never rely on the private set in your project's Fastfile.

 * Changed `increment_build_number` accepted options. Only valid option now is
   `:strategy` and can be one of `:bitrise` or `:testflight`.

 * Renamed `increment_build_number` to `increment_build_version`.

 * Renamed build tasks:
    * Lane `crashlytics_build` renamed to `publish_crashlytics`.
    * Lane `beta_build` renamed to `publish_testflight`.
    * Lane `release_build` renamed to `publish_itunes`.

## 0.2.0 (April 6, 2017)

 * Removed the need to have a clean environment for non-build tasks.
