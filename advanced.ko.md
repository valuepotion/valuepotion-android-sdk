## SDK 설치 매뉴얼

#### 프로젝트 요구 사항

* Eclipse
* ADT
* valuepotion.jar
* IAPUtil.java

#### 구성

ValuePotion Android SDK는 다음과 같이 구성되어 있습니다.

* valuepotion.jar
* AndroidManifest.xml

#### 통합 및 설정

##### SDK 파일 복사

valuepotion.jar 파일을 libs 디렉토리에 복사해 넣습니다.
![libs 디렉토리](./images/android_integration.png?raw=true =300x)

##### 필수 framework 설정

AndroidManifest.xml을 참고하여 추가 퍼미션과 Activity를 설정합니다.
```xml
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
    <uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
    <uses-permission android:name="android.permission.READ_PHONE_STATE" />
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" /> <!-- optional -->
    <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" /> <!-- optional -->
```

Interstitial/Push 캠페인을 사용할 때에는 필수적으로 아래 activity 의 설정을 하셔야 합니다.
```xml
    <application .. >
    ...
        <activity
            android:name="com.valuepotion.sdk.ValuePotionActivity"
            android:theme="@android:style/Theme.Translucent" >
        </activity>
    ...
    </application>
```


## 기본 연동

#### SDK 초기화

ValuePotion 클래스의 init(Context context, String clientId, String secretKey) 메소드를 이용해 간단하게 SDK를 초기화 할 수 있습니다.
초기화 위치는 어느 곳이나 상관 없지만, 가능한 앱이 실행될 때 즉시 호출되는 위치에 추가하는 것이 좋습니다.
일반적으로 Application 를 상속하고, onCreate() 메소드 내부에 추가하는 것이 적당합니다.

