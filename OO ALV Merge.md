## OO ALV Merge

OO ALV Merge kullanımını anlamak için bir örnek yapalım.
İlk olarak include oluşturalım .Include sıralamamız şu şekilde olmalıdır. Top yani data olarak tanımladığımız yer, pbo ekran açılmadan çalışan kısım, pai kullanıcı herhangi bir işlem yaptığı zaman class form ile class yaptığımız kısımla ilgilidir. Start of selection ile de class içinde oluşturduğumuz formları perform ile çağıralım.
Call screen ile oluşturduğumuz ekranın isminin yazarak çağıralım.

```cadence
INCLUDE ZP_P095_I003. "TOP
INCLUDE ZP_P095_I001. "PBO
INCLUDE ZP_P095_I002. "PAI
INCLUDE ZP_P095_I004. "CLASS

START-OF-SELECTION.

CREATE OBJECT OBJ_ALV_OO.
call METHOD OBJ_ALV_OO->GET_DATA.
call METHOD OBJ_ALV_OO->SHOW ALV.

call screen 0100.
```

İlk include yani data tanımladığı içeriğine bakalım.

```cadence
DATA: GO_ALV  TYPE REF TO CL_GUI_ALV_GRID,
      GO_CONT TYPE REF TO CL_GUI_CUSTOM_CONTAINER.

TYPES: BEGIN OF GTY_SCARR,
   CARRID     TYPE S_CARR_ID,
   CARRNAME   TYPE S_CARRNAME,
   CURRCODE   TYPE S_CURRCODE,
   URL        TYPE S_CARRURL,
   MESS       TYPE CHAR200,
   LINE_COLOR TYPE CHAR4,
   CELL_COLOR TYPE LVC_T_SCOL,
END OF GTY_SCARR.

DATA: GS_CELL COLOR TYPE LVC_S_SCOL,
      GT_SCARR TYPE TABLE OF GTY_SCARR.

DATA: GT_FCAT TYPE LVC_T_FCAT,
      GS_FCAT TYPE LVC_S_FCAT,
      GS_LAY  TYPE LVC_S_LAYO.

FIELD-SYMBOLS: <GFS_FC>    TYPE LVC_S_FCAT,
               <GFS_SCARR> TYPE GTY_SCARR.

```
CL GUI ALV GRID ile alv yi ekrana basmak için kullanarak referans alarak class oluşturulur .

CL_GUI_CUSTOM_CONTAINER kullanarak container oluşturulur.

Table type oluşturulur. Scarr tablosu ,yeni oluşturduğumuz sütun ve bazı kolon veya sütunları boyayabilmek için de line color ve cell color oluşturulur.

Line color c ile başlayıp rengi, acık veya koyu olması, arka plan veya yazının renklendirmesi 4 alan ile sağladığı için char4 ile tanımlanır.

Cell color istenilen alanı boyadığı için bir table type tipinde oluşturulur. Bu table type structure bakılır. Bu structure type de yeni bir structure oluşturulur.

Tanımladığımız table type tipinde bir internal tablo oluşturulur.

Fieldcat kullanabilmek için hem structure hem de internal tabloya ihtiyaç duyulur.

Layout kullanabilmek için de structure kullanılır.

Field symbols ifadede değiştirilen index modifiy olmadan kullanmadan internal tablonun istenilen içeriğin index yararlanarak değiştirir.O yüzden biz de oluşturduğumuz table type tipinde bir field symbols oluşturalım.

İkinci include olan pbo içeriğine bakalım.Oluşturduğumuz screen den status bu kısma oluşturalım.

```cadence
MODULE STATUS_0100 OUTPUT.
  SET PF-STATUS 'Status100'.
  SET TITLEBAR 'Title100'.
PERFORM display_alv.
ENDMODULE.     
```

Üçüncü include yani pai screen deki kullanıcı yaptığı göre butonlara göre işlemlerin yapılması sağlanır.

```cadence
MODULE USER_COMMAND_0100 INPUT.
CASE sy-ucomm.
  WHEN '&BACK'.
    LEAVE TO SCREEN 0.

ENDCASE.
ENDMODULE.
```

Dördüncü include from ve classlardan oluşur .İlk olarak display alv form oluşturalım .Bu internal tablomuzu alv olarak ekrana basabilmemiz için kullanılır.

Parent screen de oluşturduğumuz container direk almaz container sınıfından nesne oluşturulur. O yüzden de container sınıfından nesne oluşturulur daha sonra container name kısmına screen de oluşturulan container ismi verilir. Daha sonra da oluşturulan bu objenin ismi de parent kısmına verilir.

