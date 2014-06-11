# Valuepotion SDK for Android - Getting Started

## Before You Begin

### 1. Register Your App
Visit [ValuePotion](https://valuepotion.com) website and register the information of your app. After that, you will be given a **Client ID** and a **Secret Key**.

### 2. Import the SDK into your Android project

Unzip the sdk downloaded, and add `valuepotion.jar` to `libs` folder.

## Initialize SDK
The following code is to initialize SDK.

### init
We recommend you to initialize SDK at `onCreate` method at `android.app.Application` class.

```java
import com.valuepotion.sdk.ValuePotion;
public class MyApplication extends Application {
  @Override
  public void onCreate() {
    super.onCreate();
    // Initialize SDK using Client ID and Secret Key that you got from ValuePotion website.
    ValuePotion vp = ValuePotion.init(this, "CLIENT_ID", "SECRET_KEY");
  }
}
```

Or, you can initialize SDK at `onCreate` method at the first activity that will be launched.

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

## Integrate with Interstitial Ads

### 1. Display Interstitial Ads
If you've created a campaign at [ValuePotion](https://valuepotion.com), you can display it as an interstitial ad at your own app. Before displaying interstitial ads, you should set up a location. Otherwise, "default" location will be used by default.

**Location** is a name to distinguish many points where you want to display ads. There's no restriction but it just should be a string.

```java
// Display ads at "default" location.
ValuePotion.getInstance().openInterstitial();

// Display ads at "main_menu" location.
ValuePotion.getInstance().openInterstitial("main_menu");
```

### 2. Cache Interstitial Ads
Using `openInterstitial()` method, the SDK will download data for ads via HTTP and display on screen. So it takes some time. If you cache ads when your game launches, you can display the ads at any time with no delay.


```java
// If you cache an ad for "after_login" location once,
ValuePotion.getInstance().cacheInterstitial("after_login");

...

// Later on, you can display the ad with no delay.
ValuePotion.getInstance().openInterstitial("after_login");
```

### 3. Display Interstitial Ads Only When Caches are Available
You can display interstitial ads only when caches are available.

```java
// Check if the cache for "item_shop" location exists.
if (ValuePotion.getInstance().hasCachedInterstitial("item_shop") {
  // then, display the ad for "item_shop" location.
  ValuePotion.getInstance().openInterstitial("item_shop");
}
```


## Event Tracking
You can analyze your game with event tracking. And based on events you can create cohort to use for marketing. There are non-payment event and payment event.

### 1. Non-Payment Event
Non-payment event is not related to In-App Purchase. You can use non-payment event to analyze user behavior. To use non-payment event, you should define its name and values. The following code is an example to send non-payment event.

```java
// User has been cleared 3rd stage.
ValuePotion.getInstance().trackEvent("stage_clear","3");
ValuePotion.getInstance().trackEvent("stage_clear",3);
ValuePotion.getInstance().trackEvent("stage_clear",3f);
ValuePotion.getInstance().trackEvent("stage_clear",3.0);
```

If there's no specific value needed, you can use event name only.

```java
// User has visited "item shop" menu.
ValuePotion.getInstance().trackEvent("enter_item_shop");
```


### 2. Payment Event
Payment event is tracked when In-App Purchase(In-App Billing) has occurred. If you track payment events, you can check daily statistics of Revenue, ARPU, ARPPU, PPU, etc.
The following code is an example to send payment event occurred in your game.

#### 2-1. ValuePotionListener

`ValuePotion.ValuePotionListener` interface provides the following callback methods.

```java
public static interface ValuePotionListener {
    /**
     * This callback method is called when caching interstitial ad is successfully done after calling cacheInterstitial() method.
     */
    void onCachedInterstitial(ValuePotion vp, String location);

    /**
     * This callback method is called when caching interstitial ad is failed after calling `cacheInterstitial()` method.
     */
    void onFailedToCacheInterstitial(ValuePotion vp, String location, String error);

    /**
     * This callback method is called right before displaying interstitial ad.
     */
    void onReadyToOpenInterstitial(ValuePotion vp, String location);

    /**
     * This callback method is called when interstitial ad is not valid at the time opening it event though the data of interstitial ad exists.
     */
    void onFailedToOpenInterstitial(ValuePotion vp, String location, String error);

    /**
     * This callback method is called after view of interstitial ad closes.
     */
    void onClosedInterstitial(ValuePotion vp, String location);

    /**
     * This callback method is called when user clicks external url while interstitial ad is displayed.
     */
    void onRequestedOpen(ValuePotion vp, String location, String url);

	/**
	 * This callback method is called when user pressed 'Purchase' button while interstitial ad of IAP type is displayed.
	 */
    void onRequestedPurchase(ValuePotion vp, String location, VPPurchase purchase);

	/**
	 * This callback method is called when interstitial ad of Reward type is displayed.
	 */
    void onRequestedReward(ValuePotion vp, String location, ArrayList<VPReward> rewards);

  }
```

You can call setListener method of ValuePotion instance to register callback.

```java

ValuePotion.getInstance().setListener( new ValuePotionListener(){
  // Implement Here ...
});

```

#### 2-2. Track Event

```java
// User purchased $0.99 coin item.
ValuePotion.getInstance().trackPurchaseEvent("purchase_coin",0.99,"USD");
```

ValuePotion provides campaign of In-App Purchase (IAP) type. When a user makes revenue via an ad of IAP type, if you add extra info to payment event, you can get revenue report per campaign in detail. The following code is how to send payment event which occurred from IAP ad.

* Too see more information aboue callback method `onRequestedPurchase`, please see **void onRequestedPurchase(ValuePotion vp, String location, VPPurchase purchase);** item under **Advanced: Listener** section. *

```java

ValuePotionListener listener = new ValuePotionListener(){
  @Override
  void onRequestedPurchase(ValuePotion vp, String location, VPPurchase purchase)
  {
    // Proceed the requested payment

    ...

    // User purchased some Diamond item for KRW 1,200. So you're attaching purchase object as payment event parameters.
    ValuePotion.getInstance().trackPurchaseEvent("iap_diamond",1200,"KRW",purchase);
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

**Warning** : Before submitting your app to app store, please disable test mode. Events sent form test mode are only displayed on Developer's console but excluded from analysis.



## Integrate User Information
You can collect user information as well as events. Possible fields of user information are user id, server id which user belongs to, birthdate, gender, level and number of friends. All of them are optional so you can choose which fields to collect.

You can use this information for marketing by creating user cohort. You can update your information when it changes to integrate with ValuePotion.

```java
ValuePotion.getInstance().setUserId("support@valuepotion.com");
ValuePotion.getInstance().setUserServerId("server1");
ValuePotion.getInstance().setUserBirth("19830328");
ValuePotion.getInstance().setUserGender("M");
ValuePotion.getInstance().setUserLevel(32);
ValuePotion.getInstance().setUserFriends(219);
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


## Integrate Push Notification
If you integrate with Push Notification API, you can easily create campaigns of Push type and send message to users. So you can wake up users who haven't played game for long time, or you can also notify users new events in game, etc.

### 1. Register Certificate
Visit [ValuePotion](https://valuepotion.com) website and update your app information. Please fill in GCM ApiKey at **App Edit** page.

### 2. Implement GCM Client

Please visit [Implementing GCM Client](http://developer.android.com/intl/ko/google/gcm/client.html) for more information. The following example is based on the link.

### 3. Enable Push Notification
To enable push notification, you should send registrationId and process push notification you receive. See the following code.

```java
String regid = getRegistrationId(this);
ValuePotion.getInstance().registerPushToken(regid);
```

### 4. Disable Push Notification
To disable push notification, run the following code.

```java
// Disable Push Notification
ValuePotion.getInstance().unregisterPushToken();
```

#### 4-1. Temporarily Enable/Disable Push Notification

You can disable push notification at client side temporarily so that you don't show push message after receiving it. Use this function when you need to control push notification at app.

```java

ValuePotion.getInstance().setPushEnable(context, false);

// Returns boolean value whether push notification is enabled.
ValuePotion.getInstance().isPushEnabled();

ValuePotion.getInstance().setPushEnable(context, true);

```

### 5. Process Push Notification

Create GCMIntentService extending GCMBaseIntentService and call `ValuePotion.treatPushMessage(context, bundle)` method at onMessage() method. If that method returns true, you don't have to do anything about that push notification because it's sent by ValuePotion for promotion.

```java
public class GCMIntentService extends GCMBaseIntentService {
  @Override
  protected void onMessage(Context context, Intent intent) {
    Log.v(TAG, "onMessage");
    Bundle bundle = intent.getExtras();
    
    if( ValuePotion.treatPushMessage(context, bundle) ){
      // A push notification message sent by ValuePotion. Nothing to do.
    }
    else{
      // Your own push notification message.
    }
  }
}
```

## Advanced: Listener
`ValuePotionListener` interface has callback methods to integrate campaigns.

### 1. Callback Methods for Displaying Interstitial Ad
#### void onReadyToOpenInterstitial(ValuePotion vp, String location);
This callback method is called when displaying interstitial ad is successfully done after calling `openInterstitial` method.

```java
@Override
void onReadyToOpenInterstitial(ValuePotion vp, String location)
{
	// Put something you need to do when interstitial ad is displayed.
	// For example, you can pause game here.	
}
```

#### void onFailedToOpenInterstitial(ValuePotion vp, String location, String error);
This callback method is called when displaying interstitial ad is failed after calling `openInterstitial` method.

```java
@Override
void onFailedToOpenInterstitial(ValuePotion vp, String location, String error)
{
	// Put something you need to do when interstitial ad gets failed.
	// You can check reason of failure via error variable.
}
```

#### void onClosedInterstitial(ValuePotion vp, String location);
This callback method is called when interstitial ad closes.

```java
@Override
void onClosedInterstitial(ValuePotion vp, String location)
{
	// Put something you need to do when interstitial ad closes.
	// If you paused your game during ad is open, now you can resume it here.
}
```

### 2. Callback Methods for Caching Interstitial Ad
#### void onCachedInterstitial(ValuePotion vp, String location);
This callback method is called when caching interstitial ad is successfully done after calling `cacheInterstitial` method.

```java
void onCachedInterstitial(ValuePotion vp, String location)
{
  // Put something you need to do when caching interstitial ad is successfully done
}
```

#### void onFailedToCacheInterstitial(ValuePotion vp, String location, String error);
This callback method is called when caching interstitial ad is failed after calling `cacheInterstitial` method.

```java
void onFailedToCacheInterstitial(ValuePotion vp, String location, String error)
{
  // Put something you need to do when caching interstitial ad is failed.
  // You can check reason of failure via error variable.
}
```

### 3. Callback Methods for Interstitial Ad Action
#### void onRequestedOpen(ValuePotion vp, String location, String url);
This callback method is called when user clicks external url while interstitial ad is displayed.

```java
void onRequestedOpen(ValuePotion vp, String location, String url)
{
  // Put something you need to do when external url gets opened.
  // App soon goes background, so you can do something like saving user data, etc.
}
```

#### void onRequestedPurchase(ValuePotion vp, String location, VPPurchase purchase);
This callback method is called when user pressed 'Purchase' button while interstitial ad of IAP type is displayed.

```java
void onRequestedPurchase(ValuePotion vp, String location, VPPurchase purchase)
{
	// Put codes to process real purchase by using parameters: productId, quantity.
	// purchase object contains properties: name, productId, quantity, campaignId, contentId.
	// After purchase, call ValuePotion.getInstance().trackPurchaseEvent() method for revenue report.
}
```

#### void onRequestedReward(ValuePotion vp, String location, ArrayList<VPReward> rewards);
This callback method is called when interstitial ad of Reward type is displayed.

```java
void onRequestedReward(ValuePotion vp, String location, ArrayList<VPReward> rewards)
{
  // Array 'rewards' contains rewards which ad is about to give users.
  // With this information you should implement actual code to give rewards to users.
  for(VPReward reward : rewards){
    // The names of quantities of rewards to give
    Log.d(TAG, reward.toString() );
  }
}
```
