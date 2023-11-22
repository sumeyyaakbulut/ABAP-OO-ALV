## OO ALV Event.

Eventler top of page , hotspot click , double click  , data changed , onf4 changed den oluşur. Bu içeriklerin hepsini sırayla bakalım.

### TOP OF PAGE
İlk olarak top of page örnek üzerinde bakalım.
Bunun için ilk include programda kullandığımız içerikleri tanımladığımızın include içeriği aşağıdadır. Scarr tablosundan hem structure hem de internal tablo oluşturalım.
Fieldcat structure ve internal tablo oluşturulmuştur.
Layout tanımlanır.
Sub1 ve sub2 header ve tablo olarak iki içerik kullanacağımız için sub iki tane tanımlanır.
Oluşturduğumuz class dan nesne oluşturacağımız için , data tanımlaması olan include olduğumuz için biz bu class dan nesne oluşturmaya çalışsak bile bu class aşağıdaki include tanımlandığı için de burada bu classı tanımıyorum diye hata verecektir .Bu yüzden referans alacağımız classı deferred ile tanımlayalım. Bunu kullanmamızın amacı local class tanımlanmadan önce referans olarak kullanmak istiyorsak bu ifade kullanılır. Böylelikle de böyle bir sınıfın varlığı bilinir hale getirilir.
Top of Page yazmak istediğimiz içerikleri ekranda görünebilmesi için cl dd document den referans alınarak oluşturulur.

```cadence
DATA: GO_ALV TYPE REF TO CL_GUI_ALV_GRID,
      GO_CUST TYPE REF TO CL_GUI_CUSTOM_CONTAINER.

DATA: GT_SCARR TYPE TABLE OF SCARR,
      GS_SCARR TYPE SCARR.

DATA: GT_FCAT TYPE LVC_T_FCAT,
      GS_FCAT TYPE LVC_S_FCAT,
      GS_LAY TYPE lvc_s_layo.

DATA: go_spli TYPE REF TO CL_GUI_SPLITTER_CONTAINER,
      go_sub1 TYPE REF TO CL_GUI_CONTAINER,
      go_sub2 TYPE REF TO CL_GUI_CONTAINER.

FIELD-SYMBOLS: <GFS_FCAT> TYPE LVC_S_FCAT.

DATA : go_doc TYPE REF TO CL_DD_DOCUMENT.
CLASS CL_EVENTS DEFINITION DEFERRED.
DATA: GO_EVENT_RECEIVER TYPE REF TO CL_EVENTS.

```
Biz bu örneğimizde class kullandığımız için de ikinci include olarak class alalım. Bu class aşağıdaki şekildedir.
İlk olarak class içerindeki içerisin de kullanılacak içerikleri definition ile tanımlayalım. Method public olarak tanımlayalım. Metodun ismi biz belirliyoruz. For event ile  de cl guı alv grid  tıklayarak events kısmından istediğimiz içeriği yazalım. Bu Kısıma da o içeriğin ismini yazalım .Sonra da  cl guı alv içerisinde events altında top of page parameters bakabiliriz. O parametreler kopyalanır ve b kısma yerleştirilir.

```cadence
CLASS CL_EVENTS DEFINITION.
  PUBLIC SECTION.

    METHODS handle_top_of_page
    FOR EVENT top_of_page of CL_GUI_ALV_GRID
    IMPORTING
      E_DYNDOC_ID
      TABLE_INDEX.
```
Class implementation yani classa işlevsellik kazandırdığımız yer aşağıdaki gibidir. Defination kısmında tanımladığımız method ismini yazalım. Ekranın üzerine yazacağımız içerikleri değerini tutabilmesi için değişken tanımlayalım. Concatanate  ile de yazmak istediğimiz içerikleri birleştirelim. Add text ile ekrana yazalım ve yazarken hangi renk ve yazı boyutunu belirleyelim. Renkler list negative ,list pozitive , list total ,list header olarak renkler verilebilir.
Yazı  boyutunu da large ,small , medium olarak yapabiliriz.
Sonra da biz bu içeriği display ile go sub ile ekrana yazılır.

