# Valuepotion Android SDK 통합 가이드

## 기본 설정

### 1. 앱 정보 등록
먼저 [밸류포션](https://valuepotion.com) 웹사이트에 방문하여 SDK를 적용할 앱의 정보를 등록합니다. 앱 정보 등록을 완료하면 Client ID 와 Secret Key 가 발급됩니다.

### 2. Android 프로젝트에 SDK 가져오기

다운로드 받으신 파일의 압축을 해제한 후,
libs 디렉토리에 valuepotion.jar 를 추가합니다.

### 3. Dependencies 추가
밸류포션을 사용하시려면 아래 두 가지 dependencies 가 프로젝트에 포함되어야 합니다.

1. Google Play Services
 * [링크](https://developer.android.com/google/play-services/setup.html)를 참조하여 Eclipse 혹은 Android Studio 등 사용하시는 IDE 에 해당하는 방법으로 Google Play Services 를 설정하세요.
2. Android Support Library
 * [링크](http://developer.android.com/tools/support-library/setup.html)에서 "Adding libraries without resources" 항목을 참조하여 프로젝트에 Android Support Library 를 설정하세요.


## SDK 초기화
다음은 SDK를 초기화 하는 예제입니다. 

### init
첫번째 Activity 의 onCreate 에서 초기화를 합니다.

```java
import com.valuepotion.sdk.ValuePotion;
public class MyActivity extends Activity {
  @Override
  public void onCreate() {
    super.onCreate();
    // 밸류포션 웹사이트에서 발급받은 Client ID 와 Secret Key 를 사용해 SDK를 초기화 합니다.
    ValuePotion.init(this, "CLIENT_ID", "SECRET_KEY");
  }
}
```

### onStart / onStop

각 Activity 에서 onStart / onStop 내에 아래 코드를 추가합니다.

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

여기까지 설정하면 기본적인 session / install / update 이벤트 트래킹이 가능합니다.

## AndroidManifest.xml 설정

### 퍼미션 등록

```xml
<!-- Valuepotion Plugin Permissions -->
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
<uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
<uses-permission android:name="android.permission.READ_PHONE_STATE" />
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" /> 
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" /> 
<!-- Valuepotion Plugin Permissions end -->
```

### Valuepotion 컴포넌트 등록

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


## 인터스티셜 광고 연동

### 1. 인터스티셜 광고 노출하기
[밸류포션](https://valuepotion.com) 웹 사이트에서 생성한 캠페인을 인터스티셜 광고의 형태로 자신의 앱에 노출시킬 수 있습니다. 인터스티셜 광고를 화면에 띄우기 위해서는 플레이스먼트를 지정해야 하며, 지정하지 않는 경우 "default" 플레이스먼트가 사용됩니다.

플레이스먼트는 게임 내의 여러 지점에서 원하는 광고를 노출 시킬 수 있도록 하기 위해 부여하는 이름으로, 특별한 제약 없이 원하는 이름을
문자열로 지정하면 됩니다.

```java
// "default" 플레이스먼트에 대해 광고를 노출 합니다.
ValuePotion.getInstance().openInterstitial(activity);

// "main_menu" 플레이스먼트에 대해 광고를 노출 합니다.
ValuePotion.getInstance().openInterstitial(activity, "main_menu");
```


### 2. 인터스티셜 광고 캐싱하기
`openInterstitial:` 메소드를 사용하면 HTTP 를 통해 광고 데이터를 받아온 후 화면에 보여주기 때문에, 네트워크 상태에 따라 다소 지연이 발생할 수 있습니다. 최초 게임 구동 시 원하는 플레이스먼트에 대해 광고를 캐싱해두면,
이후 원하는 시점에 지연 없이 해당 광고를 화면에 노출시킬 수 있습니다.

```java
// 최초 "after_login" 플레이스먼트에 대해 광고를 캐싱합니다.
ValuePotion.getInstance().cacheInterstitial(activity, "after_login");

...

// 원하는 시점에 "after_login" 플레이스먼트에 대해 광고를 노출합니다.
ValuePotion.getInstance().openInterstitial(activity, "after_login");
```

### 3. 캐시가 있을 때만 인터스티셜 광고 노출하기
특정 플레이스먼트에 캐싱된 광고가 확실히 존재할 때에만 광고를 노출시킬 수도 있습니다.

```java
// "item_shop" 플레이스먼트에 캐싱된 광고가 존재하는지 체크합니다.
if (ValuePotion.getInstance().hasCachedInterstitial("item_shop") {
  // "item_shop" 플레이스먼트에 대해 광고를 노출합니다.
  ValuePotion.getInstance().openInterstitial(activity, "item_shop");
}
```


## 이벤트 트래킹
이벤트 트래킹 기능을 통해 게임에 대한 보다 세밀한 분석이 가능합니다. 또한, 이를 기반으로 유저 코호트를 생성하여 마케팅에 활용할 수 있습니다. 이벤트는 크게 비결제 이벤트와 결제 이벤트로 나뉩니다.

### 1.비결제 이벤트 트래킹
비결제 이벤트는 게임 내 결제와 무관한 이벤트로, 주로 사용자 행태 분석을 위해 사용합니다. 비결제 이벤트 트래킹을 위해서는 이벤트의 액션과 값을 지정해야 합니다. 다음은 비결제 이벤트를 전송하는 예제입니다.

```java
// 사용자가 3개의 아이템을 획득
ValuePotion.getInstance().trackEvent("get_item_ruby", 3);
ValuePotion.getInstance().trackEvent("get_item_ruby", 3f);
ValuePotion.getInstance().trackEvent("get_item_ruby", 3.0);
```

특별한 값이 필요치 않은 이벤트인 경우, 간단히 이벤트 이름만을 지정하여도 됩니다.

```java
// 사용자가 item shop 메뉴에 방문
String action = "enter_item_shop";
ValuePotion.getInstance().trackEvent(action);
```

이벤트에 계층을 두어 구분하고 싶을 때는 다음과 같이 하실 수 있습니다.

```java
String category = "item";
String action = "get_ruby";
String label = "reward_for_login";
int value = 30;
ValuePotion.getInstance().trackEvent(category, action, label, value);
```

### 2. 결제 이벤트 트래킹
결제 이벤트는 게임 내 구매(In App Purchase)가 발생했을 때 전송하는 이벤트입니다. 결제 이벤트를 트래킹하면 매출액, ARPU, ARPPU, PPU 등 유용한 지표들의 추이를 매일 확인할 수 있습니다. 다음은 게임 내에서 발생한 결제 이벤트를 전송하는 예제입니다.

#### 2-1. 이벤트 전송

```java
// 0.99 달러의 코인 아이템 구매가 발생
String action = "purchase_coin";
double amount = 0.99f;
String currency = "USD";
String orderId = 결제 성공 후 발행된 영수증 번호 ex> "1000000126295148";
String productId = 아이템의 식별자 ex> "com.valuepotion.tester.item_diamond_1";

ValuePotion.getInstance().trackPurchaseEvent(action, amount, currency, orderId, productId);
```

밸류포션은 In App Purchase (이하 IAP) 타입의 캠페인을 제공합니다. 게임 사용자가 IAP 타입의 광고를 통해 매출을 발생시킨 경우, 결제 이벤트에 추가 정보를 더해 전송하면 더욱 상세한 캠페인 별 매출 리포트를 제공 받으실 수 있습니다. 다음은 IAP 광고로부터 발생한 결제 이벤트를 전송하는 예제입니다.

*`onRequestedPurchase` 콜백 메소드에 대한 보다 자세한 정보는 "고급: Listener" 섹션의 `void onRequestedPurchase(ValuePotion vp, String placement, VPPurchase purchase);` 항목을 참고하십시오.*

```java

ValuePotionListener listener = new ValuePotionListener(){
  @Override
  void onRequestedPurchase(ValuePotion vp, String placement, VPPurchase purchase)
  {
    // 요청 받은 결제를 진행합니다

    ...

    // IAP 캠페인을 통해 1,200원의 다이아몬드 아이템 구매가 발생. purchase 객체에 있는 campaignId, contentId 를 함께 전송.
    String action = "purchase_coin";
    double amount = 0.99f;
    String currency = "USD";
    String orderId = 결제 성공 후 발행된 영수증 번호 ex> "1000000126295148";
    String productId = 아이템의 식별자 ex> "com.valuepotion.tester.item_diamond_1";
    String campaignId = purchase.getCampaignId();
    String contentId = purchase.getContentId();

    ValuePotion.getInstance().trackPurchaseEvent(action, amount, currency, orderId, productId, campaignId, contentId);
  }
  
  ...
  
};
```

비결제 이벤트처럼 category 와 label 을 지정할 수 있습니다.
```
ValuePotion.getInstance().trackPurchaseEvent(category, action, label, amount, currency, orderId, productId, campaignId, contentId);
```

#### 참고
* 정확한 집계를 위해, 결제 이벤트 전송 시에는 실제 발생한 결제 금액과 통화 코드를 지정해주십시오.
* 통화 코드는 [ISO 4217](http://en.wikipedia.org/wiki/ISO_4217) 표준을 따릅니다.

### 3. 이벤트 트래킹 테스트
SDK의 테스트 모드를 통해 정상적으로 이벤트가 전송되는지 여부를 쉽게 확인할 수 있습니다. 테스트 모드를 활성화 시키는 방법은 다음과 같습니다.

```java
// 테스트 모드로 설정. 기본값은 NO
ValuePotion.getInstance().setTest(true);
```

테스트 모드로 빌드된 앱에서 전송되는 이벤트는 [밸류포션](https://valuepotion.com) 웹사이트의 개발자 콘솔 메뉴에서 실시간으로 확인 가능합니다.

**주의** : 앱 스토어에 제출하기 위한 최종 빌드 시에는 반드시 테스트 모드를 해제하십시오. 테스트 모드에서 전송된 이벤트는 개발자 콘솔 메뉴에서만 출력되고, 실제 집계에서는 제외됩니다.



## 사용자 정보 연동
이벤트 트래킹과는 별도로, 게임 사용자의 추가 정보에 대한 수집이 가능합니다. 현재 밸류포션에서 지원하는 사용자 정보는 사용자의 계정 ID, 사용자가 속한 게임 서버의 ID, 생년월일, 성별, 레벨, 친구 수, 계정 유형의 7가지입니다. 모든 항목은 선택적이므로, 필요치 않다면 어떤 것도 설정할 필요가 없습니다.

이 정보들을 이용해 유저 코호트를 생성하여 마케팅에 활용할 수있습니다. 사용자 정보는 게임의 진행 중 변경이 있을 때마다 새로이 설정하여 주시면 자동으로 밸류포션과 연동됩니다.

```java
ValuePotion.setUserId("support@valuepotion.com");
ValuePotion.setUserServerId("server1");
ValuePotion.setUserBirth("19830328");
ValuePotion.setUserGender("M");
ValuePotion.setUserLevel(32);
ValuePotion.setUserFriends(219);
ValuePotion.setUserAccountType("guest");
```

각 사용자 정보 항목에 대한 세부 내용은 다음과 같습니다.

이름            | 설명
-------------- | ------------
**userId**    | 게임 내에서 사용되는 사용자의 계정 id를 설정합니다.
**serverId**  | 게임 유저를 서버 별로 식별해야 하는 경우 유저가 속한 서버의 id를 설정합니다.<br>serverId를 기준으로 서버별 통계를 확인할 수 있습니다.
**birth**     | 사용자의 생년월일 8자리를 문자열로 세팅합니다.<br>연도 정보만 아는 경우 "19840000"과 같이 뒤 4자리를 0으로 채웁니다.<br>생일 정보만 아는 경우 "00001109"와 같이 앞 4자리를 0으로 채웁니다.
**gender**    | 남성인 경우 "M", 여성인 경우 "F" 문자열로 설정합니다.
**level**     | 사용자의 게임 내 레벨을 설정합니다.
**friends**   | 사용자의 친구 수를 설정합니다.
**accountType**   | 사용자의 로그인 계정 타입을 설정합니다. (facebook, google, guest 등)


## Push Notification 연동
밸류포션 Push Notification API와 연동하면, 손쉽게 Push 타입의 캠페인을 생성하여 사용자에게 메시지를 전송할 수 있습니다. 장기간 게임을 플레이 하지 않은 유저들이 다시 접속하도록 유도하거나, 게임 내 이벤트 소식을 알리는 등 다방면으로 활용이 가능합니다.

### 1. 인증서 등록
[밸류포션](https://valuepotion.com) 웹사이트에서 앞서 등록한 앱 정보를 업데이트 해야 합니다. 앱 정보 수정 페이지에서 GCM ApiKey를 등록하십시오. GCM ApiKey 는 [Google Developers Console](https://console.developers.google.com/project)에서 발급 받은 Server Key 를 의미합니다.

### 2. AndroidManifest.xml 설정

* 아래 퍼미션 관련 구문을 삽입해주세요.

```xml
<manifest ...>

    <uses-permission android:name="android.permission.GET_ACCOUNTS" />
    <uses-permission android:name="android.permission.WAKE_LOCK" />
    <uses-permission android:name="com.google.android.c2dm.permission.RECEIVE" />

    <!--
       'PACKAGE_NAME' 을 앱의 패키지 네임으로 변경하세요.
        예)
        <application> 태그의 'package' 속성 값이 'com.valuepotion.testapp' 라면,
        'com.valuepotion.testapp.permission.C2D_MESSAGE' 로 값을 지정하세요.
    -->
    <permission android:name="PACKAGE_NAME.permission.C2D_MESSAGE" android:protectionLevel="signature" />
    <uses-permission android:name="PACKAGE_NAME.permission.C2D_MESSAGE" />

    <application ...
```

* 아래와 같이 receiver 와 service, 그리고 activity 를 등록해주세요.

```xml
    <application ...>
        ...

        <!--
           'PACKAGE_NAME' 을 앱의 패키지 네임으로 변경하세요.
        -->

        <receiver
            android:name="com.valuepotion.sdk.push.GcmBroadcastReceiver"
            android:permission="com.google.android.c2dm.permission.SEND">
            <intent-filter>
                <action android:name="com.google.android.c2dm.intent.RECEIVE" />
                <action android:name="com.google.android.c2dm.intent.REGISTRATION" />

                <category android:name="PACKAGE_NAME" />
            </intent-filter>
        </receiver>
        <receiver
            android:name="com.valuepotion.sdk.push.NotificationOpenedReceiver">
            <intent-filter>
                <action android:name="com.valuepotion.sdk.push.NOTIFICATION_OPENED" />
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

* 메인 액티비티에 launchMode 속성을 아래와 같이 `singleTask` 로 설정해주세요.
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

### 3. Activity 에 초기화 구문 삽입

* `Valuepotion.getInstance().init(...)` 구문을 실행한 곳에 가서 아래와 같이 initGCM 함수를 호출해주세요. 보통 메인 액티비티의 onCreate() 함수에 위치해 있습니다.

```java
ValuePotion.initGCM(this, SENDER_ID);
```

* 위의 SENDER_ID 는 [Google Developers Console](https://console.developers.google.com/project)에서 발급 받은 Project Number 입니다.

### 4. 진동, LED 설정

* 선택적으로 진동 패턴과 LED 패턴을 커스터마이징 할 수 있습니다. 설정을 하지 않으면 OS 디폴트 값이 사용됩니다.

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

* 진동 패턴과 LED 패턴에 관한 값은 SharedPreference 에 저장됩니다. 만약 설정을 되돌리고 싶다면, 위 구문을 제거하신 후 앱을 삭제한 뒤 재설치해주세요.
* 진동 패턴에 관해서는 [안드로이드 SDK 공식 문서](http://developer.android.com/reference/android/os/Vibrator.html) 를 참고해주세요.
 * 그리고 진동을 커스터마이징 해서 쓰기 원하시면 위의 코드와 더불어 아래의 퍼미션을 AndroidManifest.xml 에 추가해야 합니다.

   ```xml
<uses-permission android:name="android.permission.VIBRATE" />
```

### 5. 기타 설정

#### 푸시 토큰 제거하기

Valuepotion 서버로부터 해당 디바이스의 푸시 토큰을 제거하고 싶으면

 ```java
ValuePotion.getInstance().unregisterPushToken();
```
를 실행시키면 됩니다.

#### 푸시 임시 비활성화

GCM 메시지는 수신하지만, 그걸 푸시 알림으로 표시할 지 말지를 결정합니다.
App 내에서 푸시를 제어할 때 사용합니다.

 ```java
ValuePotion.getInstance().setPushEnable(context, false);        // 푸시를 비활성화합니다.
ValuePotion.getInstance().setPushEnable(context, true);         // 푸시를 다시 활성화합니다.

boolean isEnabled = ValuePotion.getInstance().isPushEnabled();  // 푸시가 활성화되어 있는지 boolean 리턴을 받습니다.
```

## 고급: Listener
`ValuePotionListener` 인터페이스에는 캠페인 연동 시 활용 가능한 콜백 메소드가 정의되어 있습니다.

ValuePotion 인스턴스의 setListener 함수를 이용하여 콜백에 반응할 수 있습니다.

```java

ValuePotion.getInstance().setListener( new ValuePotionListener(){
  // Implement Here ...
});

```

### 1. Interstitial 노출 관련
#### void onReadyToOpenInterstitial(ValuePotion vp, String placement);
`openInterstitial` 메소드 호출 후, 인터스티셜 광고가 성공적으로 화면에 노출되는 시점에 호출됩니다.

```java
@Override
void onReadyToOpenInterstitial(ValuePotion vp, String placement)
{
  // 인터스티셜 광고가 열릴 때 필요한 작업이 있다면 여기에 구현합니다.
  // 실행 중인 게임을 pause 시키는 등의 처리를 할 수 있습니다.
}
```

#### void onFailedToOpenInterstitial(ValuePotion vp, String placement, String error);
`openInterstitial` 메소드 호출 후, 인터스티셜 광고가 화면에 노출되지 못하는 경우 호출됩니다.

```java
@Override
void onFailedToOpenInterstitial(ValuePotion vp, String placement, String error)
{
  // 인터스티셜 광고 노출에 실패했을 때 필요한 작업이 있다면 여기에 구현합니다.
  // 실패한 원인은 error 를 통해 확인할 수 있습니다.
}
```

#### void onClosedInterstitial(ValuePotion vp, String placement);
인터스티셜 광고가 열려있는 상태에서 닫힐 때 호출됩니다.

```java
@Override
void onClosedInterstitial(ValuePotion vp, String placement)
{
  // 인터스티셜 광고가 닫힐 때 필요한 작업이 있다면 여기에 구현합니다.
  // 광고가 열려있는 동안 게임을 pause 시켰다면, 여기서 resume 시키는 등의 처리를 할 수 있습니다.
}
```

### 2. Interstitial 캐싱 관련
#### void onCachedInterstitial(ValuePotion vp, String placement);
`cacheInterstitial` 메소드 호출 후, 성공적으로 광고가 캐싱 되었을 때 호출됩니다.

```java
void onCachedInterstitial(ValuePotion vp, String placement)
{
  // 인터스티셜 광고 캐싱이 완료된 후 필요한 작업이 있다면 여기에 구현합니다.
}
```

#### void onFailedToCacheInterstitial(ValuePotion vp, String placement, String error);
`cacheInterstitial` 메소드 호출 후, 광고 캐싱에 실패했을 때 호출됩니다.

```java
void onFailedToCacheInterstitial(ValuePotion vp, String placement, String error)
{
  // 인터스티셜 광고 캐싱에 실패했을 때 필요한 작업이 있다면 여기에 구현합니다.
  // 실패한 원인은 error 를 통해 확인할 수 있습니다.
}
```

### 3. Interstitial 액션 관련
#### void onRequestedOpen(ValuePotion vp, String placement, String url);
인터스티셜 광고 노출 상태에서 사용자가 외부 링크를 클릭하는 경우 발생합니다.

```java
void onRequestedOpen(ValuePotion vp, String placement, String url)
{
  // 외부 링크를 열 때 필요한 작업이 있다면 여기에 구현합니다.
  // 앱이 Background로 진입하게 되므로, 사용자 데이터를 저장하는 등의 처리를 할 수 있습니다.
}
```

#### void onRequestedPurchase(ValuePotion vp, String placement, VPPurchase purchase);
IAP 타입의 인터스티셜 광고 노출 상태에서 사용자가 '결제하기'를 선택하는 경우 발생합니다.

```java
void onRequestedPurchase(ValuePotion vp, String placement, VPPurchase purchase)
{
  // 인자로 전달된 purchase 오브젝트를 가지고 실제 결제를 진행하도록 구현합니다.
  // purchase 오브젝트는 name, productId, quantity, campaignId, contentId 프로퍼티를 담고 있습니다.
  // 결제가 완료된 이후 ValuePotion.getInstance().trackPurchaseEvent() 메소드를 사용해
  // 결제 이벤트를 전송하면 매출 리포트가 집계됩니다.
}
```

#### void onRequestedReward(ValuePotion vp, String placement, ArrayList<VPReward> rewards);
Reward 타입의 인터스티셜 광고가 노출될 때 발생합니다.

```java
void onRequestedReward(ValuePotion vp, String placement, ArrayList<VPReward> rewards)
{
  // rewards 배열에는 해당 광고를 통해 사용자에게 지급하고자 하는 리워드 오브젝트들이 담겨있습니다.
  // 이 정보들을 가지고 사용자에게 리워드를 지급하는 코드를 구현합니다.
  for(VPReward reward : rewards){
    Log.d(TAG, reward.toString() );
  }
}
```
