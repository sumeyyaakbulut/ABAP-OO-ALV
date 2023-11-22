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

