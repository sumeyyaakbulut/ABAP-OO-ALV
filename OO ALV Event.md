## OO ALV Event

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

### HOTSPOT 

OO ALV Hotspot ile field yani tablodaki belirlediğimiz alana tıklama özelliği ekleriz bu sayede de bir tablodan başka tabloya veya farklı içeriklerin görünmesi sağlanır.

CL Events içerisin de hotspot click için method tanımlayalım. İmporting kısmında ki girdileri  cl guı alv grid içerisinden bakılarak  event  hotspot parametrelerine bakılarak yapılır.

```cadence
METHODS handle_hotspot_click
     FOR EVENT hotspot_click of CL_GUI_ALV_GRID
       IMPORTING
       E_ROW_ID
       E_COLUMN_ID.
```

İnclude tanımlamamız aynı top of page de yaptığımız şekilde yapabiliriz .Biz bu örneğimiz de tablodaki carrid ve carrname kısımlarına hotspot özelliğini eklemek istiyorum.
Set fcat form içerisin de merge altına aşağıdaki içeriği yazalım.
Field symbol olarak tanımladığımız içerikte fieldname kısmı hotspot olmasını istediğimiz alanın ismini yazalım. Bu alanlar için hotspot özelliğini aktif edelim.

```cadence
LOOP AT GT_FCAT ASSIGNING <GFS_FCAT>.
    IF <GFS_FCAT>-FIELDNAME EQ 'CARRID' OR
       <GFS_FCAT>-FIELDNAME EQ 'CARRNAME'.
      <GFS_FCAT>-HOTSPOT = ABAP_TRUE.
          ENDIF.
    ENDLOOP.
```

Display alv içerisine hotspot event eklemek için ;

```cadence
CREATE OBJECT GO_EVENT_RECEIVER.
SET HANDLER GO_EVENT_RECEIVER->HANDLE_HOTSPOT_CLICK FOR GO_ALV.
```

Daha sonra diğer örnekteki gibi de set table for first display methodu kullanılır. Diğer örneklerdeki aynı şekil de kullanılır.

Sonra da bizim tıklanabilir olarak yaptığımız sütunlara tıklanıldığı zaman ekrana bir mesaj yazmasını istiyoruz.
Bu işlem için class defination kısmın da tanımladığımız hotspot işlevsellik kazandırmak için de implementation kısmında aşağıdaki şekilde yapalım.

Mesajı ekrana basabilmek için bir değişken tanımlayalım.
Read table ile internal tablomuzun içerisin de structure yardımı ile tıklanan alanın index  ile hangi satıra geldiğini bulup subrc 0 yani o satır hangi satır olduğunu bulduysa ,tıklanan sütunun fieldname hotspot özelliğini verdiğimiz sütun ismi ile aynı  olup olmadığına bakalım. Bu işlemi de case ile yapalım. Concatenate ile de istediğimiz içerikleri ekleyerek mesaj olarak ekrana yazalım.

```cadence
 METHOD handle_hotspot_click.
    DATA : LV_MEES TYPE CHAR200.
    READ TABLE GT_SCARR INTO GS_SCARR INDEX E_ROW_ID-INDEX.
    IF SY-SUBRC EQ 0.
      CASE E_COLUMN_ID-FIELDNAME.
        WHEN 'CARRID'.
          CONCATENATE 'Tıklanan Kolon' E_COLUMN_ID-FIELDNAME 'degeri'
          GS_SCARR-CARRID INTO LV_MEES SEPARATED BY space.

          MESSAGE LV_MEES TYPE 'I'.

        WHEN 'CARRNAME'.
          CONCATENATE 'Tıklanan Kolon' E_COLUMN_ID-FIELDNAME 'degeri'
       GS_SCARR-CARRNAME INTO LV_MEES SEPARATED BY space.

          MESSAGE LV_MEES TYPE 'I'.
      ENDCASE.
    ENDIF.
  ENDMETHOD.
```

### DOUBLE CLICK
OO ALV de ekrana basılan bir tablodan herhangi bir alanına tıklanınca ekrana yazmak istediğimiz içerik görünmesi  sağlanır. Bunun için event ler için ortak olan alanları diğer eventler yapılan içerikler aynı olduğu için orayı geçelim.
Oluşturduğumuz class ismi yazıp sonra da o class daki method çağıralım ve cl guı alv grid den referans alarak oluşturduğumuz go alv çağıralım.