```cadence
CLASS CL_EVENTS IMPLEMENTATION.

  METHOD HANDLE_TOP_OF_PAGE.
    DATA: LV_TEST TYPE SDYDO_TEXT_ELEMENT.

    LV_TEST = 'FLIGHT DETAILS'.
    CALL METHOD go_doc->NEW_LINE.
    CLEAR: LV_TEST.

    CONCATENATE 'User' sy-uname INTO LV_TEST SEPARATED BY SPACE.
    CALL METHOD go_doc->ADD_TEXT
      EXPORTING
        TEXT         = LV_TEST
        SAP_COLOR    = CL_DD_DOCUMENT=>LIST_HEADING
        SAP_FONTSIZE = CL_DD_DOCUMENT=>LARGE.

    CALL METHOD GO_DOC->DISPLAY_DOCUMENT
      EXPORTING
        PARENT = GO_SUB1.
  ENDMETHOD.
```

Üçüncü include içeriği de 

```cadence
MODULE STATUS_0100 OUTPUT.
SET PF-STATUS 'STATUS100'.
  SET TITLEBAR 'Event'.
PERFORM display_alv.
ENDMODULE.                 " STATUS_0100  OUTPUT
```
Display formu burada çağırıyoruz çünkü ekran açılır açılmaz alv basılmasını istediğimiz için burada tanımlarız.


Dördüncü include içeriği aşağıdaki şekildedir.
```cadence
MODULE USER_COMMAND_0100 INPUT.
  CASE SY-UCOMM.
    WHEN '&BACK'.
      LEAVE TO SCREEN 0.
  ENDCASE.
ENDMODULE. 
```


Beşinci include içeriği ise aşağıdaki şekildedir. Bu kısmın diğer oo alv deki aynı olduğu  gibidir.

```cadence
FORM GET_DATA .
  SELECT * from scarr into CORRESPONDING FIELDS OF TABLE GT_SCARR.
ENDFORM. 

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
  .

   ENDFORM.                    " SET_FCAT
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
  GS_LAY-NO_TOOLBAR = abap_true.
ENDFORM.  
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
  .

    ENDFORM.                    " SET_FCAT
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
  GS_LAY-NO_TOOLBAR = abap_true.
ENDFORM.  
```

Bu kısımda ekrana alv ve top of page basmak için kullanılır. Container nesne oluşturulup basılır.biz alt alta yazı ve alv basacağımız için de iki tane satır bir tane de sütun olur.

```cadence
FORM DISPLAY_ALV .

  CREATE OBJECT GO_CUST
    exporting
      CONTAINER_NAME              =    'CC_ALV'.

  CREATE OBJECT GO_SPLI
    EXPORTING
      PARENT  = GO_CUST
      ROWS    = 2
      COLUMNS = 1.
  CALL METHOD GO_SPLI->GET_CONTAINER
    EXPORTING
      ROW       = 1
      COLUMN    = 1
    RECEIVING
      CONTAINER = GO_SUB1. " Container


  CALL METHOD GO_SPLI->GET_CONTAINER
    EXPORTING
      ROW       = 2
      COLUMN    = 1
    RECEIVING
      CONTAINER = GO_SUB2. " Container

  CALL METHOD GO_SPLI->SET_ROW_HEIGHT
    EXPORTING
      ID     = 1
      HEIGHT = 15.
  CREATE OBJECT GO_DOC
    EXPORTING
      STYLE = 'ALV_GRID'.
  CREATE OBJECT GO_ALV
         EXPORTING
*        I_PARENT = CL_GUI_CONTAINER=>SCREEN0.
           I_PARENT =   GO_SUB2.
  .

  CREATE OBJECT GO_EVENT_RECEIVER.

  SET HANDLER GO_EVENT_RECEIVER->HANDLE_HOTSPOT_CLICK FOR GO_ALV.
  SET HANDLER GO_EVENT_RECEIVER->HANDLE_DOUBLE_CLICK FOR GO_ALV.
  set HANDLER GO_EVENT_RECEIVER->HANDLE_TOP_OF_PAGE FOR go_alv.
  CALL METHOD GO_ALV->SET_TABLE_FOR_FIRST_DISPLAY
    EXPORTING
      IS_LAYOUT                     = GS_LAY
    CHANGING
      IT_OUTTAB                     = GT_SCARR
      IT_FIELDCATALOG               = GT_FCAT
    EXCEPTIONS
      INVALID_PARAMETER_COMBINATION = 1
      PROGRAM_ERROR                 = 2
      TOO_MANY_LINES                = 3
      OTHERS                        = 4.
  if sy-subrc <> 0.

  endif.

  CALL METHOD GO_ALV->LIST_PROCESSING_EVENTS
    exporting
      I_EVENT_NAME      =    'TOP_OF_PAGE'
      I_DYNDOC_ID       =    GO_DOC
    .
ENDFORM.            
```
