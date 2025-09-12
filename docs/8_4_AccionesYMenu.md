---
title: 8.4. Acciones y Menús
description: Desarrollando máodulos en Odoo. Acciones y Menús
---

## Acciones y menús

Si quieres conocer en más detalle cómo funcionan las acciones en Odoo, consulta el artículo **Acciones y menús en Odoo**.

El cliente web de **Odoo** contiene menús en la parte superior y a la izquierda. Estos menús, al ser activados, muestran otros menús y las pantallas del programa. Cuando pulsamos en un menú, la pantalla cambia porque hemos ejecutado una acción.

Una acción básicamente tiene:

- **type**: El tipo de acción y cómo debe interpretarse. Cuando se define en XML, no es necesario especificar el tipo, ya que lo indica el modelo en el que se guarda.
- **name**: El nombre, que puede mostrarse o no en pantalla. Se recomienda que sea legible para humanos.

Las acciones y los menús se declaran en archivos de datos XML o directamente si una función devuelve un diccionario que las define. Las acciones pueden ser llamadas de tres maneras:

- Haciendo clic en un menú.
- Haciendo clic en botones de las vistas (deben estar conectados con acciones).
- Como acciones contextuales en los objetos.

De esta forma, el cliente web puede saber qué acción debe ejecutar si recibe alguna de estas cosas:

- **false**: Indica que se debe cerrar el diálogo actual.
- **Una cadena de texto**: Con la etiqueta de la **acción de cliente** a ejecutar.
- **Un número**: Con el ID o ID externo de la acción a buscar en la base de datos.
- **Un diccionario**: Con la definición de la acción, que no está ni en XML ni en la base de datos. Generalmente, se utiliza para llamar a una acción al finalizar una función.

---

### Acciones tipo *window*

Las acciones de tipo *window* son registros del modelo **ir.actions.act_window**. Sin embargo, los menús que las llaman pueden declararse de forma más rápida mediante la etiqueta **menuitem**:

```xml
<record model="ir.actions.act_window" id="action_list_ideas">
    <field name="name">Ideas</field>
    <field name="res_model">idea.idea</field>
    <field name="view_mode">tree,form</field>
</record>

<menuitem id="menu_ideas" parent="menu_root" name="Ideas" sequence="10"
          action="action_list_ideas"/>
```

!!! Tip

    Las acciones deben declararse en el XML antes que los menús que las utilizan.

!!! Example "Ejemplo"

    ``` xml

    <?xml version="1.0" encoding="UTF-8"?>
    <openerp>
        <data>
            <!-- window action -->
            <!--
                The following tag is an action definition for a "window action",
                that is an action opening a view or a set of views
            -->
            <record model="ir.actions.act_window" id="course_list_action">
                <field name="name">Courses</field>
                <field name="res_model">openacademy.course</field>
                <field name="view_type">form</field>
                <field name="view_mode">tree,form</field>
                <field name="help" type="html">
                    <p class="oe_view_nocontent_create">Create the first course
                    </p>
                </field>
            </record>

            <!-- top level menu: no parent -->
            <menuitem id="main_openacademy_menu" name="Open Academy"/>
            <!-- A first level in the left side menu is needed
                before using action= attribute -->
            <menuitem id="openacademy_menu" name="Open Academy"
                    parent="main_openacademy_menu"/>
            <!-- the following menuitem should appear *after*
                its parent openacademy_menu and *after* its
                action course_list_action -->
            <menuitem id="courses_menu" name="Courses" parent="openacademy_menu"
                    action="course_list_action"/>
            <!-- Full id location:
                action="openacademy.course_list_action"
                It is not required when it is the same module -->
        </data>
    </openerp>
    ```

Solo el tercer nivel de menús puede tener asociada una acción. El primer nivel es el menú superior y el segundo no es "clicable".



!!! Tip

    
    Lo que hemos visto en esta sección es la definición de una acción en XML como parte de la vista, pero una acción no es más que una forma cómoda de escribir muchas cosas que hará el cliente en JavaScript para pedir algo al servidor.
    Las acciones separan y simplifican el desarrollo de la interfaz de usuario del cliente web. Un menú o botón en HTML activa una función JavaScript que, en principio, no sabe qué hacer. Esta solicita la definición de su acción. Una vez cargada, queda claro todo lo que debe solicitar (vistas, contexto, dominios, vistas de búsqueda, lugar donde cargarlo todo...). Entonces solicita las vistas y, con ayuda de estas y de los campos, solicita los registros que son los datos a mostrar.
    Por tanto, una acción es la definición —sin programar JavaScript— de lo que debe hacer el cliente. Odoo permite declarar acciones como respuesta de funciones. Estas acciones no están en la base de datos, pero se envían igualmente al cliente, que las trata como si fueran acciones normales. Un ejemplo de esto son las acciones que devuelven los botones de los wizards. De hecho, podemos hacer que un botón devuelva una acción y, por tanto, abra una vista diferente.

