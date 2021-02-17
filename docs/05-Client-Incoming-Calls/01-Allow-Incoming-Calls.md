#  手順1: 外部からの着信を許可

この手順では外部からの着信を許可し、着信時にTwilio Clientを呼び出すロジックを実装します。

## 1-1: VoceGrantを更新し、着信を許可

ここまで作業を続けてきたFunctionの`Editor`を開き、`/token`パスを開きます。

`VoiceGrant`を設定している部分で着信を許可するように`incomingAllow`を`true`と変更します。

```js
exports.handler = function(context, event, callback) {

  const identity = 'user';
  
  const AccessToken = Twilio.jwt.AccessToken;
  const VoiceGrant = AccessToken.VoiceGrant;

  // 今回変更箇所
  // 着信にも対応するようにincomingAllowをtrueに設定
  const voiceGrant = new VoiceGrant({
      outgoingApplicationSid: context.TWIML_APP_SID,
      incomingAllow: true
  });

  const token = new AccessToken(
      context.ACCOUNT_SID,
      context.API_KEY,
      context.API_SECRET,
      { identity:  identity}
  );
  
  token.addGrant(voiceGrant);
  return callback(null, { token: token.toJwt()});
};
```

このアクセストークンを用いてTwilio Client側で着信に応答できます。

## 1-2: 着信時にTwilio Client側に通話を接続

続けて`/call`パスを開きます。外部発信用の番号がパラメータとして渡されていない場合は、着信だと判断し、`user`クライアントに通話を接続します。

```js

exports.handler = function(context, event, callback) {
  
  const VoiceResponse = Twilio.twiml.VoiceResponse;
  
  const voiceResponse = new VoiceResponse();
  
  const number = event.number;
  
  const dial = voiceResponse.dial({callerId: context.PHONE_NUMBER });

  // 今回変更箇所
  // 番号が渡されている時
  if (number) {
    dial.number(number);
  }
  // 番号が渡されていない時は着信と判断し、Clientに通話を接続
  else {
    dial.client("user");
  }

  return callback(null, voiceResponse);
};

```

----
ここで指定した`user`はアクセストークン生成時に`identity`として設定した値です。複数のclientを利用する場合は電話を着信させるための`identity`を別途保持し、指定する必要があります。

----

## 次の手順

[手順2: ブラウザフォンで着信に応答](02-Respond-Incoming-Calls.md)