client id와 secret key는 [valuepotion.com](http://valuepotion.com)에서 등록하신 앱 정보 화면에서 확인 가능합니다.

##### Application.onCreate()

```java
package your.test.app.package;
import android.app.Application;
import com.valuepotion.sdk.ValuePotion;
public class MyApplication extends Application {
    private static final String TAG = "TestApplication";
    @Override
    public void onCreate() {
        super.onCreate();
        ValuePotion.init(getApplicationContext(), "app client id", "app secret key");
        }
}
```

##### AndroidManifest.xml

AndroidManifest.xml 의 application 태그의 'android:name'속성에, 위에서 정의한 MyApplication을 지정합니다.

```xml
...
<application
        android:name="your.test.app.package.MyApplication"
        ...
        >
...
```

##### Activity

```java
public class MainActivity extends Activity{
    @Override
    public void onCreate(Bundle savedInstanceState) {
        ...
    }

    @Override
    protected void onStart() {
        super.onStart();
        // Session의 트래킹 시작처리
        ValuePotion.getInstance().onStart(this);
    }

    @Override
    protected void onStop() {
        super.onStop();
        // Session의 트래킹 종료처리
        ValuePotion.getInstance().onStop(this);
    }

    @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        if( VPUnityBinder.gInstance.onActivityResult(this, requestCode, resultCode, data) ){
            // ValuePotion 처리 완료.
        }
        else{
            // 사용자의 ActivityResult 처리
            super.onActivityResult(requestCode, resultCode, data);
        }
    }
}
```

여기까지만 설정하셔도 기본적인 세션 트래킹과 install / update 이벤트 트래킹이 가능합니다.

#### 사용자 추가 정보 설정

사용자의 추가 정보에 대한 수집이 가능합니다. 해당 정보를 이용해 유저 코호트를 생성하여 활용할 수 있습니다.
현재는 사용자의 계정 id, 성별, 연령 정보를 추가로 설정하실 수 있습니다.


```java
ValuePotionManager.SetUserId("user1234");
ValuePotionManager.SetUserServerId("server1");
ValuePotionManager.SetUserLevel(10);
ValuePotionManager.SetUserGender("M");
ValuePotionManager.SetUserBirth("19840000");
ValuePotionManager.SetUserFriends(12);
```

또는

```java
Dictionary<string, string> userInfo = new Dictionary<string, string>();
userInfo.Add("userId", "user1234");
userInfo.Add("serverId", "server1");
userInfo.Add("level", "10");
userInfo.Add("gender", "M");
userInfo.Add("birth", "19840000");
userInfo.Add("friends", "12");
ValuePotionManager.SetUserInfo(userInfo);
```


##### 추가 정보 항목

이름           | 설명
-------------- | ------------
userId         | 게임 내에서 사용되는 사용자의 계정 id를 할당합니다.
birth          | 사용자의 생년월일 8자리를 문자열로 할당합니다.
               | 연도 정보만 아는 경우 "19840000"과 같이 생일 4자리를 0으로 채워 할당합니다.
               | 생일 정보만 아는 경우 "00001109"와 같이 연도 4자리를 0으로 채워 할당합니다.
gender         |  남성인 경우 "M", 여성인 경우 "F" 문자열로 할당합니다.

## 캠페인 연동

[valuepotion.com](http://valuepotion.com)에서 생성한 캠페인을 interstitial 형태로 앱에 노출시킬 수 있습니다.
모든 interstitial은 로케이션 단위로 관리되며, 로케이션 이름은 정해진 규칙 없이 자유롭게 할당하실 수 있습니다.

#### interstitial 캐싱하기

앱 실행 초기에 미리 캠페인 데이터를 받아 캐싱하고, 이후 필요한 시점에 즉시 interstitial을 보여줄 수 있습니다.
다음의 코드는 main_menu, item_shop 이라는 2가지 로케이션에 대해 interstitial을 캐싱하는 예제입니다.

```java
ValuePotion.getInstance().cacheInterstitial(CurrentActivity.this, "main_menu");
ValuePotion.getInstance().cacheInterstitial(CurrentActivity.this, "item_shop");
```

#### interstitial 노출하기

interstitial을 화면에 보여주고자 할 때에는 openInterstitial(String) 메소드를 사용합니다.
openInterstitial(String) 메소드는 지정된 로케이션에 캐시가 존재하는 경우 해당 캐시를 가지고 즉시 화면에 보여줍니다.
만약 해당 로케이션에 캐시가 존재하지 않으면 서버와 통신하여 데이터를 가져와 화면에 노출시키게 됩니다.
다음은 main_menu 로케이션에서 interstitial을 띄우는 예제입니다.

```java
ValuePotion.getInstance().openInterstitial(CurrentActivity.this, "main_menu");
```

#### 캐시가 있을 때만 interstitial 노출하기

interstitial 데이터를 필요 시마다 서버로부터 받아와서 화면에 보여주게 되면 다소 딜레이가 발생할 수 있습니다.
따라서 앱 실행 초기에 모든 로케이션에 대해 캐싱해 두고, 캐시가 존재하는 로케이션에 대해서만 interstitial을 노출하고 싶을 수도 있습니다.
다음은 item_shop 로케이션에 캐시가 존재하는 경우에만 interstitial을 노출하도록 처리한 예제입니다.

```java
if ( ValuePotion.getInstance().hasCachedInterstitial("item_shop") ) {
    ValuePotion.getInstance().openInterstitial(CurrentActivity.this, "item_shop");
}
```

#### Delegate 메소드

ValuePotionListener 인터페이스에는 캠페인 연동 시 활용 가능한 delegate 메소드들이 정의되어 있습니다.
모든 delegate 메소드는 optional이므로, 필요하지 않은 경우 구현할 필요가 없습니다.

```java
ValuePotionListener listener = new ValuePotionListener(){ ... };
ValuePotion.getInstance().setListener(listener);
```

##### 캐싱 관련 delegate

interstitial의 캐싱 성공 / 실패에 대한 delegate 처리를 할 수 있습니다.
```java
@Override
void onCachedInterstitial(ValuePotion vp, String placement)
{
    // interstitial 캐싱이 완료되었을 때 필요한 작업을 추가합니다.
}

@Override
void onFailedToCacheInterstitial(ValuePotion vp, String placement, ValuePotionException e)
{
    // interstitial 캐싱에 실패했을 때 필요한 작업을 추가합니다.
}
```

##### 노출 관련 delegate

interstitial의 노출 성공 / 실패 / 종료에 대한 delegate 처리를 할 수 있습니다.

```java
@Override
void onReadyToOpenInterstitial(ValuePotion vp, String placement)
{
    // interstitial이 노출될 때 필요한 작업을 추가합니다.
}

@Override
void onFailedToOpenInterstitial(ValuePotion vp, String placement, String error)
{
    // interstitial 노출에 실패했을 때 필요한 작업을 추가합니다.
}

@Override
void onClosedInterstitial(ValuePotion vp, String placement)
{
    // interstitial이 닫힐 때 필요한 작업을 추가합니다.
}
```

##### 액션 관련 delegate

사용자가 interstitial 내부에서 발생시킨 액션에 대한 delegate 처리를 할 수 있습니다.

```java
@Override
void onRequestedOpenURL(ValuePotion vp, String placement, String url)
{
    // interstitial에서 외부 링크에 대한 클릭이 발생했을 때 호출됩니다.
    // 일반적으로 외부 링크를 클릭하면 현재 앱은 background로 들어가게 되므로, 이 경우 필요한 처리를 여기서 구현합니다.
}

@Override
void onRequestedPurchase(ValuePotion vp, String placement, Purchase purchase)
{
    // In App Purchase 캠페인의 interstitial 내부에서 사용자가 구매하기를 선택했을 때 호출됩니다.
    // purchase 객체는 productIdentifier, quantity, name 속성을 가지고 있습니다.
    // 이 속성 정보를 가지고 실제 결제 진행을 위한 코드를 여기에서 구현합니다.
}

@Override
void onRequestedReward(ValuePotion vp, String placement, Reward reward)
{
    // 리워드 캠페인의 interstitial이 화면에 보여질 때 호출됩니다.
    // reward 객체는 name, quantity 속성을 가지고 있습니다.
    // 이 속성 정보를 가지고 해당 리워드 아이템을 사용자에게 지급하는 코드를 여기에서 구현합니다.
}
```

## 커스텀 이벤트 연동

커스텀 이벤트 전송 기능을 통해 앱에 대한 보다 세밀한 분석이 가능합니다.
또한, 커스텀 이벤트를 활용해 유저 코호트를 생성할 수도 있습니다.
커스텀 이벤트는 크게 비결제 이벤트와 결제 이벤트로 나뉩니다.

#### 비결제 이벤트 전송하기

비결제 이벤트는 게임 내 결제와 무관한 이벤트로, 이벤트 이름과 값을 인자로 받습니다.
이벤트 이름은 자유롭게 선언하여 사용하면 되며, 값은 String, double형 숫자, Map<String,Double> 타입의 객체를 할당할 수 있습니다.
다음은 비결제 이벤트를 전송하는 예제입니다.

```java
ValuePotion.getInstance().trackEvent("stage_clear", 3);
```

특별한 값이 존재하지 않는 이벤트인 경우 이벤트 이름만 넘기면 됩니다.
```java
ValuePotion.getInstance().trackEvent("login");
```

이벤트 값은 숫자 형태의 데이터인 경우에만 집계가 가능합니다.
또한 NSDictionary 객체를 이벤트 값으로 사용하는 경우, 각 value 역시 숫자 형태의 데이터만 존재해야 합니다.
다음 예제는 trackEvent(String, Double 또는 Map<String,Double> ) 메소드의 올바른 사용 예를 보여줍니다.

```java
// 올바른 예
ValuePotion.getInstance().trackEvent("item_wing_use",23);
ValuePotion.getInstance().trackEvent("error",400);

HashMap<String,Object> values = new HashMap<String,Double>();
values.put("stage", 23);
values.put("play_time", 87.2);
values.put("item_count", 3);
ValuePotion.getInstance().trackEvent("stage_clear", values);
```


#### 결제 이벤트 전송하기

결제 이벤트는 게임 내 구매(In App Billing/In App Purchase)가 발생했을 때 사용되는 이벤트입니다.

결제 이벤트를 전송하기 위해서는 trackPurchaseEvent() Method를 사용합니다.
기본적으로 eventName 과 revenueAmount, currency 의 3가지 인자(Arguments)가 필요한 Method와
IAP 캠페인을 통한 트래킹일 경우에 사용하는 productId, campaignId, contentId의 3개의 인자를 추가로 전송하는 6개 인자 Method가 있습니다. 

결제 이벤트를 전송하면 앱 별 매출 리포트 집계가 가능합니다.
다음은 결제 이벤트를 전송하는 예제입니다.

```java
ValuePotion.getInstance().trackPurchaseEvent("gold_purchase", 0.99, "USD");
ValuePotion.getInstance().trackPurchaseEvent("gold_purchase", 2500, "KRW");
```

IAP 캠페인의 interstitial로부터 구매가 발생한 경우, 추가 인자를 전달하면 더욱 상세한 리포트를 확인할 수 있습니다. (아이템/캠페인/소재 별 매출 리포트)
추가 인자는 총 3가지로, productId(구매 아이템의 id), campaignId(IAP 캠페인의 id), contentId(IAP 캠페인 소재의 id)입니다.
productId, campaignId, contentId 값은 "액션 관련 delegate" 항목의 OnRequestPurchase() 델리게이트 메소드를 통해 전달받게 됩니다.

```java
VPPurchase lastPurchase = null;
public void onRequestedPurchase(ValuePotion vp, String placement, VPPurchase purchase) {
    lastPurchase = purchase;

    // Purchase 인터스티셜을 클릭시 여기서 purchase 객체내의 정보를 이용하여 결제 프로세스를 시작합니다.  
}
...

// 결제 프로세스 종료 후
// Extra 로 제공되는 IAPUtil.java 를 이용하면 해당 productId로 가격과 통화단위를 쉽게 얻을 수 있습니다.

// InAppBillingService로부터 가격정보를 얻어옵니다.
String productId = lastPurchase.getProductId();
IAPItemDetail item = IAPUtil.getSkuDetail(context, mInAppBillingService, productId);

// 결제이벤트를 전송합니다.
ValuePotion.getInstance().trackPurchaseEvent("wing_purchase", item.getPriceAmount() , item.getPriceCurrencyCode(), lastPurchase);
```

만약 IAP 캠페인을 거치지 않은 경우에 productId 별로 데이터를 보내고 싶을때에는 
아래와 같이 productId를 직접 전달하면 됩니다.

```java
ValuePotion.getInstance().trackPurchaseEvent("wing_purchase", 1.99, "USD", "item_02_wing_2ea", null, null);
```

## Push Notification 연동

Push Notification API를 연동하면, 손쉽게 Push 타입의 캠페인을 생성하여 사용자에게 메시지를 전송할 수 있습니다.
또한, 사용자 Push 메시지를 클릭하여 게임을 실행한 경우 특정 캠페인의 interstitial을 노출시키도록 하는 것도 가능합니다.
Push Notification 연동을 위해서는 GCMIntentService 클래스에서 다음과 같이 구현합니다.

#### Android GCM

##### 수동연동

이미 GCM연동이 되어있는 경우에는, 안드로이드 플러그인 프로젝트에서 valuepotion.jar 와 valuepotionunity.jar를 클래스패스에 추가한 뒤, README.md 에 있는 GCM 관련 항목들은 AndroidManifest.xml 에 넣지 않습니다.

1. GCMIntentService 클래스에서 다음과 같이 구현합니다.

```java
@Override
protected void onHandleIntent(Intent intent) {
    Bundle extras = intent.getExtras();
    if (extras != null ){
        if (GoogleCloudMessaging.MESSAGE_TYPE_MESSAGE.equals(messageType)) {

            if( VPUnityBinder.gInstance.treatPushMessage(this, extras) ){
              // 밸류포션에서 발송한 GCM 처리성공
            }
            else{
                // 그외의 경우를 여기서 처리하시면 됩니다.
            }

        }
    }
}
```

