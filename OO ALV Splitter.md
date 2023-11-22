## OO ALV Splitter

Bu içeriği kullanmadan birden fazla container oluşturarak ekrana basabiliriz o zaman da  ekrana bastığımız tablonun genişliğine göre değil de bizim tanımladığımız container genişliğine göre ekrana basılır ve ekrana basıldıktan sonra değişiklik yapılmaz.

İlk olarak splitter kullanmadan yapalım.
Bunun için scrren de bir tane container oluşturalım.
İlk olarak iki tane container oluşturalım CC_ALV ve CC_ALV2 isimlerini verelim .Container için obje oluşturalım .Screen de oluşturduğumuz container ismini verelim .Bu containerden nesne oluşturduğumuz için de bu ismi de parent kısmına verelim.
Sonra da go alv deki set table for first metodunu çağıralım.Bu kısımda oluşturduğumuz layout ve ekrana basacağımız internal tabloyu ve fieldcat kısmı burada verilir.
Bu işlemler go alv boş ise yapılır ,boş değil dolu ise güncelleme yapılır.

Bu yukarıda yaptığımız aynı işlemleri diğer container için de yapılır burada farklı olan fieldcat ,container ismi, ve ekrana basacağımız internal tablo değerleri değiştirilir.

```cadence
 IF GO_ALV is INITIAL.
    CREATE OBJECT GO_CONT
      exporting
        CONTAINER_NAME              =    'CC_ALV'.
    CREATE OBJECT GO_ALV
      EXPORTING
        I_PARENT =   GO_CONT.
    .
    call METHOD GO_ALV->SET_TABLE_FOR_FIRST_DISPLAY
     exporting

       IS_LAYOUT                     =     GS_LAY
      changing
        IT_OUTTAB                     =     GT_SCARR
        IT_FIELDCATALOG               =     GT_FCAT
      .
         else.
    CALL METHOD go_alv->REFRESH_TABLE_DISPLAY.

  ENDIF.
 IF GO_ALV2 is INITIAL.

    CREATE OBJECT GO_CONT2
      exporting
        CONTAINER_NAME              =    'CC_ALV2'.
    CREATE OBJECT GO_ALV2
      EXPORTING
        I_PARENT =   GO_CONT2.
    .
    call METHOD GO_ALV2->SET_TABLE_FOR_FIRST_DISPLAY
     exporting
       IS_LAYOUT                     =     GS_LAY
      changing
        IT_OUTTAB                     =     GT_SFLIGHT
        IT_FIELDCATALOG               =     GT_FCAT2
      .
  else.
    CALL METHOD go_alv2->REFRESH_TABLE_DISPLAY.
  ENDIF.
```
Bu şekilde splitter kullanmadan da ekrana birden fazla tablo basılabilir.

Splitter kullanarak yaparsak eğer bir örnek üzerin yapalım.
İlk olarak include oluşturalım.

İlk include değişkenleri tanımladığımız includumuzdur.
CL GUI ALV GRID den referans alarak  iki tane go alv oluşturalım. İki tane oluşturmamızın amacı ekrana iki tane alv basacağımız içindir.
Scarr ve sflight tablolarını kullanacağımız için bunlardan internal tablo oluşturalım.
İki tablo ekrana basacağımız için de iki tane gui oluşturalım.
İki tablo olacağı için iki tane fieldcat kullanılır.
Bir tane de layout tanımlayalım.

```cadence
DATA: GO_ALV TYPE REF TO CL_GUI_ALV_GRID,
      GO_ALV2 TYPE REF TO CL_GUI_ALV_GRID,
      GO_CONT TYPE REF TO CL_GUI_CUSTOM_CONTAINER.

DATA: GT_SCARR TYPE TABLE OF SCARR.
DATA: GT_SFLIGHT TYPE TABLE OF SFLIGHT,
      GO_GUI TYPE REF TO CL_GUI_CONTAINER,
      GO_GUI2 TYPE REF TO CL_GUI_CONTAINER.

DATA: go_splitter TYPE REF TO CL_GUI_SPLITTER_CONTAINER.

DATA: GT_FCAT TYPE LVC_T_FCAT,
      GT_FCAT2 TYPE LVC_T_FCAT,
      GS_FCAT TYPE LVC_S_FCAT,
      GS_LAY TYPE LVC_S_LAYO.
```

İkinci include pbo (process before output kısmınıda) aşağıdaki şekilde yapalım.
Bir status ve title oluşturup bu kısımları oluşturduğumuz içeriklerin ismini yazalım.Bu kısımda ekrana yazacağımız içerikleri display alv içine yazalım ve bu ekran açılır açılmaz görünsün istediğimiz için de pbo içinde çağıralım.

```cadence
MODULE STATUS_0100 OUTPUT.
  SET PF-STATUS 'STATUS100'.
  SET TITLEBAR 'Splitter Container'.
PERFORM display_alv.
ENDMODULE. 
```
Bu display içeriğine bakalım. Go cont dan obje oluşturalım ve buraya screen de oluşturduğumuz container ismini verelim daha sonra bu oluşturduğumuz objeyi de splitterden obje oluşturduktan sonra parent kısmında çağıralım. Splitter kısmında row ve columns kısımları bulunur. Matrix gibi düşünebiliriz. row satır  column sütun değerlerini tutar.

