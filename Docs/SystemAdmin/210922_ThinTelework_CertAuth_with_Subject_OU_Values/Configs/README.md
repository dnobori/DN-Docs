「IPA-DN-ThinController-Private/Vars/Vars/Vars.cs」を、以下のように編集する。

## 1. 「IPA.App.ThinVars.ThinVarsGlobal.Certs」 クラスの本文部分
```
public static PalX509Certificate Html5ClientCertAuth_RootCaCert => Html5ClientCertAuth_RootCaCert_Singleton;
```
の下に、
```
// 2021/09/21 ユーザーの提示するクライアント証明書が "Cybertrust DeviceiD Public CA G3isr" の CA によって署名されていることを
// 検証するための "Cybertrust DeviceiD Public CA G3isr" のルート CA 証明書ファイルの定義
static readonly Singleton<PalX509Certificate> Html5ClientCertAuth_Sample_210921_CyberTrust_Device_ID_Public_CA_G3isr_Root_Cert_Singleton =
    new Singleton<PalX509Certificate>(() => new PalX509Certificate(new FilePath(AppGlobal.AppRes,
        "ThinWebClient_ClientCertAuth_SampleCerts/210921_CyberTrust_Device_ID_Public_CA_G3isr_Root_Cert/210921_CyberTrust_Device_ID_Public_CA_G3isr_Root_Cert.cer")));

public static PalX509Certificate Html5ClientCertAuth_Sample_210921_CyberTrust_Device_ID_Public_CA_G3isr_Root_Cert => Html5ClientCertAuth_Sample_210921_CyberTrust_Device_ID_Public_CA_G3isr_Root_Cert_Singleton;
```
という定義を追加する。これにより、`210921_CyberTrust_Device_ID_Public_CA_G3isr_Root_Cert.cer` ファイルの内容が、同クラスの `Html5ClientCertAuth_Sample_210921_CyberTrust_Device_ID_Public_CA_G3isr_Root_Cert` 変数としてイミュータブルに定義されるのである。


上記の `210921_CyberTrust_Device_ID_Public_CA_G3isr_Root_Cert.cer` は例であり、実際には、利用したい CA 証明書を、ここで指定すること。なお、上記の変数名は異なっていても問題無い。


## 2. 「IPA.App.ThinVars.ThinVarsGlobal.ThinWebClientVarsConfig.InitalizeWebServerConfig()」メソッドの本文部分
```
opt.ClientCertificateValidatorAsync = async (cert, chain, err) =>
{
    // (中略)
};
```
という非同期匿名デリケートメソッド定義として、証明書の検証ルールをスクリプトとして記述することができるようになっている。デフォルトで色々記載されているが、これを、自由に編集することができる。今回の目的においては、
```
opt.ClientCertificateValidatorAsync = async (cert, chain, err) =>
{
    var clientCertObject = cert.AsPkiCertificate();

    if (clientCertObject.CheckIfSignedByAnyOfParentCertificatesListOrExactlyMatch(
        new Certificate[] {
            Certs.Html5ClientCertAuth_Sample_210921_CyberTrust_Device_ID_Public_CA_G3isr_Root_Cert,
        },
        out bool exactlyMatchToRootCa) == false)
    {
        throw new Exception($"The client SSL certificate '{clientCertObject}' is not trusted by any of root CA certificates.");
    }
    else if (exactlyMatchToRootCa == false)
    {
        // クライアント証明書の有効期限を検査
        if (clientCertObject.IsExpired())
        {
            throw new Exception($"The client SSL certificate '{clientCertObject}' is expired or not valid. NotBefore = '{clientCertObject.NotBefore._ToDtStr()}', NotAfter = '{clientCertObject.NotAfter._ToDtStr()}'");
        }

        // クライアント証明書の Subject の値を検査する。OU の値が指定した文字列に一致しているかどうかを確認する。
        string[] ou = clientCertObject.CertData.SubjectDN.GetValueList(Org.BouncyCastle.Asn1.X509.X509Name.OU).Cast<string>().ToArray();

        if (ou[1] == "ABC" && ou[2] == "DEF")
        {
            // OK
        }
        else
        {
            // エラー
            throw new Exception($"Subject OU value check failed. Client SSL certificate: '{clientCertObject}'");
        }
    }

    return true;
};
```
というように書き換えればよい。上記のように条件式を記述すると、
```
2 つ目の OU の値と 3 つ目の OU の値の文字列がそれぞれ "ABC", "DEF" に完全一致すること。
```
というルールが実装されることとなる。なお、C# では配列は 0 番から開始されるので、`ou[1]` は 2 つ目の OU の値、`ou[2]` は 3 つ目の OU の値を意味していることに留意する。そして、単純な文字列比較を行なっているため、"ABC", "DEF" の大文字・小文字が異なるとエラーになることに留意する。


上記の ABC, DEF は例であり、実際には、利用したい証明書の OU 値のルールを、ここに記述すること。




