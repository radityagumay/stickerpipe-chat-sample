## Table of contents

- [About](#about)
- [Installation](#installation)
- [Usage](#usage)
	- [Initializing](#initializing)
	- [Showing stickers fragment](#showing-stickers-fragment)
	- [Sending stickers](#sending-stickers)
	- [Displaying stickers](#displaying-stickers)
	- [Showing pack info](#showing-pack-info)
	- [Showing new packs marker](#showing-new-packs-marker)
- [Users](#users)
- [GCM Support](#gcm-support)
	- [GCM integration module](#gcm-integration-module)
	- [Own GCM implementation](#own-gcm-implementation)
	- [Custom events](#custom-events)
	- [Notification icon](#notification-icon)
- [Customization](#customization)
	- [Colors](#colors)
	- [Stickers fragment](#stickers-fragment)
	- [Loading process](#loading-process)
	- [Pack info](#pack-info)
	- [Marked image](#marked-image)
- [Languages](#languages)
- [Credits](#credits)
- [Contact](#contact)
- [License](#license)

## About

**StickerPipe** is a stickers SDK for Android platform.
This sample demonstrates how to add stickers to your chat.

![android](static/sample.gif)

## Installation

If you use Eclipse IDE - follow [this instructions](https://github.com/908Inc/stickerpipe-android-sdk-for-eclipse).

Add stickers repository in build.gradle:
```android
repositories {
   maven { url  'http://maven.stickerpipe.com/artifactory/stickerfactory' }
}
```
Add library dependency in build.gradle:
```android
compile('vc908.stickers:stickerfactory:0.6.4@aar') {
     transitive = true;
}
```
List of available versions you can find [here](http://maven.stickerpipe.com/artifactory/stickerfactory/vc908/stickers/stickerfactory/)

Add content provider with your application package to manifest file:
```android
<provider
     android:name="vc908.stickerfactory.provider.StickersProvider"
     android:authorities="<YOUR PACKAGE>.stickersProvider"
     android:exported="false"/>
```

## Usage

### Initializing

Initialize library at your Application onCreate() method
```android
StickersManager.initialize(“72921666b5ff8651f374747bfefaf7b2", this);
```
You can get your own API Key on http://stickerpipe.com to have customized packs set.

### Showing stickers fragment

Create stickers fragment
```android
stickersFragment = new StickersFragment.Builder().build();
```

Then you only need to show fragment. See exaple with best practice.

### Sending stickers

To send stickers you need to set listener and handle results
```android
// create listener
private OnStickerSelectedListener stickerSelectedListener = new OnStickerSelectedListener() {
    @Override
    public void onStickerSelected(String code) {
        if (StickersManager.isSticker(code)) {
	        // send message
        } else {
            // append emoji to your edittext
        }
    }
};
// set listener to your stickers fragment
stickersFragment.setOnStickerSelectedListener(stickerSelectedListener)
```

Listener can take an emoji, so you need to check code first, and then send sticker code or append emoji to your edittext.

### Displaying stickers

```android
// Show sticker in adapter
if (StickersManager.isSticker(message)){ // check your chat message
StickersManager.with(context) // your context - activity, fragment, etc
        .loadSticker(message)
        .into((imageView)); // your image view
} else {
	// show a message as it is
}
```
As an additional feature, you can check, is sticker exists at user's library, and show some marks
```android
if (StickersManager.isPackAtUserLibrary(stickerCode)) {
	// show mark
} else {
	// hide mark
}
```
![listmark](static/listmark.png)  

Then you can set click listener and show pack info, where user can download pack.

### Showing pack info

You can show pack info using builder.
```android
 new PackInfoActivity.Builder(MainActivity.this)
                            .show(stickerCode);
```

<img src="static/pack.png" width="300">

### Showing new packs marker

You can use BadgedStickersButton to indicate to user, that he has a new pack
```android
            <vc908.stickerfactory.ui.view.BadgedStickersButton
                android:id="@+id/stickers_btn"
                android:layout_width="@dimen/material_48"
                android:layout_height="@dimen/material_48"
                android:layout_centerVertical="true"
                app:centerColor="@android:color/black"
                app:middleColor="@android:color/white"
                app:outColor="@android:color/white"
                android:background="?android:attr/selectableItemBackground"/>
```
![markers](static/marker.png)  

Use this view as ImageButton, other work will be doing by SDK.

## Users

When you know your user id, set it to sdk   

```Android
StickersManager.setUserID("some user id");
```
This add ability to users to manage their packs and don't lose them after reinstalling

## GCM Support

Starting from version 0.6.4 you have an ability to add push notifications to sdk. This is necessary, when you change content(add or remove packs) or want to promote some pack. There are two ways - use GcmIntegration module or use own notification system

### GCM integration module

If your application don't have gcm functionality you need to follow next steps   

- Add dependency at your build file. List of available versions you can find [here](http://maven.stickerpipe.com/artifactory/stickerfactory/vc908/stickers/stickerfactory/)
```android
    compile('vc908.stickers:gcmintegration:0.1.4@aar'){
        transitive = true;
    }
```   

- Add reciver at your Manifest file and replace yor_package_name with your real package name
```android
 <receiver
            android:name="com.google.android.gms.gcm.GcmReceiver"
            android:exported="true"
            android:permission="com.google.android.c2dm.permission.SEND" >
            <intent-filter>
                <action android:name="com.google.android.c2dm.intent.REGISTRATION" />
                <action android:name="com.google.android.c2dm.intent.RECEIVE" />
                <category android:name="yor_package_name" />
            </intent-filter>
        </receiver>
```
- Retrieve Android GCM API key and store them   
Follow [this link](https://developers.google.com/mobile/add) and setup your application to obtain GCM API key. Next folow to your [admin panel](http://stickerpipe.com/cp), press "Manage" and store key.

![store gcm key](static/storeKey.png)

- From previous step, receive sender id and set it to GcmManager at Application class
```android
GcmManager.setGcmSenderId(mContext, "86472317986");
```

### Own GCM implementation

If you have own GCM implementation, follow next steps   

- Go to your [admin panel](stickerpipe.com/cp), press "Manage"  and store your GCM API key
- Set sender id to GcmManager at Application class   
```android
GcmManager.setGcmSenderId(mContext, "86472317986");
```
- When you receive GCM token, set it to SDK
```android
StickersManager.sendGcmToken("GCM TOKEN");
```
- When you receive notification, check it for stickers data and process it with SDK if need
```android
if(EventsManager.isStickersData(bundleData)){
            EventsManager.processPush(mContext, bundleData);
        } else {
            // you own processing
        }
```

### Custom events

You can call events without gcm notifications
- When you need notify sdk that content need to be updated   
```android
EventsManager.updateContent();
```
- When you need to show notification, which opens pack info   
```android
EventsManager.showPackNotification(mContext, "Notification title", "Notification text", "packName");
```

### Notification icon

Pack notifications use default icon. You can set your own icon using next code   
```xml
        <meta-data
            android:name="vc908.stickerfactory.notification_icon"
            android:resource="@drawable/ic_your_icon"/>
```

## Customization

### Colors

You can customize all colors by overriding values with "sp_" prefix. This is next available values
```xml
    <color name="sp_primary">#5E7A87</color>
    <color name="sp_primary_dark">#455A64</color>
    <color name="sp_primary_light">@android:color/white</color>
    <color name="sp_placeholder_color_filer">@android:color/white</color>
    <color name="sp_pack_info_bg">@android:color/white</color>

    <color name="sp_stickers_tab_bg">@color/sp_primary</color>
    <color name="sp_stickers_tab_strip">#fff</color>
    <color name="sp_stickers_list_bg">@android:color/white</color>
    <color name="sp_stickers_tab_icons_filter">@android:color/white</color>
    <color name="sp_stickers_emoji">@android:color/black</color>
    <color name="sp_remove_icon">#616161</color>
    <color name="sp_reorder_icon">#9e9e9e</color>
    <color name="sp_primary_text">@android:color/black</color>
    <color name="sp_secondary_text">#616161</color>
    <color name="sp_list_item_pressed">#ffe1e1e1</color>
    <color name="sp_list_item_normal">#ffffffff</color>
```

### Stickers fragment

You can customize stickers fragment with builder:

- Set sticker selected listener
```android
setOnStickerSelectedListener(OnStickerSelectedListener listener)
```
It’s highly recommended to set listener outside of builder, directly to fragment when creating or restoring activity.

- Set custom placeholder for stickers in list
```android
setStickerPlaceholderDrawableRes(@DrawableRes int stickerPlaceholderDrawableRes)
```

- Set color filter for sticker’s placeholder
```android
setStickerPlaceholderColorFilterRes(@ColorRes int stickerPlaceholderColorFilterRes)
```
- Set background drawable resource for stickers list with Shader.TileMode.REPEAT
```android
setStickersListBackgroundDrawableRes(@DrawableRes int stickersListBgRes)
```

- Set stickers list background color
```android
setStickersListBackgroundColorRes(@ColorRes int stickersListBackgroundColorRes)
```
- Set custom placeholder for tab icons
```android
setTabPlaceholderDrawableRes(@DrawableRes int tabPlaceholderDrawableRes)
```
- Set tab placeholder filter color
```android
setTabPlaceholderFilterColorRes(@ColorRes int tabPlaceholderColorFilterRes)
```
- Set tab panel background color
```android
setTabBackgroundColorRes(@ColorRes int tabBackgroundColorRes)
```
- Set selected tab underline color
```android
setTabUnderlineColorRes(@ColorRes int tabUnderlineColorRes)
```
- Set color filter for default tab icons placeholder
```android
setTabIconsFilterColorRes(@ColorRes int tabIconsFilterColorRes)
```
- Set max width for sticker cell
```android
setMaxStickerWidth(int maxStickerWidth)
```
- Set color filter for backspace button at emoji tab
```android
setBackspaceFilterColorRes(@ColorRes int backspaceFilterColorRes)
```
- Set text for empty view at recent tab
```android
setEmptyRecentTextRes(@StringRes int emptyRecentTextRes)
```
- Set text color for empty view at recent tab
```android
setEmptyRecentTextColorRes(@ColorRes int emptyRecentTextColorRes)
```
- Set image for empty view at recent tab
```android
setEmptyRecentImageRes(@DrawableRes int emptyRecentImageRes)
```
- Set image color filter for empty view at recent tab
```android
setEmptyRecentColorFilterRes(@ColorRes int emptyRecentColorFilterRes)
```

### Loading process

You can customize sticker loading process with next setters:

- Set sticker placeholder drawable
```android
setPlaceholderDrawableRes(@DrawableRes int placeholderRes)
```
- Set color filter for default placeholder
```android
setPlaceholderColorFilterRes(@ColorRes int colorFilterRes)
```

### Pack info

You can customize pack info screen with next setters:

- Set primary color color, which used for toolbar, action link and progress under action link.
```android
setPrimaryLightColorRes(@ColorRes int colorRes)
```
- Set light color, which used for progress bar under action link
```android
setPrimaryLightColorRes(@ColorRes int colorRes)
```
- Set background color
```android
setBackgroundColor(@ColorRes int colorRes)
```
- Set placeholder drawable for stickers
```android
setPlaceholderDrawable(@DrawableRes int drawableRes)
```        
- Set placeholder color filter for stickers
```android
setPlaceholderColor(@ColorRes int colorRes)
```

### Marked image

You can customize BadgedStickersButton badge with attributes
```android
<vc908.stickerfactory.ui.view.BadgedStickersButton
	...
	app:centerColor="@android:color/white"
	app:middleColor="@android:color/black"
    app:outColor="@android:color/white"
    ... />
```
or setter
```android
setBadgeColors(@ColorRes int center, @ColorRes int middle, @ColorRes int outer) {
```

## Languages

Stickerpipe SDK support English language. If your application use another languages, you need add translation for next values
```xml
		<string name="sp_package_stored">Pack stored</string>
		<string name="sp_package_removed">Pack removed</string>
		<string name="sp_collections">Collections</string>
		<string name="sp_recently_empty">We have wonderful stickers!\nSwipe left and start using</string>
		<string name="sp_pack_info">Pack info</string>
		<string name="sp_pack_download">Download</string>
		<string name="sp_pack_remove">Remove pack</string>
		<string name="sp_no_internet_connection">No internet connection</string>
		<string name="sp_cant_process_request">Can not process request</string>
		<string name="sp_open_stickers">Open stickers</string>
```

##Credits


908 Inc.

## Contact


mail@908.vc

## License


StickerPipe is available under the Apache 2 license. See the [LICENSE](LICENSE) file for more information.
