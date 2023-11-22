## OO ALV DropDown Cell

Dropdown  cell örnek üzerinde anlatalım .Scarr tablomuzda alv alnı bizim elimizle direk doldurmak yerine çıkan listeden seçmemizi sağlar. Biz scarr tablomuzu kullanarak yurtiçi yurtdışı uçuş, koltuk numarası, Ön arka uç gibi  bilgileri üzerinde örnek yapalım.

İlk olarak yeni sütun ekleyeceğimiz için  types kısmına gelelim. Location ve koltuk numarası(seatn) adında iki sütun ekleyelim.

```cadence
TYPES: BEGIN OF gty_scarr,
icond TYPE icon_d,
CARRID  TYPE  S_CARR_ID,
CARRNAME type  S_CARRNAME,
CURRCODE type  S_CURRCODE,
URL   type S_CARRURL,
COST TYPE int4,
LOCATION TYPE CHAR20,
SEATN TYPE CHAR1,
   END OF gty_scarr.
```

Eklediğimiz sütunlar ekranda görünebilmesi için fieldcat kısmına ekleyelim. Bu kısımda drnr hndl kısmına location ile saetn farklı sütun olduğu için de ikisine farklı sayı yazılır.

```cadence
CLEAR GS_FCAT.
  GS_FCAT-FIELDNAME = 'LOCATION'.
  GS_FCAT-SCRTEXT_S = 'Konum'.
  GS_FCAT-SCRTEXT_M = 'Konum.'.
  GS_FCAT-SCRTEXT_L = 'Konum'.
  GS_FCAT-COL_OPT   = ABAP_TRUE.
  GS_FCAT-EDIT      = ABAP_TRUE.
  GS_FCAT-DRDN_HNDL = 1.
  APPEND GS_FCAT TO GT_FCAT.

  CLEAR GS_FCAT.
  GS_FCAT-FIELDNAME = 'SEATN'.
  GS_FCAT-SCRTEXT_S = 'Koltuk Harfi'.
  GS_FCAT-SCRTEXT_M = 'Koltuk Harfi.'.
  GS_FCAT-SCRTEXT_L = 'Koltuk Harfi'.
  GS_FCAT-COL_OPT   = ABAP_TRUE.
  GS_FCAT-EDIT      = ABAP_TRUE.
  GS_FCAT-DRDN_HNDL = 2.
  APPEND GS_FCAT TO GT_FCAT.
```

Daha sonra bir form oluşturalım ve oluşturduğumuz bu form da alv yi ekrana basmadan önce çağıralım.

```cadence
IF GO_ALV is INITIAL.
    CREATE OBJECT GO_ALV
      EXPORTING
        I_PARENT = CL_GUI_CONTAINER=>SCREEN0.
    .
    PERFORM set_dropdown.
    call METHOD GO_ALV->SET_TABLE_FOR_FIRST_DISPLAY
     exporting
```

Set Dropdown form içeriğini yazalım. Type olarak da lvc_t_drop tipinde internal tablo oluşturalım. Daha sonra bu internal tablonun içeriğine bakar line kısmını alarak da structure oluşturalım. Bu içerikler handle ve value değerlerini içerir.

Location sütununa handle kısmına 1 yazmıştık o yüzden burada da 1 yazalım ve ekranda görünmesini istediğimiz içeriği yazalım. Append ile de kaydedelim.Böylelikle handle 1 için ne kadar value girersek location sütununda o kadar değer görürüz.

Seatn yani koltuk pozisyonun handle kısmına 2 vermiştik o kısma da a , b, c ,d ,e ,f value olarak girelim .

Yaptığımız bu içerikleri görebilmek için de go alv (CL_GUI_ALV_GRID) SET_DROP_DOWN_TABLE metodunu kullanalım ve append ile kaydediğimiz drop ismini bu kısma yazalım.

```cadence
FORM SET_DROPDOWN .
  DATA : lt_dropdoen TYPE lvc_t_drop,
         ls_dropdoen TYPE lvc_s_drop.

  LS_DROPDOEN-HANDLE = 1.
  LS_DROPDOEN-VALUE = 'Yurtiçi'.
  APPEND LS_DROPDOEN TO LT_DROPDOEN.

  CLEAR LS_DROPDOEN.
  LS_DROPDOEN-HANDLE = 1.
  LS_DROPDOEN-VALUE = 'Yurtdısı'.
  APPEND LS_DROPDOEN TO LT_DROPDOEN.

  CLEAR LS_DROPDOEN.
  LS_DROPDOEN-HANDLE = 2.
  LS_DROPDOEN-VALUE = 'A'.
  APPEND LS_DROPDOEN TO LT_DROPDOEN.

  CLEAR LS_DROPDOEN.
  LS_DROPDOEN-HANDLE = 2.
  LS_DROPDOEN-VALUE = 'B'.
  APPEND LS_DROPDOEN TO LT_DROPDOEN.

  CLEAR LS_DROPDOEN.
  LS_DROPDOEN-HANDLE = 2.
  LS_DROPDOEN-VALUE = 'C'.
  APPEND LS_DROPDOEN TO LT_DROPDOEN.

  CLEAR LS_DROPDOEN.
  LS_DROPDOEN-HANDLE = 2.
  LS_DROPDOEN-VALUE = 'D'.
  APPEND LS_DROPDOEN TO LT_DROPDOEN.

  CLEAR LS_DROPDOEN.
  LS_DROPDOEN-HANDLE = 2.
  LS_DROPDOEN-VALUE = 'E'.
  APPEND LS_DROPDOEN TO LT_DROPDOEN.

  CLEAR LS_DROPDOEN.
  LS_DROPDOEN-HANDLE = 2.
  LS_DROPDOEN-VALUE = 'F'.
  APPEND LS_DROPDOEN TO LT_DROPDOEN.

GO_ALV->SET_DROP_DOWN_TABLE(
    exporting
      IT_DROP_DOWN       =     LT_DROPDOEN
).
ENDFORM.    
```

