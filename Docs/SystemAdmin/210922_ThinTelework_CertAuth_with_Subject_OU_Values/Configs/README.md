�uIPA-DN-ThinController-Private/Vars/Vars/Vars.cs�v���A�ȉ��̂悤�ɕҏW����B

## 1. �uIPA.App.ThinVars.ThinVarsGlobal.Certs�v �N���X�̖{������
```
public static PalX509Certificate Html5ClientCertAuth_RootCaCert => Html5ClientCertAuth_RootCaCert_Singleton;
```
�̉��ɁA
```
// 2021/09/21 ���[�U�[�̒񎦂���N���C�A���g�ؖ����� "Cybertrust DeviceiD Public CA G3isr" �� CA �ɂ���ď�������Ă��邱�Ƃ�
// ���؂��邽�߂� "Cybertrust DeviceiD Public CA G3isr" �̃��[�g CA �ؖ����t�@�C���̒�`
static readonly Singleton<PalX509Certificate> Html5ClientCertAuth_Sample_210921_CyberTrust_Device_ID_Public_CA_G3isr_Root_Cert_Singleton =
    new Singleton<PalX509Certificate>(() => new PalX509Certificate(new FilePath(AppGlobal.AppRes,
        "ThinWebClient_ClientCertAuth_SampleCerts/210921_CyberTrust_Device_ID_Public_CA_G3isr_Root_Cert/210921_CyberTrust_Device_ID_Public_CA_G3isr_Root_Cert.cer")));

public static PalX509Certificate Html5ClientCertAuth_Sample_210921_CyberTrust_Device_ID_Public_CA_G3isr_Root_Cert => Html5ClientCertAuth_Sample_210921_CyberTrust_Device_ID_Public_CA_G3isr_Root_Cert_Singleton;
```
�Ƃ�����`��ǉ�����B����ɂ��A`210921_CyberTrust_Device_ID_Public_CA_G3isr_Root_Cert.cer` �t�@�C���̓��e���A���N���X�� `Html5ClientCertAuth_Sample_210921_CyberTrust_Device_ID_Public_CA_G3isr_Root_Cert` �ϐ��Ƃ��ăC�~���[�^�u���ɒ�`�����̂ł���B


��L�� `210921_CyberTrust_Device_ID_Public_CA_G3isr_Root_Cert.cer` �͗�ł���A���ۂɂ́A���p������ CA �ؖ������A�����Ŏw�肷�邱�ƁB�Ȃ��A��L�̕ϐ����͈قȂ��Ă��Ă���薳���B


## 2. �uIPA.App.ThinVars.ThinVarsGlobal.ThinWebClientVarsConfig.InitalizeWebServerConfig()�v���\�b�h�̖{������
```
opt.ClientCertificateValidatorAsync = async (cert, chain, err) =>
{
    // (����)
};
```
�Ƃ����񓯊������f���P�[�g���\�b�h��`�Ƃ��āA�ؖ����̌��؃��[�����X�N���v�g�Ƃ��ċL�q���邱�Ƃ��ł���悤�ɂȂ��Ă���B�f�t�H���g�ŐF�X�L�ڂ���Ă��邪�A������A���R�ɕҏW���邱�Ƃ��ł���B����̖ړI�ɂ����ẮA
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
        // �N���C�A���g�ؖ����̗L������������
        if (clientCertObject.IsExpired())
        {
            throw new Exception($"The client SSL certificate '{clientCertObject}' is expired or not valid. NotBefore = '{clientCertObject.NotBefore._ToDtStr()}', NotAfter = '{clientCertObject.NotAfter._ToDtStr()}'");
        }

        // �N���C�A���g�ؖ����� Subject �̒l����������BOU �̒l���w�肵��������Ɉ�v���Ă��邩�ǂ������m�F����B
        string[] ou = clientCertObject.CertData.SubjectDN.GetValueList(Org.BouncyCastle.Asn1.X509.X509Name.OU).Cast<string>().ToArray();

        if (ou[1] == "ABC" && ou[2] == "DEF")
        {
            // OK
        }
        else
        {
            // �G���[
            throw new Exception($"Subject OU value check failed. Client SSL certificate: '{clientCertObject}'");
        }
    }

    return true;
};
```
�Ƃ����悤�ɏ���������΂悢�B��L�̂悤�ɏ��������L�q����ƁA
```
2 �ڂ� OU �̒l�� 3 �ڂ� OU �̒l�̕����񂪂��ꂼ�� "ABC", "DEF" �Ɋ��S��v���邱�ƁB
```
�Ƃ������[������������邱�ƂƂȂ�B�Ȃ��AC# �ł͔z��� 0 �Ԃ���J�n�����̂ŁA`ou[1]` �� 2 �ڂ� OU �̒l�A`ou[2]` �� 3 �ڂ� OU �̒l���Ӗ����Ă��邱�Ƃɗ��ӂ���B�����āA�P���ȕ������r���s�Ȃ��Ă��邽�߁A"ABC", "DEF" �̑啶���E���������قȂ�ƃG���[�ɂȂ邱�Ƃɗ��ӂ���B


��L�� ABC, DEF �͗�ł���A���ۂɂ́A���p�������ؖ����� OU �l�̃��[�����A�����ɋL�q���邱�ƁB




