## OO ALV Toolbar

Pusbutton yaptığımız projeye devam ederek örnek üzerin de yapalım. Toolbar kısmına buton eklenir.
Bunun için tanımladığımız class da defination kısmına metodu yazalım. Handle toolbar method oluşturalım. Bu oluşturulan metod da toolbar buton oluşturur.
User command ise bu butona tıklandığı zamana çalışan kısımdır.

```cadence
CLASS CL_EVENTS DEFINITION.
  PUBLIC SECTION.
        METHODS HANDLE_TOOLBAR
     FOR EVENT TOOLBAR OF CL_GUI_ALV_GRID
     IMPORTING
     E_OBJECT
    E_INTERACTIVE.

    METHODS HANDLE_USER_COMMAND
         FOR EVENT USER_COMMAND OF CL_GUI_ALV_GRID
         IMPORTING
         E_UCOMM.
ENDCLASS.

```
Class implementation kısmına aşağıdaki gibi yapılır .
Function kısmı butona verilen isim bu kısım da verilir.
 İcon da istediğimiz içeriği ismi kopyalanarak yazılır .Quickinfo butonun üstüne gelince çıkan yazıdır.
Disable false dersek tıklanabilir özelliği açık olur.
Bu işlemleri aynı şekil de kullanarak  diğer butonlar ekleyebiliriz.
User command ile de sy-ucomm daki gibi tıklanan butonun ismini tutan e_ucomm kullanılarak tıklanan butona göre işlem yapılır.

```cadence
METHOD HANDLE_TOOLBAR.
    DATA: LS_TOOLBAR TYPE STB_BUTTON.
    CLEAR LS_TOOLBAR.
    LS_TOOLBAR-FUNCTION = '&DEL'.
    LS_TOOLBAR-TEXT = 'Silme'.
    LS_TOOLBAR-ICON = '@11@'.
    LS_TOOLBAR-QUICKINFO = 'Silme işlemi yapar'.
    LS_TOOLBAR-DISABLED = abap_false.
    APPEND LS_TOOLBAR TO E_OBJECT->MT_TOOLBAR.

     CLEAR LS_TOOLBAR.
    LS_TOOLBAR-FUNCTION = '&DIS'.
    LS_TOOLBAR-TEXT = 'Görüntüle'.
    LS_TOOLBAR-ICON = '@18@'.
    LS_TOOLBAR-QUICKINFO = 'Görüntüleme İşlemi Yapar'.
    LS_TOOLBAR-DISABLED = abap_false.
    APPEND LS_TOOLBAR TO E_OBJECT->MT_TOOLBAR.

  ENDMETHOD.

  METHOD HANDLE_USER_COMMAND.
    CASE E_UCOMM.
      WHEN '&DEL'.
        MESSAGE 'Silme İslemi' TYPE 'I'.
      WHEN '&DIS'.
         MESSAGE 'Görüntüleme İşlemi' TYPE 'I'.
    ENDCASE.
    ENDMETHOD.
```

Form kısmına set handle olarak tanımladığımız method kullanılır.
```cadence
CREATE OBJECT GO_EVENT_RECEIVER.
    SET HANDLER GO_EVENT_RECEIVER->HANDLE_BUTTON_CLICK for GO_ALV.
    SET HANDLER GO_EVENT_RECEIVER->HANDLE_TOOLBAR for GO_ALV.
```
