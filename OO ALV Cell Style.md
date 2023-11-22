## OO ALV Cell Style

Bu içeriği mesela edit özelliği açık olan sütun bazıları kapatmak için kullanabilir.
Bunu için bir sütun ekleyeceğimiz için types begin of içine ekleyelim.(Cellstyle )

```cadence
TYPES: BEGIN OF GTY_SCARR,
CARRID   TYPE  S_CARR_ID,
CARRNAME TYPE  S_CARRNAME,
CURRCODE TYPE  S_CURRCODE,
URL       TYPE   S_CARRURL,
CELLSTYLE TYPE LVC_T_STYL,
  END OF GTY_SCARR.
```

Bu oluşturduğumuz sütunun tipine çift tıklayarak line kullanarak structure oluşturalım.
Bu eklediğimiz sütun ekranda görünmesini istemediğimiz için fieldcat kısmına yazmıyoruz.

Layout kısmına gelelim oluşturduğumuz sütün ismini stylefname kısmına yazalım.

```cadence
FORM SET_LAYOUT .
  CLEAR GS_LAY.
  GS_LAY-CWIDTH_OPT = abap_true.
  GS_LAY-STYLEFNAME = 'CELLSTYLE'.
  GS_LAY-ZEBRA = abap_true.
ENDFORM.
```
Url sütunu edit yani değiştirilebilir yapmak içn fielcat kısmına edit kısmını abap true yapılır.

```cadence
  CLEAR GS_FCAT.
  GS_FCAT-FIELDNAME = 'URL'.
  GS_FCAT-SCRTEXT_S = 'Havayolu URL.'.
  GS_FCAT-SCRTEXT_M = 'Havayolu URL.'.
  GS_FCAT-SCRTEXT_L = 'Havayolu URL'.
  GS_FCAT-COL_OPT   = ABAP_TRUE.
  GS_FCAT-EDIT = ABAP_TRUE.
```
Daha sonra hangi sütun da ve bu değere sahip olmayanlarda edit kısmı kapatan içeriği yazalım. Field symbol  currcode eşit değilse eur  o zaman oluşturduğumuz cellstyle structure fieldname url yani hangi sütun istiyorsak o sütunun ismi yazılır. Eşit olmayanlar edit kapatabilmek için CL_GUI_ALV_GRID=>MC_STYLE_DISABLED. Kullanılır. Sonra da append ile kaydedilir.

```cadence
 LOOP AT GT_SCARR ASSIGNING <GFS_SCARR>.
    IF <GFS_SCARR>-CURRCODE NE 'EUR'.
      CLEAR GS_CELLSTYLE.
      GS_CELLSTYLE-FIELDNAME = 'URL'.
      GS_CELLSTYLE-STYLE = CL_GUI_ALV_GRID=>MC_STYLE_DISABLED.
      APPEND GS_CELLSTYLE TO <GFS_SCARR>-CELLSTYLE.
    ENDIF.

  ENDLOOP.
```
Örnek
Bir sütunda satırları farklı renk ve yazı tipini değiştirmek istersek 
Bir sütun ekleyeceğimiz için bu sütunu types begin of ile ekleyelim.

```cadence
TYPES: BEGIN OF GTY_SCARR,
CARRID   TYPE  S_CARR_ID,
CARRNAME TYPE  S_CARRNAME,
CURRCODE TYPE  S_CURRCODE,
URL       TYPE   S_CARRURL,
CELLSTYLE TYPE LVC_T_STYL,
  END OF GTY_SCARR.
```
Bu eklediğimiz sütun ekranda görünmesini istemediğimiz için fieldcat kısmına bu eklediğimiz sütunu eklemeyiz.

Bu eklediğimiz sütunun ismini layout stylefname kısmına yazılır.

```cadence
FORM SET_LAYOUT .
  CLEAR GS_LAY.
  GS_LAY-CWIDTH_OPT = abap_true.
  GS_LAY-STYLEFNAME = 'CELLSTYLE'.
  GS_LAY-ZEBRA = abap_true.
ENDFORM.
```
Cellstyle olarak tanımladığımız structure de style kısmı 8 karaketer uzunluğun değer alır.biz '00000001'değerini verirsek loop da döndüğümüz için o sütunun hepsinin içeriğinin rengini aynı yapar. Ama biz bunu istemiyoruz her sütunun farklı renk ve sütun yazı tipinin farklı olmasını istediğimiz için bunun için bir değişken tanımlayalım.


Numeric ve char tipinde iki değişken tanımlayalım. Char sayaç gibi arttırarak değer verelim. Hangi sütunun renklendireceğimizi fieldname kısmına sütunun ismini yazarız.Sonra değerini arttırdığımız değişkeni de style kısmına yazalım. Daha sonra yapılan içeriği append ile kaydedelim.

```cadence
FORM GET_DATA .

  DATA: LV_NUM TYPE N LENGTH 8,
        LV_NUM_C TYPE CHAR8.

  SELECT * from scarr into CORRESPONDING FIELDS OF TABLE GT_SCARR  LOOP AT GT_SCARR ASSIGNING <GFS_SCARR>.
  LV_NUM = LV_NUM_C + 1.
  LV_NUM_C = LV_NUM.

  GS_CELLSTYLE-FIELDNAME = 'CARRNAME'.
  GS_CELLSTYLE-STYLE = LV_NUM.
  APPEND GS_CELLSTYLE TO <GFS_SCARR>-CELLSTYLE.
  ENDLOOP.
ENDFORM.
```