## Dynamic DropDown

Bir sütunda her alan için aynı dropdown görünmesini istemiyorsak başka bir kolonun değerine göre dropdown içeriğini değiştirmek istediğimiz zamanlarda kullanılır.

Bunun için de types kısmına uçaktaki pozisyonu belirlemek(ön, arka, kanat) bilgileri için sütun eklenir .Biz dynamic dropdown yapacağımız için de satıra göre değişeceği için de onun için de handl isminde satır yapalım.

```cadence
TYPES: BEGIN OF gty_scarr,
icond TYPE icon_d,
CARRID  TYPE  S_CARR_ID,
CARRNAME type  S_CARRNAME,
CURRCODE type  S_CURRCODE,
URL   type S_CARRURL,
COST TYPE int4,
LOCATION TYPE CHAR20,
SEATN TYPE CHAR1,
SEATP TYPE CHAR10,
HANDL TYPE int4,
   END OF gty_scarr.

```
Bu eklediğimiz iki sütundan sadece seatp kolonunu görünmesini istediğimiz için de fieldcat kısmına sadece bu alanı yazalım. Bu kısımda normal da dropdown daki şekilde DRDN_HNDL = 2.  kullanılmıyor. Çünkü bu şekilde kullansak bu sütunun her her alanın da aynı içerikler görünecektir biz böyle görünmesini istemediğimiz sütundaki alan içeriklerine göre değişsin istediğimiz için şu şekilde kullanalım.
Fieldcat kısmına DRDN_FIELD kısmına hangi sütun olduğunu anlamak için tanımladığımız sütun ismi yazılır.

```cadence
  CLEAR GS_FCAT.
  GS_FCAT-FIELDNAME = 'SEATP'.
  GS_FCAT-SCRTEXT_S = 'Koltuk Poz'.
  GS_FCAT-SCRTEXT_M = 'Koltuk Pozisyonu.'.
  GS_FCAT-SCRTEXT_L = 'Koltuk Pozisyonu'.
  GS_FCAT-COL_OPT   = ABAP_TRUE.
  GS_FCAT-EDIT      = ABAP_TRUE.
  GS_FCAT-DRDN_FIELD = 'HANDL'.
  APPEND GS_FCAT TO GT_FCAT.
```
Daha sonra SET_DROPDOWN form içine
Handle 3 ise ön arka ve kanat, handle 4 ise ön ve arka,
Handle 5 ise sadece kanat alanlarını içerir.
Ekrana set dropdown table ile ekrana yazalım.

```cadence
CLEAR LS_DROPDOEN.
  LS_DROPDOEN-HANDLE = 3.
  LS_DROPDOEN-VALUE = 'Ön'.
  APPEND LS_DROPDOEN TO LT_DROPDOEN.

  CLEAR LS_DROPDOEN.
  LS_DROPDOEN-HANDLE = 3.
  LS_DROPDOEN-VALUE = 'Kanat'.
  APPEND LS_DROPDOEN TO LT_DROPDOEN.

  CLEAR LS_DROPDOEN.
  LS_DROPDOEN-HANDLE = 3.
  LS_DROPDOEN-VALUE = 'Arka'.
  APPEND LS_DROPDOEN TO LT_DROPDOEN.

  CLEAR LS_DROPDOEN.
  LS_DROPDOEN-HANDLE = 4.
  LS_DROPDOEN-VALUE = 'Ön'.
  APPEND LS_DROPDOEN TO LT_DROPDOEN.

  CLEAR LS_DROPDOEN.
  LS_DROPDOEN-HANDLE = 4.
  LS_DROPDOEN-VALUE = 'Arka'.
  APPEND LS_DROPDOEN TO LT_DROPDOEN.

  CLEAR LS_DROPDOEN.
  LS_DROPDOEN-HANDLE = 5.
  LS_DROPDOEN-VALUE = 'Kanat'.
  APPEND LS_DROPDOEN TO LT_DROPDOEN.

  GO_ALV->SET_DROP_DOWN_TABLE(
    exporting
      IT_DROP_DOWN       =     LT_DROPDOEN
).

```
Daha sonra hangi durumlara göre handle 3 , 4 , 5 olacağını bulalım. Loop ile internal tablomuzun içine field symbols ile dolanalım. Structure ile de dolanabiliriz amam o zaman modify kullanmamız gerekir bu işlem daha kısa olduğu bunu tercih ettim.
Sonra currcode eğer eur eşit ise handle 3 görünsün şeklinde yapalım.Bu şekilde currcode kısımlarına göre handle yazılır.

```cadence
 LOOP AT GT_SCARR ASSIGNING <GFS_SCARR>.
    CASE <GFS_SCARR>-CURRCODE.
      WHEN 'EUR'.
        <GFS_SCARR>-HANDL = '3'.
      WHEN 'USD'.
        <GFS_SCARR>-HANDL = '4'.
      WHEN 'AUD'.
        <GFS_SCARR>-HANDL = '5'.
      WHEN OTHERS.
    ENDCASE.
  ENDLOOP.
```