```cadence
CREATE OBJECT GO_EVENT_RECEIVER.
   SET HANDLER GO_EVENT_RECEIVER->HANDLE_DOUBLE_CLICK FOR GO_ALV.

```
Kendi oluşturduğumuz class defination kısmında istediğimiz isimle oluşturarak cl gui alv grid den parametrelerine bakıp yazalım.

```cadence
METHODS handle_double_click
  FOR EVENT double_click of CL_GUI_ALV_GRID
    IMPORTING
       E_ROW
       E_COLUMN
       ES_ROW_NO.
```
Oluşturduğumuz classın implementation kısmın da yukarıda oluşturduğumuz metoda işlevsellik kazandıralım .Ekrana mesaj basacağımız için bir değişken tanımlayalım. Read table ile tıklanan içeriğin hangi satırda olduğuna bakalım. Concatenate ile de ekrana yazmak istediğimiz içerikleri boşluklu bir şekilde mesaj olarak görünmesi sağlanır. Biz double click de tablonun tüm alanlarına uyguladığımız için bu içeriği case ile kontrol etmemiz gerekmez.

```cadence
 METHOD handle_double_click.
    DATA LV_MESS2 TYPE CHAR200.

    READ TABLE GT_SCARR INTO GS_SCARR INDEX E_ROW-INDEX.
    IF sy-subrc EQ 0.
      CONCATENATE 'Tıklanan kolon   ' E_COLUMN-FIELDNAME
      ', structure ın degeri ,' GS_SCARR INTO LV_MESS2 SEPARATED BY space.
      MESSAGE LV_MESS2 TYPE 'I'.
    ENDIF.
  ENDMETHOD .
```

### DATA CHANGED

Oo ALV de ekrana basılan tabloda değiştirildikten sonra içeriği ve değiştirilmeden önceki içeriği bakılabilir.

```cadence
CREATE OBJECT GO_EVENT_RECEIVER.
  SET HANDLER GO_EVENT_RECEIVER->HANDLE_DATA_CHANGED FOR GO_ALV.
```
 Diğer event lerde yaptığımız gibi set handle methodu ve go alv kullanılır.
Oluşturduğumuz class içerisine data changed için method oluşturalum ve cl gui alv grid den bu içeriğin parametrelerini bulup yerleştirelim.

```cadence
METHODS handle_data_changed
    FOR EVENT data_changed of CL_GUI_ALV_GRID
    IMPORTING
       ER_DATA_CHANGED
       E_ONF4
       E_ONF4_BEFORE
       E_ONF4_AFTER
       E_UCOMM.
```

Tanımladığımız class ın implementation kısmında aşağıdaki şekilde tanımlayalım.

```cadence
METHOD handle_data_changed.

    DATA: LVC_M TYPE LVC_S_MODI,
          LV_ME TYPE CHAR200.

    LOOP AT ER_DATA_CHANGED->MT_GOOD_CELLS INTO LVC_M.
      READ TABLE GT_SCARR INTO GS_SCARR INDEX LVC_M-ROW_ID.
      IF SY-SUBRC EQ 0.
        CASE LVC_M-FIELDNAME.
          WHEN 'CURRCODE'.
            CONCATENATE LVC_M-FIELDNAME
                     'Eski degeri'
                     GS_SCARR-CURRCODE
                     'Yeni değeri'
                     LVC_M-VALUE
                     INTO LV_ME
                     SEPARATED BY space.
            MESSAGE LV_ME TYPE 'I'.
          WHEN 'URL'.
            CONCATENATE LVC_M-FIELDNAME
                      'Eski degeri'
                      GS_SCARR-URL
                      'Yeni değeri'
                      LVC_M-VALUE
                      INTO LV_ME
                      SEPARATED BY space.
            MESSAGE LV_ME TYPE 'I'.
        ENDCASE.

      ENDIF.
    ENDLOOP.

  ENDMETHOD.
```

### SEARCH HELP
Tanımladığımız class da definition kısmı aşağıdaki şekilde yapılır. Burada ki importing içerikleri cl gui alv grid den search help parametreleridir.

