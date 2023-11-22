## OO ALV Icon Cell

İlk olarak types icon göstermek için yeni bir sütun oluşturalım.
Kullanacağımız icon  type pool ile icon da tün iconlar bulunmaktadır.

```cadence
TYPE-POOLs icon.
TYPES: BEGIN OF gty_scarr,
icond TYPE icon_d,
```

Field cat kısmına yeni oluşturduğumuz icon sütununu eklemek için aşağıdaki işlem yapılır.

```cadence
CLEAR GS_FCAT.
  GS_FCAT-FIELDNAME = 'ICOND'.
  GS_FCAT-SCRTEXT_S = 'Icon'.
  GS_FCAT-SCRTEXT_M = 'Icon'.
  GS_FCAT-SCRTEXT_L = 'Icon'.
  GS_FCAT-COL_OPT = ABAP_TRUE.
  APPEND GS_FCAT TO GT_FCAT.
```

PAI include perform oluşturalım ve icon ekran açıldığında ve save butonuna tıklayınca açılabilmesi için bu işlemi yapalım. 

Oluşturduğumuz bu perform içeriğini form ile dolduralım.
Satırdaki değerlerin toplamı ,satır sayısı , değerlerin ortalamasını bulan değişkenler tanımlanmıştır.
Loop ile de istenilen sütundaki değerlerin toplamı bulunmaktadır.
Describe table ile tablomuzdaki satır sayısını bulunuruz.
Daha sonra da loop ile field symbols kullanıldığı için assigning kullanılır.
Loop ile  de internal tablomuzda  cost sütunundaki değerlere bakarız. Eğer cost değerimiz ortalamadan büyük bu ürünün fiyatı ortalamadan yüksek olduğunu ve  kırmızı şeklindeki icon çıkar .Eşit ise sarı icon, küçük ise de yeşil icon çıkar. Biz de bu işlemi go alv deki REFRESH_TABLE_DISPLAY metodu ile yaparız.

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

Biz bu iconların görünebilmesi için ekranda açılan ve alv basılı olan tabloda editable için tanımladığımız satıra değerler girdikten sonra save basılınca loop içerisinde yaptığımız içerikleri sağlamasına göre  basılı olan internal tablomuzda refreshleme yapılması gerekiyor bu durumu şu şekilde yapalım. Eğer bizim ekrana basacağımız alv boş ise demek ki ilk defa alv basacağız o yüzden container parent  gibi işlemler bu kısımda yapılacaktır .Eğer go alv boş değilse de tablomuz ekrana basılmıştır ama bazı değişiklikler yapılması gerektiğini gösterir.

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
    .

    CALL METHOD GO_ALV->REGISTER_EDIT_EVENT
      EXPORTING
        I_EVENT_ID = CL_GUI_ALV_GRID=>mc_evt_modified.
    .

    if sy-subrc <> 0.

    endif.

  else.
    CALL METHOD go_alv->REFRESH_TABLE_DISPLAY.

  ENDIF.
ENDFORM.
```
