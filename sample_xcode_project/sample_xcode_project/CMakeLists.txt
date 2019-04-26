set(
    resources
    ${CMAKE_CURRENT_LIST_DIR}/Base.lproj/LaunchScreen.storyboard
    ${CMAKE_CURRENT_LIST_DIR}/Base.lproj/Main.storyboard
)

add_executable(
    ShakaPlayer
    AppDelegate.h
    AppDelegate.m
    AssetSelectionTableViewController.h
    AssetSelectionTableViewController.m
    ConfigTableValue.h
    ConfigTableValue.m
    ConfigTableViewController.h
    ConfigTableViewController.m
    ErrorDisplayView.h
    ErrorDisplayView.m
    LicenseTableViewController.h
    LicenseTableViewController.m
    LicenseViewController.h
    LicenseViewController.m
    NSObject+ShakaLayoutHelpers.h
    NSObject+ShakaLayoutHelpers.m
    PlayerViewController.h
    PlayerViewController.m
    ProgressAndBufferedSlider.h
    ProgressAndBufferedSlider.m
    main.m
    ${resources}
)

target_link_libraries(
    ShakaPlayer
    PRIVATE
    shaka_player_embedded::shaka_player_embedded
)

set_target_properties(
    ShakaPlayer
    PROPERTIES
    MACOSX_BUNDLE YES
    MACOSX_BUNDLE_INFO_PLIST "${CMAKE_CURRENT_LIST_DIR}/Info.plist"
    XCODE_ATTRIBUTE_TARGETED_DEVICE_FAMILY "1,2" # Universal (iPad + iPhone)
    XCODE_ATTRIBUTE_CLANG_ENABLE_OBJC_ARC YES
    RESOURCE "${resources}"
    XCODE_ATTRIBUTE_PRODUCT_NAME "ShakaPlayer"
    XCODE_ATTRIBUTE_BUNDLE_IDENTIFIER "com.github.google.shakaplayerembedded"
    XCODE_ATTRIBUTE_PRODUCT_BUNDLE_IDENTIFIER  "com.github.google.shakaplayerembedded"
)
