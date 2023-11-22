## OO ALV Variant
Tabloda yaptığımız artan azalan sıralaması yada bazı sütunların görünmesini istemiyorsak ve programa kapatılıp açtığımızda yaptığımız bu değişikliği görebilmek istiyorsak variant kullanılmalıdır.
Bunun için de ilk olarak oo alv ekrana basmak için kullandığımız variant ve save kısımları açılır.
Reuse alv de save kısmını açtığımız zaman değişiklikleri sakla gibi içerikler için olan buton aktif haldedir.
Ama OO ALV de böyle değildir o yüzden de program adını vermek için variant structure programın ismi verilir.
Variant kısmına da structure olarak oluşturduğumuz variant yazılır.

```cadence
 GS_VARIANT-REPORT  = SY-REPID.
 CALL METHOD GO_ALV4->SET_TABLE_FOR_FIRST_DISPLAY
        exporting
           IS_VARIANT                    =     GS_VARIANT
           I_SAVE                        =     'A'
           IS_LAYOUT                     =     GS_LAY
        changing
          IT_OUTTAB                     =     GT_HEMLAK
          IT_FIELDCATALOG               =     GT_FCAT4.

```
