# HackingWeek CTF 2015: Net 4

**Category:** net |
**Points:** 4644 |
**Solves:** 33 |
**Description:**


> On dispose d'un fichier (`pcapssl-capture.pcap`), de la clef publique du serveur Web (`PublicKey.pem`), ainsi que du résultat d'une attaque Heartbleed sur le serveur pendant que l'échange se passait (`heartbleed_attack.log`).
> 
> Avec ces trois éléments, vous devez réussir à déchiffrer l'échange qui a eut lieu. La clef de validation de l'épreuve est le mot de passe de l'utilisateur qui se connecte pendant cet échange chiffré.
> 
> * [ssl-capture.pcap](http://hackingweek.fr/media/Aa0eiHuu/ssl-capture.pcap)
> * [PublicKey.pem](http://hackingweek.fr/media/Aa0eiHuu/PublicKey.pem)
> * [heartbleed_attack.log](http://hackingweek.fr/media/Aa0eiHuu/heartbleed_attack.log)


___

This challenge was really fun, here is the step to solve it.

- _Look at the PublicKey.pem_ :

## Write-up

```bash
$ openssl x509 -in PublicKey.pem -text -noout -modulus 
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            c0:35:60:84:36:24:b5:c3
    Signature Algorithm: sha1WithRSAEncryption
        Issuer: CN=ubuntu  <== LEL
//...//
Modulus=A9F1901F0A1E2CBCF8C79E9B37A73A32A67A6A525A11B78C4FF30293491C68C43AEA7FE9E60A50C32EE55640D2B21737E1EC4786982A903D41B684A79C03BE5B67B9FA6CEF65C3733C7764F86D4C38020B88695E59E62FAE1D095E22553B14F557B2968AAD3BCB0379C622F58A7478D3DFD9A5E22BD069B1E55538586ECBD23A6A0B4B2AE9E317FD4D328AF0638E999EC5B6EEA54E4DBEE30B39AD36AB35E929A49F411DF38367719AF2BD637802685CE2D9C0076567545287736BE5B51E28B1E8643E7495708878AECC07394C5A8A0BC74D68AF9CEBE40931BAFF70CEF00065740DBCCC978071AA5325A56A7A9F8244B5526EE72E2DFD32C70E45597A965D2B
```
<br><br>

Moreover, in the `heartbleed_attack.log`, we have a long interesting hexadecimal string :
```
DC05DF2DA2234A7635AAB34E3835C8DE6D63C01EF26BBC40CD9789A6DEC847A9EF505091A83085AEB63FDA003B4B07F69EADC779090E019C2B9D5008BF564DDE80EE7889D2B33F8D38C0F9E503C35769E3A0DDB7EED4617F4D82BAD877B7BC1016B631270944669C7BF221FD8FDB8B05B9DAB289F18F44F75C360E88938A2ECD
```
<br>
Converted to decimal : 
```
154505360461616922248064634925887787698458246382581570008454841098205176362058153054716586584864800572253287936092439749949400519549787548103712618564774866592690729105816373428201915463608923143735621932793997654468908714003299258111612018699395021710536083706990861285242484886738208843067246867876336512717
```

Let's see if this bigint divided le modulus in the `PublicKey` :
```
>>> 154505360461616922248064634925887787698458246382581570008454841098205176362058153054716586584864800572253287936092439749949400519549787548103712618564774866592690729105816373428201915463608923143735621932793997654468908714003299258111612018699395021710536083706990861285242484886738208843067246867876336512717 * (0xA9F1901F0A1E2CBCF8C79E9B37A73A32A67A6A525A11B78C4FF30293491C68C43AEA7FE9E60A50C32EE55640D2B21737E1EC4786982A903D41B684A79C03BE5B67B9FA6CEF65C3733C7764F86D4C38020B88695E59E62FAE1D095E22553B14F557B2968AAD3BCB0379C622F58A7478D3DFD9A5E22BD069B1E55538586ECBD23A6A0B4B2AE9E317FD4D328AF0638E999EC5B6EEA54E4DBEE30B39AD36AB35E929A49F411DF38367719AF2BD637802685CE2D9C0076567545287736BE5B51E28B1E8643E7495708878AECC07394C5A8A0BC74D68AF9CEBE40931BAFF70CEF00065740DBCCC978071AA5325A56A7A9F8244B5526EE72E2DFD32C70E45597A965D2B / 154505360461616922248064634925887787698458246382581570008454841098205176362058153054716586584864800572253287936092439749949400519549787548103712618564774866592690729105816373428201915463608923143735621932793997654468908714003299258111612018699395021710536083706990861285242484886738208843067246867876336512717) == 0xA9F1901F0A1E2CBCF8C79E9B37A73A32A67A6A525A11B78C4FF30293491C68C43AEA7FE9E60A50C32EE55640D2B21737E1EC4786982A903D41B684A79C03BE5B67B9FA6CEF65C3733C7764F86D4C38020B88695E59E62FAE1D095E22553B14F557B2968AAD3BCB0379C622F58A7478D3DFD9A5E22BD069B1E55538586ECBD23A6A0B4B2AE9E317FD4D328AF0638E999EC5B6EEA54E4DBEE30B39AD36AB35E929A49F411DF38367719AF2BD637802685CE2D9C0076567545287736BE5B51E28B1E8643E7495708878AECC07394C5A8A0BC74D68AF9CEBE40931BAFF70CEF00065740DBCCC978071AA5325A56A7A9F8244B5526EE72E2DFD32C70E45597A965D2B
True
```
<br><br>

Well, now I have `p` and `q` I can reconstruct the privatekey easily.

```
tofinish
```
