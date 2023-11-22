## OO ALV Editable

Bir sütun da değer değiştirmek için kullanılır. Bunun için ilk olarak types begin of ile oluşturduğumuz table editbable için kullanacagımız sütunu ekleyelim.

```cadence

TYPES: BEGIN OF gty_scarr,
icond TYPE icon_d,
CARRID  TYPE  S_CARR_ID,
CARRNAME type  S_CARRNAME,
CURRCODE type  S_CURRCODE,
URL   type S_CARRURL,
COST TYPE int4,
END OF gty_scarr.
```

Fieldcat de bu oluşturduğumuz sütunu ekleyelim ve edit özelliğini aktif edelim.
```cadence
CLEAR GS_FCAT.
GS_FCAT-FIELDNAME = 'LOCATION'.
GS_FCAT-SCRTEXT_S = 'Konum'.
GS_FCAT-SCRTEXT_M = 'Konum.'.
GS_FCAT-SCRTEXT_L = 'Konum'.
GS_FCAT-COL_OPT = ABAP_TRUE.
GS_FCAT-EDIT = ABAP_TRUE.
GS_FCAT-DRDN_HNDL = 1.
APPEND GS_FCAT TO GT_FCAT.
```

Screen de save butonuna isim verip kullanıcı bu butona tıklayınca aşağıdaki perform açılsın.

```cadence
MODULE USER_COMMAND_0100 INPUT.
CASE sy-ucomm.
  WHEN '&BACK'.
    SET SCREEN 0.
    WHEN '&SAVE'.
   PERFORM GET_TOTAL_SUM.
ENDCASE.
ENDMODULE.  
```
Bu form içeriği şu şekildedir.
Değerleri toplamını tutacak bir değişken tanımlayalım(lv_tt_sum)
Tablomuzda kaç satır olduğunu ,sayısını gösteren değişken tanımlayalım.
Topladığımız değerleri satır sayısına bölünce çıkan değeri tutması için ortalama değişkeni tanımlayalım.

Sonra da loop at ile internal tablo içinde structure yardımı ile dolanalım. Tablodaki tüm const değerlerini toplayarak toplam değişkenine atalım.

Tablomuzda kaç sütun olduğunu bulmak için describe table structure lines ile satır sayısını tutan değişkene atalım.

```cadence
FORM GET_TOTAL_SUM .
  DATA: lv_tt_sum TYPE int4,
        lv_lines TYPE int4,
        lv_avg TYPE int4.

  LOOP AT GT_SCARR INTO GS_SCARR.
    LV_TT_SUM = LV_TT_SUM + GS_SCARR-COST.
  ENDLOOP.

  DESCRIBE TABLE GT_SCARR LINES LV_LINES.

  LV_AVG = LV_TT_SUM / LV_LINES.
  LOOP AT GT_SCARR ASSIGNING <GFS_SCARR>.
    IF <GFS_SCARR>-COST > lv_avg.
      <GFS_SCARR>-ICOND = '@0A@'.

    ELSEIF <GFS_SCARR>-COST < lv_avg.
      <GFS_SCARR>-ICOND = '@08@'.

    ELSE.
      <GFS_SCARR>-ICOND = '@09@'.
    ENDIF.
  ENDLOOP.
ENDFORM.           
```

Daha sonra ekrana alv basmak için go_alv type(CL_GUI_ALV_GRID) oluşturmuştuk
Eğer go alv boş ise yani alv hic ekrana basılmamış demektir.O zaman ilk kez basılacağı için aşağıdaki işlemleri yapabiliriz.
Go ALV deki method kullanarak kullanıcı enter bastıktan sonra save butonuna tıklayınca işlemleri toplayarak ekrana gösterilir.
Yada kullanıcı direk save basınca görebilir ama o zaman da üstünde bulunduğu sütunu değerini toplamaz .Üstünde bulunan sütun hariç hepsini toplar. 

```cadence
IF GO_ALV is INITIAL.
    CREATE OBJECT GO_ALV
      EXPORTING
        I_PARENT = CL_GUI_CONTAINER=>SCREEN0.
    .
    PERFORM set_dropdown.
    call METHOD GO_ALV->SET_TABLE_FOR_FIRST_DISPLAY
     exporting

       IS_LAYOUT                     =     GS_LAY
      changing
        IT_OUTTAB                     =     GT_SCARR
        IT_FIELDCATALOG               =     GT_FCAT
      .

    CALL METHOD GO_ALV->REGISTER_EDIT_EVENT
      EXPORTING
        I_EVENT_ID = CL_GUI_ALV_GRID=>mc_evt_enter.

    CALL METHOD GO_ALV->REGISTER_EDIT_EVENT
      EXPORTING
        I_EVENT_ID = CL_GUI_ALV_GRID=>mc_evt_modified.
```
