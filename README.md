# Valuepotion SDK for Android - Getting Started

## Before You Begin

### 1. Register Your App
Visit [ValuePotion](https://valuepotion.com) website and register the information of your app. After that, you will be given a **Client ID** and a **Secret Key**.

### 2. Import the SDK into your Android project

Unzip the sdk downloaded, and add `valuepotion.jar` to `libs` folder.

### 3. Add Dependencies
In order to use Valuepotion, these two dependencies are required to be included in your project.

1. Google Play Services
 * Follow the [link](https://developer.android.com/google/play-services/setup.html) and put "Google Play Services" to your project.
2. Android Support Library
 * Follow the [link](http://developer.android.com/tools/support-library/setup.html) and refer to "Adding libraries without resources" section since Valuepotion does not need resources from Android Support Library.

## Initialize SDK
The following code is to initialize SDK.

### init
Initialize SDK at `onCreate` method at the first activity that will be launched.

```java
import com.valuepotion.sdk.ValuePotion;
public class MyActivity extends Activity {
  @Override
  public void onCreate() {
    super.onCreate();
    // Initialize SDK using Client ID and Secret Key that you got from ValuePotion website.
    ValuePotion vp = ValuePotion.init(this, "CLIENT_ID", "SECRET_KEY");
  }
}
```

### onStart / onStop

Add the following code into each activity.

```java
  @Override
  protected void onStart() {
    super.onStart();
    ValuePotion.getInstance().onStart(this);
  }
    
  @Override
  protected void onStop() {
    super.onStop();
    ValuePotion.getInstance().onStop(this);
  }
```

If you've done all right so far, you should be able to see statistics of session, install and update events on ValuePotion dashboard.

## Configure AndroidManifest.xml

### Add permissions

```xml
<!-- Valuepotion Plugin Permissions -->
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
<uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
<uses-permission android:name="android.permission.READ_PHONE_STATE" />
<!-- Valuepotion Plugin Permissions end -->
```

### Add the components of Valuepotion

```xml
<!-- Valuepotion Components -->
	<!-- for interstital ad interface -->
	<activity
			android:name="com.valuepotion.sdk.VPInterstitialActivity"
			android:theme="@android:style/Theme.Translucent" >
	</activity>

	<!-- for CPI tracking -->
	<receiver
			android:name="com.valuepotion.sdk.VPInstallReceiver"
			android:exported="true" >
			<intent-filter>
					<action android:name="com.android.vending.INSTALL_REFERRER" />
			</intent-filter>
	</receiver>
<!-- Valuepotion Components End -->
```



## Integrate with Interstitial Ads

### 1. Display Interstitial Ads
If you've created a campaign at [ValuePotion](https://valuepotion.com), you can display it as an interstitial ad at your own app. Before displaying interstitial ads, you should set up a placement. Otherwise, "default" placement will be used by default.

**placement** is a name to distinguish many points where you want to display ads. There's no restriction but it just should be a string.

```java
// Display ads at "default" placement.
ValuePotion.getInstance().openInterstitial();

// Display ads at "main_menu" placement.
ValuePotion.getInstance().openInterstitial("main_menu");
```

### 2. Cache Interstitial Ads
Using `openInterstitial()` method, the SDK will download data for ads via HTTP and display on screen. So it takes some time. If you cache ads when your game launches, you can display the ads at any time with no delay.


```java
// If you cache an ad for "after_login" placement once,
ValuePotion.getInstance().cacheInterstitial("after_login");

...

// Later on, you can display the ad with no delay.
ValuePotion.getInstance().openInterstitial("after_login");
```

### 3. Display Interstitial Ads Only When Caches are Available
You can display interstitial ads only when caches are available.

```java
// Check if the cache for "item_shop" placement exists.
if (ValuePotion.getInstance().hasCachedInterstitial("item_shop") {
  // then, display the ad for "item_shop" placement.
  ValuePotion.getInstance().openInterstitial("item_shop");
}
```


## Event Tracking
You can analyze your game with event tracking. And based on events you can create cohort to use for marketing. There are non-payment event and payment event.

### 1. Non-Payment Event
Non-payment event is not related to In-App Purchase. You can use non-payment event to analyze user behavior. To use non-payment event, you should send a name of action and its value. The following code is an example to send non-payment event.

```java
// User has been acquired 3 items.
ValuePotion.getInstance().trackEvent("get_item_ruby", 3);
ValuePotion.getInstance().trackEvent("get_item_ruby", 3f);
ValuePotion.getInstance().trackEvent("get_item_ruby", 3.0);
```

If there's no specific value needed, you can set "action" only.

```java
// User has visited "item shop" menu.
String action = "enter_item_shop";
ValuePotion.getInstance().trackEvent(action);
```

If you want to build a hierarchy of events, you can specify that like following:

```java
String category = "item";
String action = "get_ruby";
String label = "reward_for_login";
int value = 30;
ValuePotion.getInstance().trackEvent(category, action, label, value);
```

### 2. Payment Event
Payment event is tracked when In-App Purchase(In-App Billing) has occurred. If you track payment events, you can check daily statistics of Revenue, ARPU, ARPPU, PPU, etc.
The following code is an example to send payment event occurred in your game.

#### 2-1. Track Event

```java
// User purchased $0.99 coin item.
String eventName = "purchase_coin";
double amount = 0.99f;
String currency = "USD";
String orderId = The identifier of receipt after completing purchase. ex> "1000000126295148";
String productId = The identifier of item. ex> "com.valuepotion.tester.item_diamond_1";

ValuePotion.getInstance().trackPurchaseEvent(eventName, amount, currency, orderId, productId);
```

ValuePotion provides campaign of In-App Purchase (IAP) type. When a user makes revenue via an ad of IAP type, if you add extra info to payment event, you can get revenue report per campaign in detail. The following code is how to send payment event which occurred from IAP ad.

*Too see more information aboue callback method `onRequestedPurchase`, please see `void onRequestedPurchase(ValuePotion vp, String placement, VPPurchase purchase);` item under "Advanced: Listener" section.*

```java

ValuePotionListener listener = new ValuePotionListener(){
  @Override
  void onRequestedPurchase(ValuePotion vp, String placement, VPPurchase purchase)
  {
    // Proceed the requested payment

    ...

    // User purchased some Diamond item for KRW 1,200 via IAP campaign. So you're attaching campaignId and contentId from purchase object as payment event parameters.
    String eventName = "purchase_coin";
    double amount = 0.99f;
    String currency = "USD";
    String orderId = The identifier of receipt after completing purchase. ex> "1000000126295148";
    String productId = The identifier of item. ex> "com.valuepotion.tester.item_diamond_1";
    String campaignId = purchase.getCampaignId();
    String contentId = purchase.getContentId();

    ValuePotion.getInstance().trackPurchaseEvent(eventName, amount, currency, orderId, productId, campaignId, contentId);
  }
  
  ...
  
};
```

#### Reference
* For accurate analysis, please specify real purchase amount and currency.
* We follow [ISO 4217](http://en.wikipedia.org/wiki/ISO_4217) for currency.

### 3. Test If Event Tracking Works
You can test if event tracking works by using test mode of the SDK. The following code will activate test mode.

```java
// Activate test mode. Default is false
ValuePotion.getInstance().setTest(true);
```

If you send events from an app built with test mode, you should see the events on developer's console at [ValuePotion](https://valuepotion.com) at real time.

**Warning** : Before submitting your app to app store, please disable test mode. Events sent from test mode are only displayed on Developer's console but excluded from analysis.



## Integrate User Information
You can collect user information as well as events. Possible fields of user information are user id, server id which user belongs to, birthdate, gender, level and number of friends and type of user account. All of them are optional so you can choose which fields to collect.

You can use this information for marketing by creating user cohort. You can update your information when it changes to integrate with ValuePotion.

```java
ValuePotion.setUserId("support@valuepotion.com");
ValuePotion.setUserServerId("server1");
ValuePotion.setUserBirth("19830328");
ValuePotion.setUserGender("M");
ValuePotion.setUserLevel(32);
ValuePotion.setUserFriends(219);
ValuePotion.setUserAccountType("guest");
```

The following is the detail on each field.

Field         | Description
------------- | ------------
**userId**    | User account id used in game
**serverId**  | If you need to distinguish users by server which they belong to, you should set serverId.<br>Then you can get statistics based on serverId.
**birth**     | Date of birth in YYYYMMDD. <br>If you know only year of birth, fill last four digits with "0" like "19840000".<br>If you know only date of birth(but not year), fill first four digits with "0" like "00001109".
**gender**    | "M" for male, "F" for female.
**level**     | Level of user in game.
**friends**   | Number of user's friends.
**accountType**   | Type of user's account type. (facebook, google, guest, etc)

## Integrate Push Notification
If you integrate with Push Notification API, you can easily create campaigns of Push type and send message to users. So you can wake up users who haven't played game for long time, or you can also notify users new events in game, etc.

### 1. Register Certificate
Visit [ValuePotion](https://valuepotion.com) website and update your app information. Please fill in GCM ApiKey at **App Edit** page. 'GCM ApiKey' is 'Server Key' you created from [Google Developers Console](https://console.developers.google.com/project).

### 2. Configure AndroidManifest.xml

* Append the following permissions.

```xml
<manifest ...>

    <uses-permission android:name="android.permission.GET_ACCOUNTS" />
    <uses-permission android:name="android.permission.WAKE_LOCK" />
    <uses-permission android:name="com.google.android.c2dm.permission.RECEIVE" />

    <!--
        Replace 'PACKAGE_NAME' with your app's package name.
        ex)
        if 'package' of <application> tag is 'com.valuepotion.testapp',
        put 'com.valuepotion.testapp.permission.C2D_MESSAGE'.
    -->
    <permission android:name="PACKAGE_NAME.permission.C2D_MESSAGE" android:protectionLevel="signature" />
    <uses-permission android:name="PACKAGE_NAME.permission.C2D_MESSAGE" />

    <application ...
```

* Register the following receiver, service and activity.

```xml
    <application ...>
        ...

        <!--
           Replace 'PACKAGE_NAME' with your app's package name.
        -->

        <receiver
            android:name="com.valuepotion.sdk.push.GcmBroadcastReceiver"
            android:permission="com.google.android.c2dm.permission.SEND">
            <intent-filter>
                <action android:name="com.google.android.c2dm.intent.RECEIVE" />
                <action android:name="com.google.android.c2dm.intent.REGISTRATION" />
                <action android:name="com.valuepotion.sdk.push.NOTIFICATION_OPENED" />

                <category android:name="PACKAGE_NAME" />
            </intent-filter>
        </receiver>
        <service android:name="com.valuepotion.sdk.push.GcmIntentService" />

        <!-- GCM push-notification pop-up style -->
        <activity
            android:name="com.valuepotion.sdk.VPPopupActivity"
            android:launchMode="singleInstance"
            android:theme="@android:style/Theme.Translucent" />

    ...
    </application>
```

* Set `launchMode` of your main activity as `singleTask`.
 ```xml
    <application ...>
        ...
        <activity
            android:name="com.mycompany.testapp.MainActivity"
            android:label="@string/app_name"
            android:launchMode="singleTask" >
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
        ...
    </application>
```

### 3. Initialize GCM on your main activity

* Find the line you're calling `Valuepotion.getInstance().init(...)` method and call initGCM() method. It's usually located at onCreate() method of your main activity.

```java
ValuePotion.initGCM(this, SENDER_ID);
```

* `SENDER_ID` above is 'Project Number' from [Google Developers Console](https://console.developers.google.com/project).

### 4. Configure Vibration, LED (optional)

* You can customize vibration pattern and LED pattern. If not, OS default value will be used.

```java
ValuePotion.init(this, CLIENT_ID, SECRET_KEY);
ValuePotion.initGCM(this, SENDER_ID);

// LED Lights
int argb = 0xff2E691F;
int onMs = 2000;
int offMs = 1000;
ValuePotion.getInstance().setNotificationLights(context, argb, onMs, offMs);

// Vibration Pattern
long[] pattern = {...};
ValuePotion.getInstance().setNotificationVibrate(context, pattern);
```

* Vibration pattern and LED pattern will be stored in SharedPreference. If you want to revert the configuration, remove those lines from your code, remove your app on your phone and reinstall it to reset SharedPreference.
* To know more about vibration pattern, refer to [Android SDK Documentation](http://developer.android.com/reference/android/os/Vibrator.html).
 * To customize vibration pattern, you need to add the following permission to AndroidManifest.xml as well.

   ```xml
<uses-permission android:name="android.permission.VIBRATE" />
```

### 5. Misc

#### Remove push token

If you want to remove push token of a device from Valuepotion server, call the following method.

 ```java
ValuePotion.getInstance().unregisterPushToken();
```

#### Temporarily disable push notification

After GCM message, you can display push notification or just ignore it.
In your app, you can decide it dynamically.

 ```java
ValuePotion.getInstance().setPushEnable(context, false);        // Disable push notification.
ValuePotion.getInstance().setPushEnable(context, true);         // Enable push notification again.

boolean isEnabled = ValuePotion.getInstance().isPushEnabled();  // Returns boolean value indicating if push notification is enabled.
```

## Advanced: Listener
`ValuePotionListener` interface has callback methods to integrate campaigns.

You can call setListener method of ValuePotion instance to register callback.

```java

ValuePotion.getInstance().setListener( new ValuePotionListener(){
  // Implement Here ...
});

```

### 1. Callback Methods for Displaying Interstitial Ad
#### void onReadyToOpenInterstitial(ValuePotion vp, String placement);
This callback method is called when displaying interstitial ad is successfully done after calling `openInterstitial` method.

```java
@Override
void onReadyToOpenInterstitial(ValuePotion vp, String placement)
{
	// Put something you need to do when interstitial ad is displayed.
	// For example, you can pause game here.	
}
```

#### void onFailedToOpenInterstitial(ValuePotion vp, String placement, String error);
This callback method is called when displaying interstitial ad is failed after calling `openInterstitial` method.

```java
@Override
void onFailedToOpenInterstitial(ValuePotion vp, String placement, String error)
{
	// Put something you need to do when interstitial ad gets failed.
	// You can check reason of failure via error variable.
}
```

#### void onClosedInterstitial(ValuePotion vp, String placement);
This callback method is called when interstitial ad closes.

```java
@Override
void onClosedInterstitial(ValuePotion vp, String placement)
{
	// Put something you need to do when interstitial ad closes.
	// If you paused your game during ad is open, now you can resume it here.
}
```

### 2. Callback Methods for Caching Interstitial Ad
#### void onCachedInterstitial(ValuePotion vp, String placement);
This callback method is called when caching interstitial ad is successfully done after calling `cacheInterstitial` method.

```java
void onCachedInterstitial(ValuePotion vp, String placement)
{
  // Put something you need to do when caching interstitial ad is successfully done
}
```

#### void onFailedToCacheInterstitial(ValuePotion vp, String placement, String error);
This callback method is called when caching interstitial ad is failed after calling `cacheInterstitial` method.

```java
void onFailedToCacheInterstitial(ValuePotion vp, String placement, String error)
{
  // Put something you need to do when caching interstitial ad is failed.
  // You can check reason of failure via error variable.
}
```

### 3. Callback Methods for Interstitial Ad Action
#### void onRequestedOpen(ValuePotion vp, String placement, String url);
This callback method is called when user clicks external url while interstitial ad is displayed.

```java
void onRequestedOpen(ValuePotion vp, String placement, String url)
{
  // Put something you need to do when external url gets opened.
  // App soon goes background, so you can do something like saving user data, etc.
}
```

#### void onRequestedPurchase(ValuePotion vp, String placement, VPPurchase purchase);
This callback method is called when user pressed 'Purchase' button while interstitial ad of IAP type is displayed.

```java
void onRequestedPurchase(ValuePotion vp, String placement, VPPurchase purchase)
{
	// Put codes to process real purchase by using parameters: productId, quantity.
	// purchase object contains properties: name, productId, quantity, campaignId, contentId.
	// After purchase, call ValuePotion.getInstance().trackPurchaseEvent() method for revenue report.
}
```

#### void onRequestedReward(ValuePotion vp, String placement, ArrayList<VPReward> rewards);
This callback method is called when interstitial ad of Reward type is displayed.

```java
void onRequestedReward(ValuePotion vp, String placement, ArrayList<VPReward> rewards)
{
  // Array 'rewards' contains rewards which ad is about to give users.
  // With this information you should implement actual code to give rewards to users.
  for(VPReward reward : rewards){
    // The names of quantities of rewards to give
    Log.d(TAG, reward.toString() );
  }
}
```