```cadence
CREATE OBJECT GO_EVENT_RECEIVER.
  SET HANDLER GO_EVENT_RECEIVER->HANDLE_ONF4 FOR GO_ALV.

 METHODS handle_onf4
        FOR EVENT onf4 of CL_GUI_ALV_GRID
        IMPORTING
       E_FIELDNAME
       E_FIELDVALUE
       ES_ROW_NO
       ER_EVENT_DATA
       ET_BAD_CELLS
       E_DISPLAY.
```
Daha sonra da implementation kısmında aşağıdaki şekilde yazılır .Biz search help url sütununa ekleyeceğimiz için url data elementi yazılır.

Lt_value tab search help de  içerisine yazdığımız ve ekranda açıldığında seçeceğimiz verilerin görünmesini sağlayan internal tablodur. Bu internal tablonun structure da oluşturalım.

Lt return ile structure ve internal tablosu oluşturur .Bu tablonun içeriği  fieldname kısmı vardır bu kısım da ilk seçtiğimiz içerik  f0001 değerini verilir .Daha sonra da bu birer birer artarak devam eder. Bizim search help ekranında seçtiğimiz içerik ve f0001 li isim 

Search help de görünmesini istediğimiz içerikleri  ls_value_tab  url sütununa  ekleyelim.
Sonra da f4if fonksiyonu çağıralım. Retfield  de hangi sütun için kullanılacaksa onun ismi yazılır.
Value Org kısmına da celll ve structure yazılabilir , biz örneğimiz de structure kullanacağımız için de S ile yazılır.
 Sonra da read table ile return internal tablomuzun fieldname f0001 olan ve subrc 0 ise yani böyle bir değer bulduysa read table gt_scarr içeriği alv ilk açıldığında olan içerikleri yani search help ekranı açıldıktan sonra seçilen içerik değil scarr ilk hali yazılır.
Daha sonra da scarr da oluşturduğumuz  yani seçtiğimiz değerin ekrana yazılması için  ls_return fieldval yani seach help de seçtiğimiz kısmını url sütuna atarız.
Yaptığımız bu değişiklik sistem de değişse bile ekrana önceden alv basıldığı için bu alv refreshlemiz gerekmektedir.

```cadence
METHOD handle_onf4.

    TYPES: BEGIN OF LTY_VALUE_TAB,
      URL TYPE S_CARRURL,
      END OF LTY_VALUE_TAB.

    DATA: LT_VALUE_TAB TYPE TABLE OF LTY_VALUE_TAB,
          LS_VALUE_TAB TYPE LTY_VALUE_TAB.

    DATA: LT_RETURN_TAB TYPE TABLE OF DDSHRETVAL,
          LS_RETURN_TAB TYPE  DDSHRETVAL.

    CLEAR LS_VALUE_TAB.
    LS_VALUE_TAB-URL = 'Türk hava yolları'.
    APPEND LS_VALUE_TAB TO LT_VALUE_TAB.

    CLEAR LS_VALUE_TAB.
    LS_VALUE_TAB-URL = 'Anadolu Jet 2'.
    APPEND LS_VALUE_TAB TO LT_VALUE_TAB.

    CALL FUNCTION 'F4IF_INT_TABLE_VALUE_REQUEST'
      EXPORTING
        RETFIELD     = 'URL'
        WINDOW_TITLE = 'URL F4'
        VALUE_ORG    = 'S'
      TABLES
        VALUE_TAB    = LT_VALUE_TAB
        RETURN_TAB   = LT_RETURN_TAB.

    READ TABLE LT_RETURN_TAB INTO LS_RETURN_TAB
    WITH KEY FIELDNAME = 'F0001'.
    IF SY-SUBRC EQ 0.
      READ TABLE GT_SCARR ASSIGNING <GFS_SCARR> INDEX ES_ROW_NO-ROW_ID.
      IF SY-SUBRC EQ 0.
        <GFS_SCARR>-URL = LS_RETURN_TAB-FIELDVAL.
        GO_ALV->REFRESH_TABLE_DISPLAY( ).
      ENDIF.

    ENDIF.

    ER_EVENT_DATA->M_EVENT_HANDLED = 'X'.
  ENDMETHOD.

FORM REFISTER_F4 .
  DATA :lt_f4 TYPE LVC_T_F4,
        ls_f4 TYPE LVC_S_F4.

CLEAR ls_f4.
ls_f4-FIELDNAME = 'URL'.
ls_f4-REGISTER = ABAP_TRUE.
APPEND ls_f4 TO LT_F4.
CALL METHOD GO_ALV->REGISTER_F4_FOR_FIELDS
  exporting
    IT_F4 = lt_f4
  .
ENDFORM.   

```
