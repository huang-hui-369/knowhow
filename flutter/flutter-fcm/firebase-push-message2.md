#　FCMを利用して、送受信開発流れ

#　1．serverでメッセージ送信開発
    メッセージは、Firebase Admin SDK を介して送信する
    JavaバージョンのFirebase Admin SDK を利用して、アプリサーバーを開発する
    Admin FCM API 可处理后端身份验证工作，同时便于发送消息和管理主题订阅。
    使用 Firebase Admin SDK，您可以执行以下操作：
    - 向个别设备发送消息
    - 向主题和与一个或多个主题匹配的条件语句发送消息。
    - 为设备订阅和退订主题
    - 针对不同目标平台构建量身定制的消息负载。
    
* [api ドキュメント](https://firebase.google.com/docs/reference/admin/java/reference/com/google/firebase/messaging/package-summary?authuser=0 )
* [FCM HTTP v1 API プロトコル](https://firebase.google.com/docs/reference/fcm/rest/v1/projects.messages?authuser=0)

## 1.1 Java　Firebase Admin SDK　導入

1. dependency導入
``` java
<dependency>
  <groupId>com.google.firebase</groupId>
  <artifactId>firebase-admin</artifactId>
  <version>7.0.1</version>
</dependency>
```
2. JSON 格式的私钥文件的导入

![](img/2020-12-21-16-00-21.png)

3. 设置私钥文件环境变量
```
export GOOGLE_APPLICATION_CREDENTIALS="/home/user/Downloads/service-account-on-file.json"
```
完成上述步骤后，应用默认凭据 (ADC) 能够隐式确定您的凭据，如此您便能在非 Google 环境中测试或运行时使用服务帐号凭据

## 1.2 Java　授权访问

1. 初始化 SDK java 代码
```java
// 在不使用参数的情况下进行初始化，该SDK使用 Google 应用默认凭据
FirebaseApp.initializeApp();

```
或者 使用凭据来创建访问令牌
```java
private static String getAccessToken() throws IOException {
  GoogleCredential googleCredential = GoogleCredential
      .fromStream(new FileInputStream("service-account.json"))
      .createScoped(Arrays.asList(SCOPES));
  // 在您的访问令牌到期后，系统会自动调用令牌刷新方法以检索更新的访问令牌
  googleCredential.refreshToken();
  return googleCredential.getAccessToken();
}

```
2. 如需将访问令牌添加到 HTTP 请求标头中，请执行以下操作
```java
    URL url = new URL(BASE_URL + FCM_SEND_ENDPOINT);
    HttpURLConnection httpURLConnection = (HttpURLConnection) url.openConnection();
    httpURLConnection.setRequestProperty("Authorization", "Bearer " + getAccessToken());
    httpURLConnection.setRequestProperty("Content-Type", "application/json; UTF-8");
    return httpURLConnection;
}

```

3. 检查服务器密钥的有效性。例如，在 Linux 上运行以下命令：
```shell

api_key=YOUR_SERVER_KEY

curl --header "Authorization: key=$api_key" \
     --header Content-Type:"application/json" \
     https://fcm.googleapis.com/fcm/send \
     -d "{\"registration_ids\":[\"ABC\"]}"

```

## 1.3 Java　构建应用服务器发送请求
使用 Firebase Admin SDK 或 FCM 应用服务器协议，您可以构建消息请求并将其发送到以下各类目标：

* 主题名称
* 条件
* 设备注册令牌
* 设备组名称（仅限旧版协议或 Node.js 版 Firebase Admin SDK）
您可以发送包含通知载荷（由预定义字段组成）的消息、包含数据载荷（由您自己的用户定义字段组成）的消息，或者包含这两种载荷的消息。

### 1.3.1 向特定设备发送消息
  ```java
  // This registration token comes from the client FCM SDKs.
    String registrationToken = "YOUR_REGISTRATION_TOKEN";

    // See documentation on defining a message payload.
    Message message = Message.builder()
        .putData("score", "850")
        .putData("time", "2:45")
        .setToken(registrationToken)
        .build();

    // Send a message to the device corresponding to the provided
    // registration token.
    String response = FirebaseMessaging.getInstance().send(message);
    // Response is a message ID string.
    System.out.println("Successfully sent message: " + response);FirebaseMessagingSnippets.java
  ```

### 1.3.2 向多台设备发送消息
  借助 REST API 与 Admin FCM API，您可以将消息多播到一系列设备注册令牌。每次调用时，您最多可以指定 **100** 个设备注册令牌（**对于 Java 和 Node.js，最多可以指定 500 个**）。

  ```java
  // Create a list containing up to 500 registration tokens.
    // These registration tokens come from the client FCM SDKs.
    List<String> registrationTokens = Arrays.asList(
        "YOUR_REGISTRATION_TOKEN_1",
        // ...
        "YOUR_REGISTRATION_TOKEN_n"
    );

    MulticastMessage message = MulticastMessage.builder()
        .putData("score", "850")
        .putData("time", "2:45")
        .addAllTokens(registrationTokens)
        .build();
    BatchResponse response = FirebaseMessaging.getInstance().sendMulticast(message);
    // See the BatchResponse reference documentation
    // for the contents of response.
    System.out.println(response.getSuccessCount() + " messages were sent successfully");FirebaseMessagingSnippets.java
  ```

### 1.3.3 向主题发送消息
  借助 REST API 与 Admin FCM API，您可以将消息多播到一系列设备注册令牌。每次调用时，您最多可以指定 **100** 个设备注册令牌（**对于 Java 和 Node.js，最多可以指定 500 个**）。
  ```java
  // The topic name can be optionally prefixed with "/topics/".
  String topic = "highScores";

  // See documentation on defining a message payload.
  Message message = Message.builder()
      .putData("score", "850")
      .putData("time", "2:45")
      .setTopic(topic)
      .build();

  // Send a message to the devices subscribed to the provided topic.
  String response = FirebaseMessaging.getInstance().send(message);
  // Response is a message ID string.
  System.out.println("Successfully sent message: " + response);FirebaseMessagingSnippets.java
  ``` 
  **或者根据条件发送消息**
  如需向主题组合发送消息，请指定一个条件，该条件是一个指定目标主题的布尔表达式。例如，如需向已订阅 TopicA 和 TopicB 或 TopicC 的设备发送消息，请设置如下条件：
  `'TopicA' in topics && ('TopicB' in topics || 'TopicC' in topics)`
  FCM 首先对括号中的所有条件求值，然后从左至右对表达式求值。在上述表达式中，只订阅某个单一主题的用户将不会接收到消息。同样地，未订阅 TopicA 的用户也不会接收到消息。下列组合将会接收到消息：

  TopicA 和 TopicB
  TopicA 和 TopicC
  您最多可以在条件表达式中添加五个主题。
  ```java
  // Define a condition which will send to devices which are subscribed
  // to either the Google stock or the tech industry topics.
  String condition = "'stock-GOOG' in topics || 'industry-tech' in topics";

  // See documentation on defining a message payload.
  Message message = Message.builder()
      .setNotification(Notification.builder()
          .setTitle("$GOOG up 1.43% on the day")
          .setBody("$GOOG gained 11.80 points to close at 835.67, up 1.43% on the day.")
          .build())
      .setCondition(condition)
      .build();

  // Send a message to devices subscribed to the combination of topics
  // specified by the provided condition.
  String response = FirebaseMessaging.getInstance().send(message);
  // Response is a message ID string.
  System.out.println("Successfully sent message: " + response);FirebaseMessagingSnippets.java
  ``` 

## 1.4 服务器管理主题
借助 Firebase Admin SDK，您可以从服务器端执行基本的主题管理任务。如果有客户端应用实例的注册令牌，您还可以使用服务器逻辑批量订阅和退订这些实例。

您可以为客户端应用实例订阅任何现有主题，也可创建新主题。当您使用 API 为客户端应用订阅新主题（您的 Firebase 项目中尚不存在的主题）时，系统会在 FCM 中创建一个使用该名称的新主题，随后任何客户端都可订阅该主题。

您可以将注册令牌列表传递给 Firebase Admin SDK 订阅方法，以便为相应的设备订阅主题：
### 1.4.1 订阅主题
```java
// These registration tokens come from the client FCM SDKs.
List<String> registrationTokens = Arrays.asList(
    "YOUR_REGISTRATION_TOKEN_1",
    // ...
    "YOUR_REGISTRATION_TOKEN_n"
);

// Subscribe the devices corresponding to the registration tokens to the
// topic.
TopicManagementResponse response = FirebaseMessaging.getInstance().subscribeToTopic(
    registrationTokens, topic);
// See the TopicManagementResponse reference documentation
// for the contents of response.
System.out.println(response.getSuccessCount() + " tokens were subscribed successfully");
```

### 1.4.2 退订主题
```java
// These registration tokens come from the client FCM SDKs.
List<String> registrationTokens = Arrays.asList(
    "YOUR_REGISTRATION_TOKEN_1",
    // ...
    "YOUR_REGISTRATION_TOKEN_n"
);

// Unsubscribe the devices corresponding to the registration tokens from
// the topic.
TopicManagementResponse response = FirebaseMessaging.getInstance().unsubscribeFromTopic(
    registrationTokens, topic);
// See the TopicManagementResponse reference documentation
// for the contents of response.
System.out.println(response.getSuccessCount() + " tokens were unsubscribed successfully");
```


##　２．クライアントの受信開発

## 1. 個々のデバイスに

## 2. デバイス グループに

## 3. 特定トピックの配信登録をしているデバイスに

##　3．送信テスト
     Notifications Composer を使うと、送信テストをする。

# 完整的发送message的例子

```java
package com.cy.core;

import java.io.DataOutputStream;
import java.io.FileInputStream;
import java.io.IOException;
import java.io.InputStream;
import java.net.HttpURLConnection;
import java.net.URL;
import java.util.ArrayList;
import java.util.Arrays;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.Scanner;
import java.util.UUID;

import org.jsoup.Jsoup;
import org.jsoup.nodes.Document;
import org.jsoup.nodes.Element;
import org.jsoup.select.Elements;
import org.springframework.core.io.ClassPathResource;

import com.cy.common.exception.BussinessException;
import com.cy.pojo.TNoticePojo;
import com.google.api.client.googleapis.auth.oauth2.GoogleCredential;
import com.google.auth.oauth2.GoogleCredentials;
import com.google.firebase.FirebaseApp;
import com.google.firebase.FirebaseOptions;
import com.google.firebase.messaging.AndroidConfig;
import com.google.firebase.messaging.BatchResponse;
import com.google.firebase.messaging.FirebaseMessaging;
import com.google.firebase.messaging.FirebaseMessagingException;
import com.google.firebase.messaging.MulticastMessage;
import com.google.firebase.messaging.Notification;
import com.google.firebase.messaging.SendResponse;
import com.google.gson.Gson;
import com.google.gson.GsonBuilder;
import com.google.gson.JsonObject;

public class GoogleMessaging {
    private static final String PROJECT_ID = "futari-passport-metro-tokyo";
    private static final String BASE_URL = "https://fcm.googleapis.com";
    private static final String FCM_SEND_ENDPOINT = "/v1/projects/" + PROJECT_ID + "/messages:send";

    private static final String MESSAGING_SCOPE = "https://www.googleapis.com/auth/firebase.messaging";
    private static final String[] SCOPES = { MESSAGING_SCOPE };

    private static final String TITLE = "FCM Notification";
    private static final String BODY = "Notification from FCM";
    public static final String MESSAGE_KEY = "message";

    /**
     * Retrieve a valid access token that can be use to authorize requests to the FCM REST
     * API.
     *
     * @return Access token.
     * @throws IOException
     */
    // [START retrieve_access_token]
    private static String getAccessToken() throws IOException {
        GoogleCredential googleCredential = GoogleCredential
                .fromStream(new FileInputStream("service-account.json"))
                .createScoped(Arrays.asList(SCOPES));
        googleCredential.refreshToken();
        return googleCredential.getAccessToken();
    }
    // [END retrieve_access_token]

    /**
     * Create HttpURLConnection that can be used for both retrieving and publishing.
     *
     * @return Base HttpURLConnection.
     * @throws IOException
     */
    private static HttpURLConnection getConnection() throws IOException {
        // [START use_access_token]
        URL url = new URL(BASE_URL + FCM_SEND_ENDPOINT);
        HttpURLConnection httpURLConnection = (HttpURLConnection) url.openConnection();
        httpURLConnection.setRequestProperty("Authorization", "Bearer " + getAccessToken());
        httpURLConnection.setRequestProperty("Content-Type", "application/json; UTF-8");
        return httpURLConnection;
        // [END use_access_token]
    }

    /**
     * Send request to FCM message using HTTP.
     *
     * @param fcmMessage Body of the HTTP request.
     * @throws IOException
     */
    private static void sendMessage(JsonObject fcmMessage) throws IOException {
        HttpURLConnection connection = getConnection();
        connection.setDoOutput(true);
        DataOutputStream outputStream = new DataOutputStream(connection.getOutputStream());
        outputStream.writeBytes(fcmMessage.toString());
        outputStream.flush();
        outputStream.close();

        int responseCode = connection.getResponseCode();
        if (responseCode == 200) {
            String response = inputstreamToString(connection.getInputStream());
            System.out.println("Message sent to Firebase for delivery, response:");
            System.out.println(response);
        } else {
            System.out.println("Unable to send message to Firebase:");
            String response = inputstreamToString(connection.getErrorStream());
            System.out.println(response);
        }
    }

    /**
     * Send a message that uses the common FCM fields to send a notification message to all
     * platforms. Also platform specific overrides are used to customize how the message is
     * received on Android and iOS.
     *
     * @throws IOException
     */
    private static void sendOverrideMessage() throws IOException {
        JsonObject overrideMessage = buildOverrideMessage();
        System.out.println("FCM request body for override message:");
        prettyPrint(overrideMessage);
        sendMessage(overrideMessage);
    }

    /**
     * Build the body of an FCM request. This body defines the common notification object
     * as well as platform specific customizations using the android and apns objects.
     *
     * @return JSON representation of the FCM request body.
     */
    private static JsonObject buildOverrideMessage() {
        JsonObject jNotificationMessage = buildNotificationMessage();

        JsonObject messagePayload = jNotificationMessage.get(MESSAGE_KEY).getAsJsonObject();
        messagePayload.add("android", buildAndroidOverridePayload());

        JsonObject apnsPayload = new JsonObject();
        apnsPayload.add("headers", buildApnsHeadersOverridePayload());
        apnsPayload.add("payload", buildApsOverridePayload());

        messagePayload.add("apns", apnsPayload);

        jNotificationMessage.add(MESSAGE_KEY, messagePayload);

        return jNotificationMessage;
    }

    /**
     * Build the android payload that will customize how a message is received on Android.
     *
     * @return android payload of an FCM request.
     */
    private static JsonObject buildAndroidOverridePayload() {
        JsonObject androidNotification = new JsonObject();
        androidNotification.addProperty("click_action", "android.intent.action.MAIN");

        JsonObject androidNotificationPayload = new JsonObject();
        androidNotificationPayload.add("notification", androidNotification);

        return androidNotificationPayload;
    }

    /**
     * Build the apns payload that will customize how a message is received on iOS.
     *
     * @return apns payload of an FCM request.
     */
    private static JsonObject buildApnsHeadersOverridePayload() {
        JsonObject apnsHeaders = new JsonObject();
        apnsHeaders.addProperty("apns-priority", "10");

        return apnsHeaders;
    }

    /**
     * Build aps payload that will add a badge field to the message being sent to
     * iOS devices.
     *
     * @return JSON object with aps payload defined.
     */
    private static JsonObject buildApsOverridePayload() {
        JsonObject badgePayload = new JsonObject();
        badgePayload.addProperty("badge", 1);

        JsonObject apsPayload = new JsonObject();
        apsPayload.add("aps", badgePayload);

        return apsPayload;
    }

    /**
     * Send notification message to FCM for delivery to registered devices.
     *
     * @throws IOException
     */
    public static void sendCommonMessage() throws IOException {
        JsonObject notificationMessage = buildNotificationMessage();
        System.out.println("FCM request body for message using common notification object:");
        prettyPrint(notificationMessage);
        sendMessage(notificationMessage);
    }

    /**
     * Construct the body of a notification message request.
     *
     * @return JSON of notification message.
     */
    private static JsonObject buildNotificationMessage() {
        JsonObject jNotification = new JsonObject();
        jNotification.addProperty("title", TITLE);
        jNotification.addProperty("body", BODY);

        JsonObject jMessage = new JsonObject();
        jMessage.add("notification", jNotification);
        jMessage.addProperty("topic", "news");

        JsonObject jFcm = new JsonObject();
        jFcm.add(MESSAGE_KEY, jMessage);

        return jFcm;
    }

    /**
     * Read contents of InputStream into String.
     *
     * @param inputStream InputStream to read.
     * @return String containing contents of InputStream.
     * @throws IOException
     */
    private static String inputstreamToString(InputStream inputStream) throws IOException {
        StringBuilder stringBuilder = new StringBuilder();
        Scanner scanner = new Scanner(inputStream);
        while (scanner.hasNext()) {
            stringBuilder.append(scanner.nextLine());
        }
        return stringBuilder.toString();
    }

    /**
     * Pretty print a JsonObject.
     *
     * @param jsonObject JsonObject to pretty print.
     */
    private static void prettyPrint(JsonObject jsonObject) {
        Gson gson = new GsonBuilder().setPrettyPrinting().create();
        System.out.println(gson.toJson(jsonObject) + "\n");
    }

    public static void push( List<String> registrationTokens,TNoticePojo pojo) throws BussinessException {

        Map<String,List<String>> map = new HashMap<String,List<String>>();
        int listSize=registrationTokens.size();
        if (listSize == 0) return;
        int size = 499;
        int toIndex=size;
        int keyToken = 0;
        for(int i = 0;i<registrationTokens.size();i+=size){
            if(i+size>listSize){
                toIndex=listSize-i;
            }
            List newList = registrationTokens.subList(i,i+toIndex);
            map.put("keyName"+keyToken, newList);
            keyToken++;
        }

        try{
            ClassPathResource classPathResource = new ClassPathResource("futari-passport-metro-tokyo-firebase-adminsdk-o83bl-0a349140fd.json");

            FirebaseOptions options = new FirebaseOptions.Builder()
                    .setCredentials(GoogleCredentials.fromStream(classPathResource.getInputStream()))
                    .build();

            FirebaseMessaging firebaseMessaging = FirebaseMessaging.getInstance(FirebaseApp.initializeApp(options,"futari-passport-metro-tokyo-firebase" + UUID.randomUUID().toString().replaceAll("-","")));

            for(List<String> tokens:map.values()){
                BatchResponse response = firebaseMessaging.sendMulticast(
                        MulticastMessage.builder()
                                // 手机sdk自动处理的数据，不支持html，显示在通知面板
                                .setNotification(Notification.builder()
                                        .setTitle(pojo.getTitle())
                                        .setBody(pojo.getInfo())
                                        .build())
                                // 没有 click_action android手机不会调用onLanch，onResume()事件方法       
                                .setAndroidConfig(AndroidConfig.builder()
                                		.putData("click_action", "FLUTTER_NOTIFICATION_CLICK").build())
//                                .putData("id", pojo.getId().toString())
                                // 自己手机APP程序处理的数据
                                .putData("title", pojo.getTitle())
                                .putData("info", pojo.getInfo())
                                // 发送目的手机的token
                                .addAllTokens(tokens)
                                .build());

                if (response.getFailureCount() > 0) {
                    List<SendResponse> responses = response.getResponses();
                    List<String> failedTokens = new ArrayList<>();
                    for (int i = 0; i < responses.size(); i++) {
                        if (!responses.get(i).isSuccessful()) {
                            failedTokens.add(registrationTokens.get(i));
                        }
                    }

                    System.out.println("List of tokens that caused failures: " + failedTokens);
                }
            }

        }catch (IOException io){
            BussinessException ex =new BussinessException();
            ex.error("E_0036");
            throw ex;
        }catch (FirebaseMessagingException firebase){
            BussinessException ex =new BussinessException();
            ex.error("E_0036");
            throw ex;
        }

    }
    private static String getStackMsg(Exception e) {

        StringBuffer sb = new StringBuffer();
        StackTraceElement[] stackArray = e.getStackTrace();
        for (int i = 0; i < stackArray.length; i++) {
            StackTraceElement element = stackArray[i];
            sb.append(element.toString() + "\n");
        }
        return sb.toString();
    }
   

//    public static void main(String[] args) throws IOException, FirebaseMessagingException {
//        List<String> registrationTokens = Arrays.asList(
//                "fKNOjYFjRNaQIMKp1Tn8cP:APA91bGkfPgsitmdBurkVvMolgvH_drah3LONS3tHkp-wEpm0KKG5RCXV8xJ9fklvPu6FDLTK46TJ-mLmhxX9g1d0hgyEQP65P_gBIR0DMjsAMxO7l4794IZikzkR_6PUxUkLWx6rpa6"
//        );
//
//        MulticastMessage message = MulticastMessage.builder()
//                .setNotification(Notification.builder()
//                        .setTitle("test title")
//                        .setBody("test body")
//                        .build())
//                .addAllTokens(registrationTokens)
//                .build();
//        FileInputStream serviceAccount = new FileInputStream(ResourceUtils.getFile("classpath:futari-passport-metro-tokyo-firebase-adminsdk-o83bl-0a349140fd.json"));
//
//        FirebaseOptions options = new FirebaseOptions.Builder()
//                .setCredentials(GoogleCredentials.fromStream(serviceAccount))
//                .build();
//        BatchResponse response = FirebaseMessaging.getInstance(FirebaseApp.initializeApp(options))
//                .sendMulticast(message);
//        if (response.getFailureCount() > 0) {
//            List<SendResponse> responses = response.getResponses();
//            List<String> failedTokens = new ArrayList<>();
//            for (int i = 0; i < responses.size(); i++) {
//                if (!responses.get(i).isSuccessful()) {
//                    // The order of responses corresponds to the order of the registration tokens.
//                    failedTokens.add(registrationTokens.get(i));
//                }
//            }
//
//            System.out.println("List of tokens that caused failures: " + failedTokens);
//        }
//    }
}
```