Daha sonra method çağırılır  ve layout fieldcat ve ekrana çıktı olarak vereceğimiz internal tablomuz gt_scarr yazılır.

```cadence
FORM DISPLAY_ALV .
  CREATE OBJECT GO_CONT
    EXPORTING
      CONTAINER_NAME = 'CC_ALV'.

  CREATE OBJECT GO_ALV
    EXPORTING
      I_PARENT = GO_CONT.

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
ENDFORM.            
```
Dördüncü include eklediğimiz diğer form bakalım. Table type tipin de oluşturduğumuz internal tablomuzun içeriğini dolduralım. Corresponding kullanmamızın sebebi scarr tablosunda olmayan bizim internal tabloya eklediğimiz alanlar olduğu için alan uyuşmazlığı olmaması için yapılmıştır.

Line color ile istediğimiz sütunu boyayabilmek için loop ile internal tablomuzu field symbol ile dolaşalım.Field symbols isteğimiz sütun ismini yazıp içeriği şu olanları field symbol ile boyayalım.

Cell color kullanmak istersek de cell color için oluşturduğumuz structure fname sütun imini yazarak bu bilgilere ait olan field yani alan boyarız append ile de fieldcat ile boyarız.

```cadence
FORM GET_DATA .
  SELECT * from scarr into CORRESPONDING FIELDS OF TABLE  GT_SCARR.

  LOOP AT GT_SCARR ASSIGNING <GFS_SCARR>.
    CASE <GFS_SCARR>-CARRNAME.
      WHEN 'Swiss'.
        <GFS_SCARR>-LINE_COLOR = 'C710'.
      WHEN 'Air Berlin'.
        <GFS_SCARR>-LINE_COLOR = 'C510'.

      WHEN 'Delta Airlines'.
        CLEAR GS_CELL_COLOR.
        GS_CELL_COLOR-FNAME = 'URL'.
        GS_CELL_COLOR-COLOR-COL = '3'.
        GS_CELL_COLOR-COLOR-INT = '1'.
        GS_CELL_COLOR-COLOR-INV = '0'.
       APPEND GS_CELL_COLOR TO <GFS_SCARR>-CELL_COLOR.

       CLEAR GS_CELL_COLOR.
        GS_CELL_COLOR-FNAME = 'CARRID'.
        GS_CELL_COLOR-COLOR-COL = '4'.
        GS_CELL_COLOR-COLOR-INT = '1'.
        GS_CELL_COLOR-COLOR-INV = '0'.
       APPEND GS_CELL_COLOR TO <GFS_SCARR>-CELL_COLOR.
  ENDCASE.
      ENDLOOP.
    ENDFORM.          
```
Bu include içerinde diğer form bakalım .Merge kullanmamızın amacı ekrana basılan tablonun her sütunun ismini sütun uzunluğunu sırasını tek tek elimizle yazmak istemezsek merge kullanabiliriz.
Structure name kısmı structure table ve view alabilir .Biz bu örneğimizde se11 ile structure oluşturalım ve bu structure ismini verelim.
Field symbol de oluşturduğumuz fieldcat tipinde içeriği kullanalım. Table type eklediğimiz sütunu buraya yazalım.

```cadence
FORM GET_FCAT .
  CALL FUNCTION 'LVC_FIELDCATALOG_MERGE'
    EXPORTING
      I_STRUCTURE_NAME       = 'ZP_SC010'
    CHANGING
      CT_FIELDCAT            = GT_FCAT
    EXCEPTIONS
      INCONSISTENT_INTERFACE = 1
      PROGRAM_ERROR          = 2
      OTHERS                 = 3.
  IF SY-SUBRC <> 0.
  ENDIF.

  READ TABLE GT_FCAT ASSIGNING <GFS_FC> WITH KEY FIELDNAME = 'MESS'.
  IF sy-subrc eq 0.
    <GFS_FC>-EDIT = abap_true.
  ENDIF.
ENDFORM.    
```
Diğer methodumuz bakalım.Lyout ile tüm tablomuzda değişiklik yapabiliriz.Line ve cell color kullanacağımızı bu kısımda gösterilir.

```cadence
FORM SET_LAYOUT .
  CLEAR GS_LAY.
  GS_LAY-CWIDTH_OPT = abap_true.
  GS_LAY-ZEBRA = abap_true.
  GS_LAY-INFO_FNAME = 'LINE_COLOR'.
  GS_LAY-CTAB_FNAME = 'CELL_COLOR'.
ENDFORM.   
```
