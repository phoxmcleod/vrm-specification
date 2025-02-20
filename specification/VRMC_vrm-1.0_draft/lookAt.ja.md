# `VRMC_vrm.lookAt`

本文書では、 `VRMC_vrm` 拡張のうち `lookAt` フィールドについての仕様を示します。

| 名前                    | 備考                                                 |
|:------------------------|:-----------------------------------------------------|
| type                    | bone または expression                                  |
| offsetFromHeadBone      | lookAtの基準位置(両目の間が目安)へのヘッドボーンからの位置offsetです |
| rangeMapHorizontalInner | 水平内側の目の可動範囲                                 |
| rangeMapHorizontalOuter | 水平外側の目の可動範囲                                 |
| rangeMapVerticalDown    | 下方向の目の可動範囲                                   |
| rangeMapVerticalUp      | 上方向の目の可動範囲                                   |

### LookAtType

| 名前       | 備考                                                               |
|:-----------|:-------------------------------------------------------------------|
| bone       | Humanoid で規定された leftEyeボーンとrightEyeボーンで視線制御します                      |
| expression | Expression のLookAt, LookDown, LookLeft, LookRightで視線制御します |

### 目の可動範囲の調整

* rangeMapHorizontalInner
* rangeMapHorizontalOuter
* rangeMapVerticalDown
* rangeMapVerticalUp

の４つで目の可動範囲を設定できます。

LookAtType が expression の場合は、 LookLeft, LookRight に対して `rangeMapHorizontalOuter` の値を使用します。
LookLeft, LookRight が両目をまとめて可動させるため、片方に `Inner` 反対に `Outer` というように適用することが不可能であるためです。

#### 水平内側

`extensions.VRMC_vrm.lookAt.rangeMapHorizontalInner`

* 左目の右方向
* 右目の左方向
* boneタイプ: outputScale には leftEye, rightEye ボーンの Euler 角(radian) による最大回転角度を指定します
* expressionタイプ: outputScale には LookLeft, LookRight expression の最大適用量を指定します(最大1.0)

```
Y = clamp(yaw, 0, rangeMapHorizontalInner.inputMaxValue)/rangeMapHorizontalInner.inputMaxValue * rangeMapHorizontalInner.outputScale
```

`expression` タイプでは使用されません。水平方向は `rangeMapHorizontalOuter` が使われます。

#### 水平外側

`extensions.VRMC_vrm.lookAt.rangeMapHorizontalOuter`

* 左目の左方向
* 右目の右方向
* boneタイプ: outputScale には leftEye, rightEye ボーンの Euler 角(radian) による最大回転角度を指定します
* expressionタイプ: outputScale には LookLeft, LookRight expression の最大適用量を指定します(最大1.0)

```
Y = clamp(yaw, 0, rangeMapHorizontalOuter.inputMaxValue)/rangeMapHorizontalOuter.inputMaxValue * rangeMapHorizontalOuter.outputScale
```

`expression` タイプでは水平方向として使われます。

#### 垂直下方向

`extensions.VRMC_vrm.lookAt.rangeMapVerticalDown`

* 左目の下方向
* 右目の下方向
* boneタイプ: outputScale には leftEye, rightEye ボーンの Euler 角(radian) による最大回転角度を指定します
* expressionタイプ: outputScale には LookLeft, LookRight expression の最大適用量を指定します(最大1.0)

```
Y = clamp(yaw, 0, rangeMapVerticalDown.inputMaxValue)/rangeMapVerticalDown.inputMaxValue * rangeMapVerticalDown.outputScale 
```

#### 垂直上方向

`extensions.VRMC_vrm.lookAt.rangeMapVerticalUp`

* 左目の上方向
* 右目の上方向
* boneタイプ: outputScale には leftEye, rightEye ボーンの Euler 角(radian) による最大回転角度を指定します
* expressionタイプ: outputScale には LookLeft, LookRight Expression の最大適用量を指定します(最大1.0)

```
Y = clamp(yaw, 0, rangeMapVerticalUp.inputMaxValue)/rangeMapVerticalUp.inputMaxValue * rangeMapVerticalUp.outputScale
```

#### Boneタイプの可動範囲の適用

可動範囲調整後の Yaw, Pitch 角の値をオイラー角として leftEye, rightEye ボーンの LocalRotation に適用します。

#### Expressionタイプの可動範囲の適用

可動範囲調整後の Yaw, Pitch 角をExpressionのweight値として LookLeft, LookRight, LookDown, LookUp Expressionに適用します。

### LookAtのアルゴリズム

* HeadBoneのforwardベクトルと、HeadBone + firstPersonBoneOffset の位置から注視点を結んだベクトルが為す方向の Yaw Pitch 角を計算
* x, y = ClampAndScale(yaw, pitch)
* apply(x, y)

#### Boneタイプ

| bone と yaw, pitch               | 備考                                            |
|:--------------------------------|:----------------------------------------------|
| leftEye + yaw(左)               | rangeMapHorizontalOuter を適用してオイラー角として反映します |
| leftEye + yaw(右)               | rangeMapHorizontalInner を適用してオイラー角として反映します |
| rightEye + yaw(左)              | rangeMapHorizontalInner を適用してオイラー角として反映します |
| rightEye + yaw(右)              | rangeMapHorizontalOuter を適用してオイラー角として反映します |
| leftEye or rightEye + pitch(下) | rangeMapVerticalDown を適用してオイラー角として反映します    |
| leftEye or rightEye + pitch(上) | rangeMapVerticalUp を適用してオイラー角として反映します      |

#### Expressionタイプ

| bone と yaw, pitch               | 備考                                                               |
|:--------------------------------|:-----------------------------------------------------------------|
| leftEye + yaw(左)               | rangeMapHorizontalOuter を適用して Expression LookLeft の値として反映します  |
| leftEye + yaw(右)               | rangeMapHorizontalOuter を適用して Expression LookRight の値として反映します |
| rightEye + yaw(左)              | rangeMapHorizontalOuter を適用して Expression LookLeft の値として反映します  |
| rightEye + yaw(右)              | rangeMapHorizontalOuter を適用して Expression LookRight の値として反映します |
| leftEye or rightEye + pitch(下) | rangeMapVerticalDown を適用して Expression LookDown の値として反映します     |
| leftEye or rightEye + pitch(上) | rangeMapVerticalUp を適用して Expression LookUp の値として反映します         |

LookAtのExpressionタイプは、 MorphTarget タイプと TextureUVOffset タイプがありますが、ここでの処理は同じです。
