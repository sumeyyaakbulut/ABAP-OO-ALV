## OO ALV PustButton
Yeni bir sütun ekleyerek bu eklediğimiz sütuna da buton ekleriz.

İlk include aşağıdaki şekilde yazalım. İcon kullanabilmek için type pools ile programa iconu tanımlayalım.
CL GUI ALV GRID  referansa alarak class oluşturalım.
Container screen de tanımlamamıza gerek yoktur çünkü biz fullscreen container olmadan ekrana basacağımız için bu içerik yazılmaz.

Daha sonra table type tanımlayalım .Direk scarr dan internal tablo oluşturuşsak eğer scarr aynı sütunlara sahip bir internal tablo elde etmiş oluruz. Biz yeni sütun eklemek  için types ile tanımlayalım.

Tanımladığımız table type dan da structure ve internal tablo oluşturalım.
Oluşturduğumuz bu tablodaki sütun isimlerini yazabilmek ve sütunun içeriklerini yazabilmek için de fieldcat structure ve internal olarak tanımlayalım.

Field symbol tanımlayalım. Bu bir pointer gibi işaretleyicidir. Bu işaretleme işleminde bu işlem için bir yer tutmaz.

Bu işlem için bir class oluşturacağız ileride ama şimdi bu class dan referans alıp oluşturacağımız için bu class dan refernas alınır. Elimiz de şuan böyle bir class olmadığı için biz burada böyle bir class olduğunu belirtmek için deferred kullanılır.

```cadence
TABLES: scarr.
TYPE-POOLs icon.

DATA: go_alv TYPE REF TO CL_GUI_ALV_GRID.

TYPES: BEGIN OF gty_scarr,
DELETE TYPE CHAR10,
CARRID  TYPE  S_CARR_ID,
CARRNAME type  S_CARRNAME,
CURRCODE type  S_CURRCODE,
URL   type S_CARRURL,

   END OF gty_scarr.

DATA: GT_SCARR TYPE TABLE OF gty_scarr.
DATA: GS_SCARR TYPE gty_scarr.

DATA: GT_FCAT TYPE LVC_T_FCAT,
      GS_FCAT TYPE LVC_S_FCAT,
      GS_LAY TYPE LVC_S_LAYO .

FIELD-SYMBOLS: <gfs_scarr> TYPE GTY_SCARR.

CLASS CL_EVENTS DEFINITION DEFERRED.
DATA: GO_EVENT_RECEIVER TYPE REF TO CL_EVENTS.

```
İkinci include da ise classlar bu kısımdadır. Defination kısmında button click için method oluşturulur.CL GUI ALV GRID de bu metodun parametrelerine bakılır.

Implementation kısmında ise bu method da eklediğimiz sütuna tıklandığı zaman mesaj ekrana basılabilmesi için bir değişken tanımlayalım. Read table ile internal tablomuzun içerisin de structure ile dolanalım. Index olarak da tıkladığımız içeriğin kaçıncı satırda olduğu gösterir.
Subrc değerimiz sıfır ise yani böyle bir değer varsa tıkladığımız sütun ismi bizim istediğimiz sütun ismi ise ekrana o sütunla ilgili bilgileri yazar.

```cadence
CLASS CL_EVENTS DEFINITION.
  PUBLIC SECTION.
    METHODS HANDLE_BUTTON_CLICK
    FOR EVENT BUTTON_CLICK OF CL_GUI_ALV_GRID
    IMPORTING
      ES_COL_ID
     ES_ROW_NO.
ENDCLASS.

class CL_EVENTS IMPLEMENTATION.
  METHOD HANDLE_BUTTON_CLICK.

    DATA LV_MESS  TYPE CHAR200.
    READ TABLE GT_SCARR INTO GS_SCARR INDEX ES_ROW_NO-ROW_ID.
    IF SY-SUBRC EQ 0.
      CASE ES_COL_ID-FIELDNAME.
        WHEN 'DELETE'.
          CONCATENATE ES_COL_ID-FIELDNAME
                      ','
                      GS_SCARR
                      INTO LV_MESS
                      SEPARATED BY SPACE.
      ENDCASE.
      MESSAGE LV_MESS TYPE 'I'.
    ENDIF.
  ENDMETHOD.
ENDCLASS.
```
Üçüncü sütunumuz pbo da  status ve title bar yazılır .Display burada çağrılır çünkü ekran açılır açılmaz alv ekrana basılmasını istiyoruz.

```cadence
MODULE STATUS_0100 OUTPUT.
  SET PF-STATUS '0100'.
  SET TITLEBAR 'xxx'.

PERFORM display_alv.
ENDMODULE.
```

Dördüncü include kullanıcı buton ve içerikleri tıklanınca ekran açıldıktan sonra yapılan işlem bu kısımda yapılır. 

```cadence
MODULE USER_COMMAND_0100 INPUT.
CASE sy-ucomm.
  WHEN '&BACK'.
    SET SCREEN 0.
    WHEN '&SAVE'.

ENDCASE.
ENDMODULE.
```
Beşinci include da parent full screen olarak  alv yi ekrana basalım. CL_EVENTS isimli oluşturduğumuz class dan referans alarak oluşturulmuştur. Bu içerikler instance   olduğu için create ile nesne oluşturmalıyız. 

