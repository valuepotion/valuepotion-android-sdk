# Change Log

## v1.0.20
* New APIs
  ```java
  ValuePotion.getInstance().trackEvent(String category, String action, String label, int value);
ValuePotion.getInstance().trackEvent(String category, String action, String label, float value);
ValuePotion.getInstance().trackEvent(String category, String action, String label, double value);

  ValuePotion.getInstance().trackPurchaseEvent(String eventName, double revenueAmount, String currency, String orderId, String productId);
  
  ValuePotion.setUserAccountType(String accountType);
  ```

* Deprecated APIs
  ```java
  ValuePotion.getInstance().trackEvent(String action, String value);
  ValuePotion.getInstance().trackEvent(String action, Map<String, String> values);

  ValuePotion.getInstance().trackPurchaseEvent(String eventName, double revenueAmount, String currency);
  ValuePotion.getInstance().trackPurchaseEvent(String eventName, double revenueAmount, String currency, String orderId);
  ValuePotion.getInstance().trackPurchaseEvent(String eventName, double revenueAmount, String currency, VPPurchase purchase);
  ValuePotion.getInstance().trackPurchaseEvent(String eventName, double revenueAmount, String currency, String orderId, VPPurchase purchase);
  ```

## v1.0.19
* Now supports customizable notification LED.

  ```java
  ValuePotion.getInstance().setNotificationLights(Context context, int argb, int onMs, int offMs);
  ```
* Now supports customizable notification vibration pattern.

  ```java
  ValuePotion.getInstance().setNotificationVibrate(Context context, long[] pattern);
  ```

## v1.0.18
* Fixed a bug that push token is not registered very occasionally.

## v1.0.17
* Fixed a bug app crashing when users' android device does not have Google Play Services.

## v1.0.16
* Fixed a bug that WebView doesn’t work properly after showing interstitial ad.

## v1.0.15
* Renamed “location” to “placement” from all variables and method names.
* Use Advertising Id ( https://play.google.com/intl/en/about/developer-content-policy.html )