GUI iki tane alv ekrana basacağımız için onların konumunu yazalım. Mesela biz alv yan yana yazmak için 1 1 dersek sol da ilk satıra yazar , 1 2 yazarsak da il satırda sağ tarafta yani ikinci sütunda olur.
Sonra da parent ile ikisi de yazılır.

Bu iki tabloyu ekrana basabilmek için go alv  set table for first display metodu çağıralım .Bu kısımda oluşturduğumuz layout ekrana basmak istediğimiz structure ismini ve fieldcat ismini yazalım.
Diğer tablomuz da aynı işlemi yapalım .Bu kısımda da aynı işlem yapılır tek fark ise ALV ekrana basacağımız internal tablo ve fieldcat kısmını değiştirip aynı şekilde yazalım.

```cadence
FORM DISPLAY_ALV .
    CREATE OBJECT GO_CONT
      exporting
        CONTAINER_NAME              =    'CC_ALV'.

    CREATE OBJECT GO_SPLITTER
      exporting
        PARENT                  =  GO_CONT
        ROWS                    =   1
        COLUMNS                 =   2
      .
CALL METHOD GO_SPLITTER->GET_CONTAINER
  exporting
    ROW       =  1
    COLUMN    =  1
  receiving
    CONTAINER =  GO_GUI   " Container
  .
    CALL METHOD GO_SPLITTER->GET_CONTAINER
  exporting
    ROW       =  1
    COLUMN    =  2
  receiving
    CONTAINER =  GO_GUI2   " Container
  .
    CREATE OBJECT GO_ALV
      EXPORTING
        I_PARENT =   GO_GUI.
    .
       CREATE OBJECT GO_ALV2
      EXPORTING
        I_PARENT =   GO_GUI2.
    .
    call METHOD GO_ALV->SET_TABLE_FOR_FIRST_DISPLAY
     exporting
       IS_LAYOUT                     =     GS_LAY
      changing
        IT_OUTTAB                     =     GT_SCARR
        IT_FIELDCATALOG               =     GT_FCAT
      .
call METHOD GO_ALV2->SET_TABLE_FOR_FIRST_DISPLAY
     exporting
       IS_LAYOUT                     =     GS_LAY
      changing
        IT_OUTTAB                     =     GT_SFLIGHT
        IT_FIELDCATALOG               =     GT_FCAT2
      .
```
Üçüncü include muzda ise pai process after input kısmı kullanıcı ekranda herhangi  bir içeriği butona bastığı zaman bu kısımdan kontrolleri ve işlemleri yapılır.

```cadence
MODULE USER_COMMAND_0100 INPUT.
CASE sy-ucomm.
  WHEN '&BACK'.
    LEAVE TO SCREEN 0.
ENDCASE.
ENDMODULE.
```
Dördüncü include içerisi
Get data kısmı içerisin de oluşturduğumuz include tabloların içeriğini select ile dolduralım. Biz sap hazır tablolarından kullandığımız ve ondan internal tablo oluşturduğumuz için select ile into table (correspording kullanmamıza gerek yok kendimiz type table yapsaydık o zaman sütun uyuşmazlığı olmasın diye yapmamız gerekliydi)

```cadence
FORM GET_DATA .
  SELECT * from scarr into TABLE GT_SCARR.
    SELECT * from SFLIGHT into TABLE GT_SFLIGHT.
ENDFORM. 
```

Daha sonra da fieldcat ile tüm satırları tek tek özelliklerini vermek yerine merge kullanılarak yapılır. Structure kısmına istersek view yada sapnın hazır tablosu yada se11 oluşturulan structure kullanılabilir. Bizim iki tane tablomuz olduğu için iki tane merge kullanılır.

```cadence
FORM SET_FCAT .
  CALL FUNCTION 'LVC_FIELDCATALOG_MERGE'
    EXPORTING
      I_STRUCTURE_NAME       = 'SCARR'
    CHANGING
      CT_FIELDCAT            = GT_FCAT
    EXCEPTIONS
      INCONSISTENT_INTERFACE = 1
      PROGRAM_ERROR          = 2
      OTHERS                 = 3.
  IF SY-SUBRC <> 0.
  ENDIF.

    CALL FUNCTION 'LVC_FIELDCATALOG_MERGE'
    EXPORTING
      I_STRUCTURE_NAME       = 'SFLIGHT'
    CHANGING
      CT_FIELDCAT            = GT_FCAT2
    EXCEPTIONS
      INCONSISTENT_INTERFACE = 1
      PROGRAM_ERROR          = 2
      OTHERS                 = 3.
  IF SY-SUBRC <> 0.
  ENDIF.
ENDFORM.   
```

Layout da ekrana bastığımız alv sütun genişliği otomatik olarak düzenlenir.

```cadence
FORM SET_LAYOUT .
  CLEAR GS_LAY.
  GS_LAY-CWIDTH_OPT = abap_true.
*  GS_LAY-NO_TOOLBAR = abap_true.
ENDFORM.
```

Splitter ile ekrana basılan tabloları ekranda belli büyüklerde görmek istersek aşağıdaki şekil de kullanılır. Genişlik belirlerken

```cadence
 CALL METHOD GO_SPLITTER->SET_COLUMN_WIDTH
        EXPORTING
          ID    = 1
          WIDTH = 60.

Yükseklik belirlenirken kullanılır.
  CALL METHOD GO_SPLI->SET_ROW_HEIGHT
    EXPORTING
      ID     = 1
      HEIGHT = 15.
```