Set handler ile oluşturduğumuz class da button click  for dan sonra da cl gui alv grid den referans alarak oluşturduğumuz  içerik yazalım.

Go alv içerisi boş ise ekrana basılır, eğer alv basılı ve güncelleme yapılmak istenirse de refresh  ile yapılır.

Get data ile tanımladığımız internal tablomuzun içeriğini dolduralım correspording kullanmamızın amacı scarr da olmayan bir sütun  internal tablomuz da olduğu ,sütun uyuşmazlığı olmasın diye correspording kullanılır.

Looop at ile internal tablou field symbol ile delete butonunu ekleyelim.

Fieldcat kısmında ekranda görünmesini istediğimiz sütunları tanımladığımız kısımdır.
Biz örneğimiz de delete sütununa buton eklemek istediğimiz için de bu kısımda style kısmına yazılır.
Icon da abap true yapılır.

 Layout ile tüm tablonun açık koyu yani zebra bir şekilde görünmesi sağlanır.

 ```cadence
FORM DISPLAY_ALV .

  IF GO_ALV is INITIAL.
    CREATE OBJECT GO_ALV
      EXPORTING
        I_PARENT = CL_GUI_CONTAINER=>SCREEN0.
    .
    CREATE OBJECT GO_EVENT_RECEIVER.
    SET HANDLER GO_EVENT_RECEIVER->HANDLE_BUTTON_CLICK for GO_ALV.
        call METHOD GO_ALV->SET_TABLE_FOR_FIRST_DISPLAY
         exporting
           IS_LAYOUT                     =     GS_LAY
          changing
            IT_OUTTAB                     =     GT_SCARR
            IT_FIELDCATALOG               =     GT_FCAT.
  else.
    CALL METHOD go_alv->REFRESH_TABLE_DISPLAY.

  ENDIF.
ENDFORM.

FORM GET_DATA .
  SELECT * from scarr into CORRESPONDING FIELDS OF TABLE GT_SCARR.

  LOOP AT  GT_SCARR ASSIGNING <GFS_SCARR>.
    <GFS_SCARR>-DELETE = '@11@'.
  ENDLOOP.

ENDFORM.                    " GET_DATA

FORM GET_FCAT .
  CLEAR GS_FCAT.
  GS_FCAT-FIELDNAME = 'DELETE'.
  GS_FCAT-SCRTEXT_S = 'Silme'.
  GS_FCAT-SCRTEXT_M = 'Silme'.
  GS_FCAT-SCRTEXT_L = 'Silme'.
  GS_FCAT-STYLE = cl_gui_alv_grid=>MC_STYLE_BUTTON.
  GS_FCAT-COL_POS = 1.
  GS_FCAT-COL_OPT = ABAP_TRUE.
  GS_FCAT-ICON = ABAP_TRUE.
  APPEND GS_FCAT TO GT_FCAT.

  CLEAR GS_FCAT.
  GS_FCAT-FIELDNAME = 'CARRID'.
  GS_FCAT-SCRTEXT_S = 'Havayolu T.'.
  GS_FCAT-SCRTEXT_M = 'Havayolu Tanımı.'.
  GS_FCAT-SCRTEXT_L = 'Havayolu Şirketi Tanımı.'.
  GS_FCAT-COL_POS = 2.
  GS_FCAT-COL_OPT = ABAP_TRUE.
  GS_FCAT-KEY = ABAP_TRUE.
  APPEND GS_FCAT TO GT_FCAT.

  CLEAR GS_FCAT.
  GS_FCAT-FIELDNAME = 'CARRNAME'.
  GS_FCAT-SCRTEXT_S = 'Havayolu A.'.
  GS_FCAT-SCRTEXT_M = 'Havayolu Adı.'.
  GS_FCAT-SCRTEXT_L = 'Havayolu Şirketi Tanımı.'.
  GS_FCAT-COL_POS = 3.
  GS_FCAT-COL_OPT = ABAP_TRUE.
  APPEND GS_FCAT TO GT_FCAT.

  CLEAR GS_FCAT.
  GS_FCAT-FIELDNAME = 'CURRCODE'.
  GS_FCAT-REF_TABLE = 'SCARR'.
  GS_FCAT-REF_FIELD = 'CURRCODE'.
  GS_FCAT-COL_POS = 4.
  APPEND GS_FCAT TO GT_FCAT.

  CLEAR GS_FCAT.
  GS_FCAT-FIELDNAME = 'URL'.
  GS_FCAT-SCRTEXT_S = 'Havayolu URL.'.
  GS_FCAT-SCRTEXT_M = 'Havayolu URL.'.
  GS_FCAT-SCRTEXT_L = 'Havayolu URL'.
  GS_FCAT-COL_OPT   = ABAP_TRUE.
  GS_FCAT-COL_POS = 5.
  APPEND GS_FCAT TO GT_FCAT.


ENDFORM.                    " GET_FCAT
*&---------------------------------------------------------------------*
*&      Form  SET_LAYOUT
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
*  -->  p1        text
*  <--  p2        text
*----------------------------------------------------------------------*
FORM SET_LAYOUT .
  CLEAR GS_LAY.
  GS_LAY-CWIDTH_OPT = abap_true.
  GS_LAY-ZEBRA = abap_true.
ENDFORM.
```



