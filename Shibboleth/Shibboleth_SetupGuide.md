# ArcGIS Online / ArcGIS Enterprise Shibboleth 連携ガイド

## 1. はじめに

### 1.1 本ガイドについて

Shibboleth は、組織内や組織間などの利用時に Web 上でシングルサインオン（SSO）を実現する標準的なオープンソース ソフトウェアのパッケージであり、Web アプリケーションやクラウド サービスへの SSO を実現します。国内では学術認証フェデレーション「学認」を実現する仕組みとして、広く利用されています。

本ガイドでは、Shibboleth を用いた ArcGIS Online への SSO が可能になるまでの手順について紹介します。また、ArcGIS Enterprise でも Shibboleth を用いた SSO を実現することができます。ArcGIS Enterprise と Shibboleth との連携においても、本ガイドで記載している手順で行うことができますので、連携の際は本ガイドの手順をご参照ください。
<br>
※ Shibboleth 以外を使用した ID プロバイダー（Idp）については、ArcGIS Online 公式ガイドの [SAML IDP](https://doc.arcgis.com/ja/arcgis-online/administer/saml-logins.htm#ESRI_SECTION1_E666FC046A1746C6B25F5EBF3058B992)を参照してください。

ArcGIS Online は、エンタープライズ ログインのアカウント構成に SAML (Security Assertion Markup Language) 2.0 をサポートしています。
<br>
SAML は、認証サーバーである ID プロバイダー (Idp：本ガイドでは Shibboleth が該当) とサービスを提供するアプリケーションであるサービス プロバイダー (SP：本ガイドでは ArcGIS Online が該当) との間で認証/認可データを安全に交換するためのオープン規格です。
<br>
ArcGIS Online は SAML 2.0 に準拠しており、この規格に準拠する Idp と統合することができます。
<br>
本ガイドで紹介する Shibboleth 3.2x および 3.3x は SAML 2.0 に準拠しているので、Idp として ArcGIS Online で使用することができます。
<br>
エンタープライズ ログインを構成することで、組織サイトのメンバーは、エンタープライズ システムにアクセスするときと同一のログイン情報 (1 つの ID・パスワード) を使用して ArcGIS Online に SSO をすることが可能となります。
<br>
また、ID・パスワードの再入力を行わずに利用できる環境を実現することができます。
<br>

ユーザーが ArcGIS Online にアクセスして SSO するまでの流れを下記に示します。  
<img width="500" alt="" src="assets\shibboleth01.png">

1. ユーザーが ArcGIS Online にアクセスします。
2. ArcGIS Online は認証要求と共にリクエストを Shibboleth へリダイレクトします。
3. ユーザーは Shibboleth へアクセスし、ユーザー情報を Shibboleth に送信します。
4. Shibboleth は認証要求を受けると、ユーザーやコンピューターの情報を集中管理する「ディレクトリ サービス」LDAP (Lightweight Directory Access Protocol) へユーザー情報を送信します。
5. LDAP はユーザーを認証します。
6. ユーザー認証を Shibboleth へ通知します。
7. ユーザーが認証されたことを Shibboleth が確認した後、Shibboleth はアサーション（認証応答）を発行し、ユーザーに送信します。
8. ユーザーは Shibboleth から送られたアサーションとともに、ArcGIS Online へリダイレクトします。
9. ArcGIS Online はアサーションを確認した後、ユーザーをログインさせます。

### 1.2 環境情報

本ガイドで使用する Shibboleth および ArcGIS Online の環境情報を以下に記載します。
<br>
本ガイドで使用する Shibboleth と LDAP は同一のマシンに構成しています。
<br>
なお、連携の手順では、ArcGIS Online へのアクセスおよび ArcGIS Online 上での設定に加えて、Shibboleth への設定も行いますが、本ガイドではこれらの設定は Shibboleth をインストールしている端末（CentOS）で行います。Shibboleth への設定では、root ユーザーとしてログインしている状態で手順を進めます。
<br>

<table>
	<tbody>
		<tr>
			<td colspan="2">環境情報</td>
			<td>内容</td>
			<td>備考</td>
		</tr>
		<tr>
			<td colspan="2">Shibboleth をインストールしているマシンの OS</td>
			<td>CentOS 7.0</td>
			<td>Shibboleth のバージョン：3.4.4</td>
		</tr>
		<tr>
			<td colspan="2">ArcGIS Online の組織サイト名</td>
			<td>nk226.maps.arcgis.com</td>
			<td>2020 年 7 月時点のアップデートのものを使用</td>
		</tr>
		<tr>
			<td rowspan="3">LDAP の情報</td>
			<td>FQDN</td>
			<td>encent1061.esrij.com</td>
			<td></td>
		</tr>
		<tr>
			<td>baseDN</td>
			<td>dc=abc,dc=def,dc=com</td>
			<td></td>
		</tr>
		<tr>
			<td>bindDN</td>
			<td>cn=Manager,dc=abc,dc=def,dc=com</td>
			<td></td>
		</tr>
	</tbody>
</table>

## 2. ArcGIS Online と Shibboleth の連携設定

本章では、ArcGIS Online と Shibboleth を連携させるための設定の手順について説明します。本ガイドで記載している手順は、Shibboleth をインストールした端末上で行います。連携までの手順は以下になります。
<br>
なお、以下に記載している ArcGIS Online の設定画面は 2020 年 7 月時点での画面であり、今後のアップデートで変更される可能性があります。

- [2.1 ArcGIS Online に Shibboleth を Idp として登録](#21-arcgis-online-に-shibboleth-を-idp-として登録)
- [2.2 Shibboleth に ArcGIS Online を SP として登録](#22-shibboleth-に-arcgis-online-を-sp-として登録)

### 2.1 ArcGIS Online に Shibboleth を Idp として登録

Shibboleth をエンタープライズ Idp として ArcGIS Online に登録します。
<br>

1. Shibboleth のメタデータのファイル内容を確認します。メタデータは、\<Shibboleth インストールディレクトリ>/metadata/ にある idp-metadata.xml ファイルです。  
（Github に Shibboleth メタデータの[サンプル ファイル](https://github.com/EsriJapan/arcgis-saml-samples/blob/master/Shibboleth/idp-metadata.xml)をアップロードしているので、設定の際は、こちらも併せてご参照ください。）

   1. Idp-metadata.xml ファイルを開き、以下に記載する Name Space の定義がファイルに記載されていることを確認します。

      ```xml
      <EntityDescriptor  xmlns="urn:oasis:names:tc:SAML:2.0:metadata" xmlns:ds="http://www.w3.org/2000/09/xmldsig#" xmlns:shibmd="urn:mace:shibboleth:metadata:1.0" xmlns:xml="http://www.w3.org/XML/1998/namespace" xmlns:mdui="urn:oasis:names:tc:SAML:metadata:ui" xmlns:req-attr="urn:oasis:names:tc:SAML:protocol:ext:req-attr" validUntil="2019-07-16T07:55:59.469Z" entityID="https://encent1061.esrij.com/idp/shibboleth">
      ```

   2. メタデータ内の証明書情報を確認します。証明書情報はメタデータの \<KeyDescriptor use="signing"> タグと、\<KeyDescriptor use="encryption"> タグに記載されています。

      1. \<KeyDescriptor use="signing"> タグに記載されている証明書情報が \<Shibboleth インストールディレクトリ>/conf/idp.properties の [idp.signing.cert] で設定されている証明書であることを確認します（\<KeyDescriptor use="signing"> タグはメタデータ内に 4 か所あります）。

         - idp-metadata.xml

           ```xml
            <KeyDescriptor use="signing">
               <ds:KeyInfo>
                  <ds:X509Data>
                     <ds:X509Certificate>
                        ～証明書情報の文字列～
                     </ds:X509Certificate>
                	</ds:X509Data>
               </ds:KeyInfo>
            </KeyDescriptor>
           ```

         - idp.properties

           ```
           idp.signing.cert=%{idp.home}/credentials/idp-signing.crt
           ```

      2. \<KeyDescriptor use="encryption"> タグに記載されている証明書情報が \<Shibboleth インストールディレクトリ>/conf/idp.properties の [idp.encryption.cert] で設定されている証明書であることを確認します（\< KeyDescriptor use="encryption"> タグはメタデータ内に 2 か所あります）。

         - idp-metadata.xml

           ```xml
           <KeyDescriptor use="encryption">
               <ds:KeyInfo>
                  <ds:X509Data>
                     <ds:X509Certificate>
                        ～証明書情報の文字列～
                     </ds:X509Certificate>
                  </ds:X509Data>
               </ds:KeyInfo>
            </KeyDescriptor>
           ```

         - idp.properties

           ```
           idp.encryption.cert=%{idp.home}/credentials/idp-encryption.crt
           ```

2. ArcGIS Online で管理者権限のあるユーザーでログイン後、[組織] → [設定] → [セキュリティ] を選択します。

3. [ログイン] セクションで、[SAML ログインの設定] オプションを選択します。  
   <img width="500" alt="" src="assets\shibboleth02.png">

4. [1 つの ID プロバイダー] を選択します。  
   <img width="500" alt="" src="assets\shibboleth03.png">

5. [ID プロバイダーの設定] ウィンドウが表示されます。このウィンドウでは、任意の名前を入力し、ユーザーが [自動] または [管理者から招待されたとき] のどちらで組織に加入できるかを選択します（入力した名前は、SAML ログイン ボタンに表示されます）。  
   メタデータは、手順 1 で確認した \<Shibboleth インストールディレクトリ>/metadata/idp-metadata.xml ファイルを選択します。[高度な設定を表示] についてはデフォルト設定です。こちらの ID プロバイダーの設定は、あとで編集も可能です。  
   内容に問題がなければ IP プロバイダーの設定をクリックして設定を反映します。  
    <img width="500" alt="" src="assets\shibboleth04.png">

6. 設定反映後は以下のような画面になります。設定を変更する場合は、編集ボタン（下図赤枠の鉛筆ボタン）をクリックします。  
   <img width="500" alt="" src="assets\shibboleth05.png">

7. 編集ボタンをクリックすると次のような ID プロバイダーの編集画面が表示されますので、必要に応じて設定を行ってください。ログインと同時にユーザーを ArcGIS Online の組織サイトに加入させる場合は、[ユーザーは次の条件で加入できます] 下の [自動] を選択します（本ガイドでは [自動] と設定しました）。  
   <img width="500" alt="" src="assets\shibboleth06.png">

   また、ID プロバイダーの編集画面では、ArcGIS Online のエンティティ ID を確認することができます（編集画面の [高度な設定を表示] をクリックすると表示されます）。Shibboleth に ArcGIS Online を SP として登録する手順では、ArcGIS Online のエンティティ ID が必要です。入力の際はこちらの画面に表示されているエンティティ ID をご入力ください。  
   <img width="500" alt="" src="assets\shibboleth07.png">

<!--
以下の表はお客様のエンティティ ID のメモ用にお使いください。
<table>
	<tbody>
		<tr>
			<td>エンティティ ID</td>
			<td>　　　　　　　　　　　　　　　　　　　　</td>
		</tr>
	</tbody>
</table>
-->

### 2.2 Shibboleth に ArcGIS Online を SP として登録

ArcGIS Online を信頼できるサービス プロバイダー（SP）として Shibboleth に登録します。登録にあたって、Shibboleth IDP に格納されている以下の 6 つのファイルを編集します。なお、本手順で使用する６つのファイルについては、ESRI ジャパンの [GitHub](https://github.com/EsriJapan/arcgis-saml-samples) にも公開していますので、ご参照ください。

* [metadata-providers.xml](https://github.com/EsriJapan/arcgis-saml-samples/blob/master/Shibboleth/metadata-providers.xml)：ArcGIS Online を信頼できるサービス プロバイダーとして Shibboleth に追加するファイルです。
* [ldap.properties](https://github.com/EsriJapan/arcgis-saml-samples/blob/master/Shibboleth/ldap.properties)：Shibboleth が使用する LDAP に関する設定を行うファイルです。
* [attribute-resolver.xml](https://github.com/EsriJapan/arcgis-saml-samples/blob/master/Shibboleth/attribute-resolver.xml)：LDAP から Shibboleth IDP に送信されるユーザーの属性情報の設定を行うファイルです。
* [attribute-filter.xml](https://github.com/EsriJapan/arcgis-saml-samples/blob/master/Shibboleth/attribute-filter.xml)：attribute-resolver で設定した属性情報のうち、Shibboleth から ArcGIS Online に送信する属性の設定を行うファイルです。
* [relying-party.xml](https://github.com/EsriJapan/arcgis-saml-samples/blob/master/Shibboleth/relying-party.xml)：アサーション（認証応答）に関する設定を行うファイルです。
* [saml-nameid.xml](https://github.com/EsriJapan/arcgis-saml-samples/blob/master/Shibboleth/saml-nameid.xml)：Shibboleth IDP と ArcGIS Online の間で共有する NameID を定義するファイルです。

以下に、Shibboleth に ArcGIS Online を SP として登録する手順を記載します。なお、本手順の「Shibboleth への登録」は Shibboleth をインストールした端末上で行います。

1.  ArcGIS Online を信頼できる SP として Shibboleth に登録します。

    1. ArcGIS Online で管理者権限のあるユーザーでログイン後、[組織] → [設定] → [セキュリティ] をクリックします。

    2. [ログイン] → [SAML ログイン] の [サービス プロバイダーのメタデータのダウンロード] をクリックし、 ArcGIS Online のメタデータを保存します。（本ガイドでは、メタデータを nk226_sp_metadata.xml として、\<Shibboleth インストール ディレクトリ>/conf/ に配置しています）。  
       <img width="500" alt="" src="assets\shibboleth08.png">

    3. \<Shibboleth インストール ディレクトリ>/conf/metadata-providers.xml ファイルで、MetadataProvider エレメント内に次のスニペットを追加し、ダウンロードした nk226_sp_metadata.xml のパスを指定します。

       ```xml
       <MetadataProvider xmlns="urn:mace:shibboleth:2.0:metadata"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" id="EsriSP"
       xsi:type="FilesystemMetadataProvider" xsi:schemaLocation="urn:mace:shibboleth:2.0:metadata
       http://shibboleth.net/schema/idp/shibboleth-metadata.xsd" failFastInitialization="true"
       metadataFile="/opt/shibboleth-idp/conf/nk226_sp_metadata.xml"/>
       ```

2.  Shibboleth が使用する LDAP に関する設定（LDAP サーバーの URL、baseDN の設定など）を行います。
    LDAP の設定は \<Shibboleth インストールディレクトリ>/conf/ldap.properties を使用します。
    本ガイドでは、ldap.properties ファイルに対して以下のような設定で構成しました（編集部分をコードブロックで記載しています）。

    * idp.authn.LDAP.ldapURL：LDAP サーバーの URL  
      （本ガイドでは ldaps://encent1061.esrij.com）
    * idp.authn.LDAP.baseDN：LDAP サーバーの baseDN  
      （以下に記載されている baseDN は本ガイドで使用する baseDN（dc=abc,dc=def,dc=com）です）
    * idp.authn.LDAP.bindDN：LDAP サーバーの bindDN  
      （以下に記載されている bindDN は本ガイドで使用する bindDN（cn=Manager,dc=abc,dc=def,dc=com）です）
    * idp.authn.LDAP.bindDNCredential：bindDN のパスワード  
      （本ガイドでは manager)

      ```
      idp.authn.LDAP.ldapURL            = ldaps://encent1061.esrij.com
      idp.authn.LDAP.useStartTLS        = false   ## コメントアウトを解除し、false と設定

      idp.authn.LDAP.baseDN             = dc=abc,dc=def,dc=com

      idp.authn.LDAP.bindDN             = cn=Manager,dc=abc,dc=def,dc=com
      idp.authn.LDAP.bindDNCredential   = manager
      ```

3.  LDAP から Shibboleth IDP に送信されるユーザーの属性情報の設定を attribute-resolver.xml ファイルを使用して行います。attribute-resolver.xml は \<Shibboleth インストール ディレクトリ>/conf/ に配置されています。

    1.  \<Shibboleth インストール ディレクトリ>/conf/attribute-resolver.xml ファイル内の既存のすべてのサンプル属性定義とデータ コネクタにコメントを付けるか、削除します。

    2.  \<Shibboleth インストール ディレクトリ>/conf/attribute-resolver-full.xml ファイルに記載されている mail 属性（sourceAttributeID = "mail" の部分）および givenName 属性（sourceAttributeID = "givenName" の部分）の定義を attribute-resolver.xml ファイルにコピーします。

        ```xml
        <AttributeDefinition xsi:type="Simple" id="mail" sourceAttributeID="mail">
        	<Dependency ref="myLDAP" />
        	<AttributeEncoder xsi:type="SAML1String" name="urn:mace:dir:attribute-def:mail" encodeType="false" />
        	<AttributeEncoder xsi:type="SAML2String" name="urn:oid:0.9.2342.19200300.100.1.3" friendlyName="mail" encodeType="false" />
        </AttributeDefinition>

        <AttributeDefinition xsi:type="Simple" id="givenName" sourceAttributeID="givenName">
        	<Dependency ref="myLDAP" />
        	<AttributeEncoder xsi:type="SAML1String" name="urn:mace:dir:attribute-def:givenName" encodeType="false" />
        	<AttributeEncoder xsi:type="SAML2String" name="urn:oid:2.5.4.42" friendlyName="givenName" encodeType="false" />
        </AttributeDefinition>
        ```

    3.  \<Shibboleth インストール ディレクトリ>/conf/attribute-resolver-ldap.xml に記載されている、id = "myLDAP" の LDAP データ コネクタ セクションを attribute-resolver.xml にコピーします。
        また、NAMEID 属性として返す uid の属性の定義（sourceAttributeID ="uid" の部分）をコピーします。
        - LDAP データ コネクタ セクションの部分

            ```xml
           	<DataConnector id="myLDAP" xsi:type="LDAPDirectory"
           	   ldapURL="%{idp.attribute.resolver.LDAP.ldapURL}"
           		baseDN="%{idp.attribute.resolver.LDAP.baseDN}"
           		principal="%{idp.attribute.resolver.LDAP.bindDN}"
           		principalCredential="%{idp.attribute.resolver.LDAP.bindDNCredential}"
           		useStartTLS="%{idp.attribute.resolver.LDAP.useStartTLS:true}"
           		connectTimeout="%{idp.attribute.resolver.LDAP.connectTimeout}"
           		trustFile="%{idp.attribute.resolver.LDAP.trustCertificates}"
           		responseTimeout="%{idp.attribute.resolver.LDAP.responseTimeout}">
           		<FilterTemplate>
           			<![CDATA[
           				%{idp.attribute.resolver.LDAP.searchFilter}
           			]]>
           		</FilterTemplate>
           	</DataConnector>
            ```
         - uid 属性の部分

            ```xml
            <AttributeDefinition xsi:type="Simple" id="uid" sourceAttributeID="uid">
            	<Dependency ref="myLDAP" />
            	<AttributeEncoder xsi:type="SAML1String" name="urn:mace:dir:attribute-def:uid" encodeType="false" />
            	<AttributeEncoder xsi:type="SAML2String" name="urn:oid:0.9.2342.19200300.100.1.3" friendlyName="uid" encodeType="false" />
         	</AttributeDefinition>
            ```

4.  attribute-resolver で設定した属性情報のうち、Shibboleth から ArcGIS Online に返されるユーザーの属性を設定します。ArcGIS Online に返す属性の設定は、\<Shibboleth インストール ディレクトリ>/conf/attribute-filter.xml ファイルで行います。以下の内容をファイルに追記します（”Requester” value には、[「2.1. ArcGIS Online に Shibboleth を Idp として登録」](#21-arcgis-online-に-shibboleth-を-idp-として登録)で確認した ArcGIS Online のエンティティ ID（本ガイドでは nk226.arcgis.com）を入力します）。

    ```xml
    <AttributeFilterPolicy id="ArcGIS">
    	<PolicyRequirementRule xsi:type="Requester" value="nk226.maps.arcgis.com" />
    	<AttributeRule attributeID="uid">
    		<PermitValueRule xsi:type="ANY" />
    	</AttributeRule>
    	<AttributeRule attributeID="givenName">
    		<PermitValueRule xsi:type="ANY" />
    	</AttributeRule>
    	<AttributeRule attributeID="mail">
    		<PermitValueRule xsi:type="ANY" />
    	</AttributeRule>
    </AttributeFilterPolicy>
    ```

5.  アサーション（認証応答）に関する設定を行います。アサーションの設定は \<Shibboleth インストール ディレクトリ>/conf/relying-party.xml ファイルを使用して行います。

    以下の XML コードを shibboleth.RelyingPartyOverrides エレメント内に貼り付けて、Shibboleth IDP のデフォルト構成を無効にします。relyingPartyIds には、ArcGIS Online のエンティティ ID（本ガイドでは nk226.arcgis.com）を入力します。nameIDFormatPrecedence パラメーターを使用して、ArcGIS Online で必要となる SAML 名前 ID 属性を unspecified 形式で送信するように IDP に指示します。また、encryptAssertions パラメーターを false に設定して、Shibboleth IDP でのアサーションの暗号化を無効にします。

    ```xml
    <bean parent="RelyingPartyByName" c:relyingPartyIds="nk226.maps.arcgis.com">
    	<property name="profileConfigurations">
    		<list>
    			<bean parent="SAML2.SSO"   p:nameIDFormatPrecedence="urn:oasis:names:tc:SAML:1.1:nameid-format:unspecified" p:encryptAssertions="false" />
    		</list>
    	</property>
    </bean>
    ```

6.  Shibboleth IDP と ArcGIS Online の間で共有する NameID（LDAP で管理するユーザー アカウントと ArcGIS Online のユーザー アカウントを紐づけるユーザー識別子）を定義します。設定は <Shibboleth インストール ディレクトリ>/conf/saml-nameid.xml ファイルを使用します。  
    以下の部分を

      ```xml
      <!--
      	<bean parent="shibboleth.SAML2AttributeSourcedGenerator"
      		p:format="urn:oasis:names:tc:SAML:1.1:nameid-format:emailAddress"
       		p:attributeSourceIds="#{ {'mail'} }" />
      -->
      ```

      次のように変更します。

      ```xml
      <bean parent="shibboleth.SAML2AttributeSourcedGenerator"
      	p:format="urn:oasis:names:tc:SAML:1.1:nameid-format:unspecified"
       	p:attributeSourceIds="#{ {'uid'} }" />
      ```

### 2.3. 動作確認

エンタープライズ ログインを使用して ArcGIS Online にアクセスできるか確認します。

1. Shibboleth のデーモン (Linux) を再起動します。

2. ArcGIS Online 組織サイトにアクセスし、ログイン ボタン（「2. ArcGIS Online と Shibboleth の連携設定」 の手順 5 で入力した名前が表示されています）を選択します。

   <br>
   <img width="500" alt="" src="assets\shibboleth09.png">
   <br>

3. Shibboleth のログイン画面が表示されます。ユーザー名とパスワードを入力し、[Login] をクリックします。

   <br>
   <img width="500" alt="" src="assets\shibboleth10.png">
   <br>

      ArcGIS Online の組織サイトにログインされます。
      なお、SAML を使用する場合、ユーザー名の命名規則は uid\_<サブドメイン> となります（本ガイドでは test002_nk226）。

      以上で動作確認は終了です。
   <br>
   <img width="500" alt="" src="assets\shibboleth11.png">
   <br>

## 3. 参考資料

本章では参考資料として、ArcGIS Online に新規追加されるユーザーに、デフォルトで割り当てるユーザー タイプやロールなどを設定する方法を以下に記載します。

### デフォルトで割り当てるユーザータイプやロール等を設定する方法

1. ArcGIS Online に管理者権限を持つユーザーでログインし、[組織] → [設定] → [新しいメンバーのデフォルト設定] に移動します。
2. 新規追加するユーザーにデフォルトで割り当てるユーザー タイプやロールを設定する画面が表示されます。ユーザー タイプあるいはロールの鉛筆ボタンをクリックします。

   <br>
   <img width="500" alt="" src="assets\shibboleth12.png">
   <br>

3. 設定画面が表示されるので、デフォルトで割り当てたいユーザー タイプとロールを設定し、[保存] をクリックします。

   <br>
   <img width="500" alt="" src="assets\shibboleth13.png">
   <br>

   ユーザー タイプやロールの設定方法は以上です。なお、ユーザー タイプやロール以外にも、デフォルトで割り当てるアドオン ライセンスやグループを設定することも可能です。それらの設定を行う場合は、以下の手順にお進みください

#### ・アドオンライセンスを割り当てる手順

1. [新しいメンバーのデフォルト設定] タブの [アドオン ライセンス] セクションで [管理] をクリックします。

   <br>
   <img width="500" alt="" src="assets\shibboleth14.png">
   <br>

2. 設定画面が表示されるので、ユーザーにデフォルトで割り当てたいアドオン ライセンスにチェックを入れ、[保存] をクリックします。

   <br>
   <img width="500" alt="" src="assets\shibboleth15.png">

#### ・グループを割り当てる方法

1. [新しいメンバーのデフォルト設定] タブの [グループ] セクションで [管理] をクリックします。

   <br>
   <img width="500" alt="" src="assets\shibboleth16.png">

2. 設定画面が表示されるので、デフォルトで割り当てたいグループにチェックを入れ、[保存] をクリックします。

   <br>
   <img width="500" alt="" src="assets\shibboleth17.png">
