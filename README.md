## SAML (Security Assertion Markup Language) 2.0 

ArcGIS Online / ArcGIS Enterprise は、エンタープライズ ログインのアカウント構成に SAML (Security Assertion Markup Language) 2.0 をサポートしています。SAML は、認証サーバーである ID プロバイダーとサービスを提供するアプリケーションであるサービス プロバイダーとの間で認証/認可データを安全に交換するためのオープン規格です。

ArcGIS Online / ArcGIS Enterprise は SAML 2.0 に準拠しており、この規格に準拠する ID プロバイダー (Idp) と統合することができます。

- Shibboleth
  - Shibboleth 3.2x および 3.3x は SAML 2.0 に準拠しているので、Idp として ArcGIS Online / ArcGIS Enterprise で使用することができます。
  - [Shibboleth](/Shibboleth) には、連携に利用する SAML の各設定ファイルを公開しています。ArcGIS Online / ArcGIS Enterprise と Shibboleth の連携手順については、「[ArcGIS Online/ArcGIS Enterprise - Shibboleth 連携ガイド](/Shibboleth/Shibboleth_SetupGuide.md)」をご参照ください。

- Active Directory Federation Service (ADFS)
  - Active Directory Federation Service (ADFS) は SAML 2.0 に準拠しているので、Idp として ArcGIS Online / ArcGIS Enterprise で使用することができます。
  - ArcGIS Online / ArcGIS Enterprise と Active Directory Federation Service (ADFS) の連携手順については、「[ArcGIS Online/ArcGIS Enterprise - Active Directory Federation Service 連携ガイド](/ADFS/ADFS_SetupGuide.md)」をご参照ください。

## ライセンス

Copyright 2020 Esri Japan Corporation.

Apache License Version 2.0（「本ライセンス」）に基づいてライセンスされます。あなたがこのファイルを使用するためには、本ライセンスに従わなければなりません。
本ライセンスのコピーは下記の場所から入手できます。

> http://www.apache.org/licenses/LICENSE-2.0

適用される法律または書面での同意によって命じられない限り、本ライセンスに基づいて頒布されるソフトウェアは、明示黙示を問わず、いかなる保証も条件もなしに「現状のまま」頒布されます。本ライセンスでの権利と制限を規定した文言については、本ライセンスを参照してください。

ライセンスのコピーは本リポジトリの[ライセンス ファイル](./LICENSE)で利用可能です。
