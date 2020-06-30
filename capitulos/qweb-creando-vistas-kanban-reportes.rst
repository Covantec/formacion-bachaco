  :banner: banners/qweb.jpg

====
QweB
====



QweB - Creando vistas Kanban y Reportes
=======================================

**QWeb** es un motor de plantillas (*engine template*) por Odoo. Está
basado en XML y es utilizado para generar fragmentos y páginas HTML.
QWeb fue introducido por primera vez en la versión 7.0 para habilitar
vistas Kanban más ricas, y con las versión 8.0, también se usa para la
generación de reportes y páginas web CMS (CMS: Sistemas de Gestión de
Contenido).

Aquí aprenderás acerca de la sintaxis QWeb y como usarla para crear tus
propias vistas Kanban reportes personalizados.

Para entender los tableros Kanban, **Kanban** es una palabra de origen
japonés que es usada para representar un método de gestión de colas
(queue) de trabajo. Fue inspirado del Sistema de Producción y
Fabricación Ligera (lean) de Toyota, y se ha vuelto popular en la la
industria del software con su adopción en las metodologías Ágiles.

El **tablero Kanban** es una herramienta para visualizar la cola de
trabajo. Los elementos de trabajo están representados por
tarjetas que son organizadas en columnas representando las **etapas**
(stages) del proceso de trabajo. Nuevos elementos de trabajo inician en
la columna más a la izquierda y viaja a través del tablero hasta que
alcanzan la columna más a la derecha, representando el trabajo
completado.


Iniciándose con el tablero Kanban
---------------------------------

La simplicidad y el impacto visual del tablero Kanban los hace excelente
para soportar procesos de negocio simples. Un ejemplo básico de un
tablero Kanban puede tener tres columnas, como se muestra en la
siguiente imagen: ToDo "Por hacer", Doing "Haciendo" y Done "Hecho", pero,
por supuesto puede ser extendido a cualquier paso de un proceso específico
que necesite:

.. figure:: images/280_1.jpg
  :align: center
  :alt: Gráfico 8.1 - tablero Kanban

  Gráfico 8.1 - tablero Kanban

Las vistas Kanban una característica distintiva de Odoo, haciendo fácil
implementar estos tableros. Aprenda a cómo usarlos.


Vistas Kanban
-------------

En las vistas de formulario, usa mayormente elementos XML
específicos, tales como ``<field>`` y ``<group>``, y algunos elementos
HTML, tales como ``<h1>`` o ``<div>``. Con las vistas Kanban, es un poco
lo opuesto; ellas son plantillas basadas en HTML y soportan solo dos
elementos específicos de Odoo, ``<field>`` y ``<button>``.

El HTML puede ser generado dinámicamente usando el motor de plantilla
Qweb. Éste procesa los atributos de etiqueta especiales en los elementos
HTML para producir el HTML final para ser presentado por el cliente web.
Esto proporciona mucho control sobre cómo renderizar el contenido, pero
también permite hacer diseños de vistas más complejas.

Las vistas Kanban son tan flexibles que pueden haber muchas formas
diferentes de diseñarlas, y puede ser difícil proveer una receta para
seguir. Una buena regla general es encontrar un vista Kanban existente
similar a lo que querrá alcanzar, y crear su nuevo trabajo de
vista Kanban basada en ella.

Observando las vistas Kanban usadas en los módulos estándar, es posible
identificar dos estilos de vistas Kanban principales: viñeta y tarjeta

Ejemplos de las vistas Kanban de estilo **viñeta** pueden ser
encontrados en **Clientes, Productos**, y también, **Aplicaciones y
Módulos**. Ellos usualmente no tienen borde y son decorados con imágenes
en el lado de la izquierda, tal como se muestra en la siguiente imagen:

.. figure:: images/281_1.jpg
  :align: center
  :alt: Gráfico 8.2 - Ejemplo de vistas Kanban tipo Viñeta

  Gráfico 8.2 - Ejemplo de vistas Kanban tipo Viñeta

El estilo **tarjeta** Kanban es usualmente usada para mostrar tarjetas
organizadas en columnas para las etapas de procesos. Ejemplo de esto son
las **Oportunidades CRM y las Tareas de Proyectos**. El contenido
principal es mostrado en el área superior de la tarjeta y la información
adicional puede ser mostrada en las áreas inferior derecha e inferior
izquierda, tal como se muestra en la siguiente imagen:

.. figure:: images/281_2.jpg
  :align: center
  :alt: Gráfico 8.3 - Ejemplo de estilo de tarjeta Kanban

  Gráfico 8.3 - Ejemplo de estilo de tarjeta Kanban

Vera el esqueleto y elementos típicos usados en ambos estilos de
vistas tal que puedas sentirte cómodo adaptándolos a tus casos de usos
particular.


Diseña vistas Kanban
--------------------

La primera cosa es crear un nuevo módulo agregando sus vistas
Kanban a la lista de tareas por hacer. En un trabajo del mundo real, una
situación de uso de un módulo para esto podría ser, probablemente,
excesiva y ellas podrían ser perfectamente agregadas directamente en el
módulo ``todo_ui``. Pero para una explicación más clara, usara un nuevo
módulo y evitara demasiados, y posiblemente confusos, cambios en
archivos ya creados. Lo nombrara ``todo_kanban`` y creara los
archivos iniciales tal como sigue:

.. code-block:: console

    $ cd ~/odoo-dev/custom-addons
    $ mkdir todo_kanban 
    $ touch todo_kanban/__init__.py

Ahora, edita el archivo descriptor ``todo_kanban/__opernerp__.py`` tal
como sigue:

.. code-block:: python

    {
      'name': 'To-Do Kanban',
      'description': 'Kanban board for to-do tasks.',
      'author': 'Daniel Reis',
      'depends': ['todo_ui'],
      'data': ['todo_view.xml']
    }

A continuación, cree el archivo XML donde irán sus nuevas y brillantes
vistas Kanban y configurar Kanban como la vista por defecto en la
acción ``action`` de ventana de la aplicación *tareas por hacer*, tal como
se muestra a continuación:

.. code-block:: xml

    <?xml version="1.0"?>
    <openerp>
        <data>
            <!-- Agrega el modo de vista kanban al menu Action: -->
        <act_window id="todo_app.action_todo_task" name="To-Do Tasks"  res_model="todo.task" view_mode="kanban,tree,form,calendar,gantt,graph" context="{'search_default_filter_my_tasks':True}" />
            <!-- Agregar vista kanban -->
              <record id="To-do Task Kanban" model="ir.ui.view">
                <field name="name">To-do Task Kanban</field>
                <field name="model">todo.task</field>
                <field name="arch" type="xml">
                   <!-- vacío por ahora, pero el Kanban irá aquí! -->
                </field>
             </record></data>
    </openerp>

Ahora tiene ubicado el esqueleto básico para su módulo. Las
plantillas usada en las vistas ``kanban`` y los reportes son extendidos
usando las técnicas regulares usadas para otras vistas, por ejemplos
usando expresiones XPATH. Para más detalles, ve al `Capítulo 3 <herencia-extendiendo-funcionalidad-aplicaciones-existentes.rst>`_, Herencia – Extendiendo Aplicaciones
Existentes.

Antes de iniciar con las vistas kanban, necesita agregar un para de
campos en el modelo de la aplicación *tareas por hacer*.


Prioridad y estado Kanban
-------------------------

Los dos campos que son frecuentemente usados en las vistas ``kanban`` son:
priority y kanban state.

- **Priority** permite a los usuarios organizar sus elementos de trabajo,
  señalando lo que debería estar ubicado primero.

- **Kanban state** señala cuando una tarea está lista para pasar a la siguiente
  etapa o si es bloqueada por alguna razón. Ambos son soportados por campos
  ``selection`` y tienen widgets específicos para ser usados en las vistas de
  formulario y kanban.

Para agrega estos campos a su modelo, agregara al archivo ``todo_kanban/todo_task.py``,
tal como se muestra a continuación:

.. code-block:: python

    from openerp import models, fields

    class TodoTask(models.Model):
        _inherit = 'todo.task'

        priority = fields.Selection([
                                    ('0','Low'),
                                    ('1','Normal'),
                                    ('2','High')],
                                    'Priority',default='1')
        kanban_state = fields.Selection([
                                        ('normal', 'In Progress'),
                                        ('blocked', 'Blocked'),
                                        ('done', 'Ready for next stage')],
                                        'Kanban State', default='normal')


No olvide el archivo ``todo_kanban/__init__.py`` que cargará el código
precedente:

.. code-block:: python

    from . import todo model


Elementos de la vista kanban
----------------------------

La arquitectura de la vista kanban tiene un elemento superior y la
siguiente estructura básica:

.. code-block:: xml

    <kanban>
      <!-- Fields to use in expressions... -->
      <field name="a_field" />
      <templates>
        <t t-name="kanban-box">
          <!-- HTML Qweb template ... -->
        </t>
      </templates>
    </kanban>

El elemento contiene las plantillas para los fragmentos HTML a usar —uno
o más. La plantilla principal a ser usada debe ser nombrada ``kanban-box``.
Otras plantillas son permitidas para fragmentos HTML para se incluido en
la plantilla principal.

Las plantillas usan html estándar, pero pueden incluir etiquetas
``<field>`` para insertar campos del modelo. También pueden ser usadas
algunas directivas especiales de Qweb para la generación dinámica de
contenido, tal como el ``t-name`` usado en el ejemplo previo.

Todos los campos del modelo usados deben ser declarados con una etiqueta
``<field>``. Si ellos son usados solo en expresiones, tiene que
declararlos antes de la sección ``<templates>``. Uno de esos campos se
le permite tener un valor agregado, mostrado en en el área superior de
las columnas ``kanban``. Esto se logra mediante la adición de un atributo
con la agregación a usar, por ejemplo:

.. code-block:: xml

    <field name="effort_estimated" sum="Total Effort" />

Aquí, la suma para el campo de estimación de esfuerzo es presentada en
el área superior de las columnas ``kanban`` con la etiqueta Total Effort.
Las agregaciones soportadas son ``sum``, ``avg``, ``min``, ``max`` y ``count``.

El elemento superior también soporta algunos atributos interesantes:

-  ``default_group_by``: Establece el campo a usar para la agrupación por
   defecto de columnas.

-  ``default_order``: Establece un orden por defecto para usarse en los
   elementos ``kanban``.

-  ``quick_create="false"``: Deshabilita la opción de creación rápida en la
   vista ``kanban``.

-  ``class``: Añade una clase CSS al elemento raíz en la vista ``kanban``
   renderizada.

Ahora de una mirada más de cerca a las plantillas Qweb usadas en
las vistas ``kanban``.

La vista ``kanban`` viñeta

Para las plantillas QWeb de las viñetas kanban, el esqueleto se ve así:

.. code-block:: xml

    <t t-name="kanban-box"/>
        <div class="oe_kanban_vignette">
            <!-- Left side image: -->
            <img class="oe_kanban_image" name="..." >
                <div class="oe_kanban_details">
                    <!-- Title and data -->
                    <h4>Title</h4>
                    <br>Other data <br/>
                    <ul>
                         <li>More data</li>
                    </ul>
               </div>
        </div>
    </t>

Puedes ver las dos clases CSS principales provistas para los ``kanban`` de
estilo viñeta: ``oe_kanban_vignette`` para el contenedor superior y
``oe_kanban_details`` para el contenido de datos.

La vista completa de viñeta ``kanban`` para las tareas por hacer es como
sigue:

.. code-block:: xml

    <kanban>
        <templates>
            <t t-name="kanban-box">
               <div class="oe_kanban_vignette">
                  <img t-att-src="kanban_image('res.partner', 
                                               'image_medium',
                                               record.id.value)"
                       class="oe_kanban_image"/>
                    <div class="oe_kanban_details">
                        <!-- Title and Data content -->
                        <h4><a type="open">
                            <field name="name"/> </a></h4>
                            <field name="tags" />
                              <ul>
                                <li><field name="user_id" /></li>
                                <li><field name="date_deadline"/></li>
                              </ul>
                            <field name="kanban_state" widget="kanban_state_selection"/>
                            <field name="priority" widget="priority"/>
                    </div>
                </div>
            </t>
        </templates>
    </kanban>

Podrá ver los elementos discutidos hasta ahora, y también algunos
nuevos. En la etiqueta , tiene el atributo QWeb especial ``t-att-src``.
Esto puede calcular el contenido ``src`` de la imagen desde un campo
almacenado en la base de datos. Se explicara esto en otras directivas
QWeb en un momento. También podrá ver el uso del atributo especial
``type`` en la etiqueta ``<a>``. Eche un vistazo más de cerca.


Acciones en las vistas Kanban
-----------------------------

En las plantillas Qweb, la etiqueta para enlaces puede tener un atributo
``type``. Este establece el tipo de acción que el enlace ejecutará para que
los enlaces puedan actuar como los botones en los formularios regulares.
En adición a los elementos ``<button>``, las etiquetas ``<a>`` también
pueden ser usadas para ejecutar acciones Odoo.

Así como en las vistas de formulario, el tipo de acción puede ser acción
u objeto, y debería ser acompañado por atributo nombre, que identifique
la acción específica a ejecutar. Adicionalmente, los siguientes tipos de
acción también están disponibles:

-  ``open``: Abre la vista formulario correspondiente.

-  ``edit``: Abre la vista formulario correspondiente directamente en el
   modo de edición.

-  ``delete``: Elimina el registro y remueve el elemento de la vista kanban.

**La vista kanban de tarjeta** El **tarjeta** de ``kanban`` puede ser un poco
más complejo. Este tiene un área de contenido principal y dos
sub-contenedores al pie, alineados a cada lado de la tarjeta. También
podría contener un botón de apertura de una acción de menú en la esquina
superior derecha de la tarjeta.

El esqueleto para esta plantilla se vería así:

.. code-block:: xml

    <t t-name="kanban-box">
        <div class="oe_kanban_card">
            <div class="oe_dropdown_kanban oe_dropdown_toggle">
            <!-- Top-right drop down menu -->
            </div>
            <div class="oe_kanban_content">
                <!-- Content fields go here... -->
                <div class="oe_kanban_bottom_right"></div>
                <div class="oe_kanban_footer_left"></div>
            </div>
        </div>
    </t>

Un **tarjeta** ``kanban`` es más apropiada para las tareas to-do, así que en
lugar de la vista descrita en la sección anterior, mejor debería usar
la siguiente:

.. code-block:: xml

    <t t-name="kanban-box">
        <div class="oe_kanban_card">
            <div class="oe_kanban_content">
                <!-- Option menu will go here! -->
                <h4><a type="open">
                    <field name="name" />
                    </a></h4>
                    <field name="tags" />
                    <ul>
                        <li><field name="user_id" /></li>
                        <li><field name="date_deadline" /></li>
                    </ul>
                    <div class="oe_kanban_bottom_right">
                        <field name="kanban_state" widget="kanban_state_selection"/>
                    </div>
                    <div class="oe_kanban_footer_left">
                        <field name="priority" widget="priority"/>
                    </div>
            </div>
        </div>
    </t>

Hasta ahora ha visto vistas ``kanban`` estáticas, usando una combinación
de HTML y etiquetas especiales (``field``, ``button``, ``a``). Pero podrá tener
resultados mucho más interesantes usando contenido HTML generado
dinámicamente. Vea como podrá hacer eso usando Qweb.


Agregando contenido dinámico Qweb
---------------------------------

El analizador Qweb busca atributos especiales (directivas) en las
plantillas y las reemplaza con HTML generado dinámicamente.

Para las vistas ``kanban``, el análisis se realiza mediante Javascript del
lado del cliente. Esto significa que las evaluaciones de expresiones
hechos por Qweb deberían ser escritas usando la sintaxis Javascript, no
Python.

Al momento de mostrar una vista kanban, los pasos internos son
aproximadamente los siguientes:

-  Obtiene el XML de la plantilla a renderizar.

-  Llama al método de servidor ``read()`` para obtener la data de los
   campos en las plantillas.

-  Ubica la plantilla ``kanban-box`` y la analiza usando Qweb para la
   salida de los fragmentos HTML finales.

-  Inyecta el HTML en la visualización del navegador (el DOM).

Esto no significa que sea exacto técnicamente. Es solo un mapa mental
que puede ser útil para entender como funcionan las cosas en las vistas
kanban.

A continuación explorara las distintas directiva Qweb disponibles,
usando ejemplos que mejorarán su tarjeta ``kanban`` de la tarea to-do.


Renderizado Condicional con t-if
--------------------------------

La directiva ``t-if``, usada en el ejemplo anterior, acepta expresiones
JavaScript para ser evaluadas. La etiqueta y su contenido serán
renderizadas si la condición se evalúa verdadera.

Por ejemplo, en la tarjeta kanban, para mostrar el esfuerzo estimado de
la Tarea, solo si este contiene un valor, después del campo
``date_deadline``, agrega lo siguiente:

.. code-block:: xml

    <t t-if="record.effort_estimate.raw_value > 0">
        <li>Estimate <field name="effort_estimate"/></li>
    </t>

El contexto de evaluación JavaScript tiene un objeto de registro que
representa el registro que está siendo renderizado, con las campos
solicitados del servidor. Los valores de campo pueden ser accedidos
usando el atributo ``raw_value`` o el ``value``:

-  ``raw_value``: Este es el valor retornado por el método de servidor
   ``read()``, así que se ajusta más para usarse en expresiones
   condicionales.

-  ``value``: Este es formateado de acuerdo a las configuraciones de
   usuario, y está destinado a ser mostrado en la interfaz del usuario.

El contexto de evaluación de Qweb también tiene referencias disponibles
para la instancia JavaScript del cliente web. Para hacer uso de ellos,
se necesita una buena comprensión de la arquitectura de cliente web,
pero no podrá llegar a ese nivel de detalle. Para propósitos
referenciales, los identificadores siguientes están disponibles en la
evaluación de expresiones Qweb:

-  ``widget``: Esta es una referencia al objeto widget ``KanbanRecord``,
   responsable por el renderizado del registro actual dentro de la
   tarjeta kanban. Expone algunas funciones de ayuda útiles que podrá
   usar.

-  ``record``: Este es un atajo para ``widget.records`` y provee acceso
   a los campos disponibles, usando notación de puntos.

-  ``read_only_mode``:

-  ``widget``: Esta es una referencia al widget actual `` KanbanRecord``
   objeto, responsable de la representación del registro actual en un
   tarjeta ``kanban``. Expone algunas funciones ``helper`` útiles que
   puede usar.

-  ``record``: Este es un acceso directo para ``widget.records`` y
   proporciona acceso a los campos disponibles, utilizando la notación de
   puntos.

-  ``read_only_mode``: Esto indica si la vista actual está en modo de
   lectura (y no en modo de edición). Es un atajo para ``widget.view.options.read_only_mode``.

-  ``instance``: Esta es una referencia a la instancia completa del
   cliente web.

También es digno de mención que algunos caracteres no están permitidos
dentro expresiones El signo inferior a (``<``) es un caso así. Puedes
usar un negado ``>=`` en su lugar. De todos modos, hay símbolos alternativos
disponibles para operaciones de desigualdad de la siguiente manera:

-  ``lt``: Esto es para *menor que*.

-  ``lte``: Esto es para *menor o igual que*.

-  ``gt``: Esto es para *mayor que*.

-  ``gte``: Esto es para *mayor o igual que*.



Renderinzando valores con t-esc y t-raw
---------------------------------------

Usted ha utilizado el elemento para representar el contenido del campo. Pero
los valores de campo también se puede presentar directamente sin una etiqueta.
La directiva ``t-esc`` evalúa una expresión y representa su valor escapado de
HTML, como se muestra en el seguimiento:

.. code-block:: xml

    <t t-esc="record.message_follower_ids.raw_value" />

En algunos casos, y si se garantiza que los datos de origen sean seguros, la
directiva ``t-raw`` puede se utilizará para representar el valor sin procesar
del campo, sin ningún escape, como se muestra en el siguiente código:

.. code-block:: xml

    <t t-raw="record.message_follower_ids.raw_value" />


Bucle de renderizado con t-foreach
----------------------------------

Un bloque de HTML puede repetirse iterando a través de un bucle. Usted podrá
usar para agregar los avatares de los seguidores de tareas a las tareas que
comienzan por representando solo las ID de socio de la tarea, de la siguiente
manera:

.. code-block:: xml

    <t t-foreach="record.message_follower_ids.raw_value" t-as="rec"/>
      <t t-esc="rec" />;
    </t>

La directiva ``t-foreach`` acepta una expresión JavaScript que evalúa
colección para iterar. En la mayoría de los casos, este será solo el
nombre de un campo de relación *a muchos*. Se utiliza con una directiva
``t-as`` para establecer el nombre que se utilizará para referirse a cada
elemento en la iteración.

En el ejemplo anterior, recorre los seguidores de la tarea, almacenados
en el campo ``message_follower_ids``. Como hay espacio limitado en el tarjeta
Kanban, podría haber usado la función de JavaScript ``slice()`` para limitar
el número de seguidores a mostrar, como se muestra a continuación:

.. code-block:: xml

    t-foreach="record.message_follower_ids.raw_value.slice(0, 3)" 

La variable ``rec`` contiene cada avatar de iteraciones almacenado en la
base de datos. Las vistas Kanban proporcionan una función auxiliar para
generar convenientemente eso: ``kanban_image()``. Acepta como argumentos
el nombre del modelo, el nombre del campo sosteniendo la imagen que quiere
y la ID para recuperar el registro.

Con esto, puede reescribir el bucle de seguidores de la siguiente manera:

.. code-block:: xml

    <div>
      <t t-foreach="record.message_follower_ids.raw_value.slice(0, 3)" t-as="rec">
          <img t-att-src="kanban_image(
                                 'res.partner',
                                 'image_small', rec)"
                class="oe_kanban_image oe_kanban_avatar_smallbox"/>
      </t>
    </div>

Lo usa para el atributo ``src``, pero cualquier atributo puede ser dinámicamente
generado con un prefijo ``t-att-``.

Sustitución de cadenas en atributos con los prefijos ``t-attf-``.

Otra forma de generar dinámicamente atributos de etiqueta es usar cadena
sustitución. Esto es útil para generar partes de cadenas más grandes
dinámicamente, como una dirección URL o nombres de clase CSS.

La directiva contiene bloques de expresión que serán evaluados y reemplazado
por el resultado. Estos están delimitados por ``{{ and }}`` o por ``#{ and }``.
El contenido de los bloques puede ser cualquier expresión JavaScript válida
y puede usar cualquiera de las variables disponibles para las expresiones QWeb,
como registro y widget.

Ahora va a modificar para usar una sub-plantilla. Deberá comenzar agregando
otra plantilla para su archivo XML, dentro del elemento, después del nodo
``<t t-name="kanban-box">``, como se muestra a continuación:

.. code-block:: xml

    <t t-name="follower_avatars">
        <div>
            <t t-foreach="record.message_follower_ids.raw_value.slice(0, 3)" t-as="rec">
            <img t-att-src="kanban_image('res.partner', 'image_small', rec)"
                 class="oe_kanban_image oe_kanban_avatar_smallbox"/>
            </t>
      </div>
    </t>

Llamarlo desde la plantilla principal de ``kanban-box`` es bastante sencillo para
cada uno existe en el valor del llamador al realizar la llamada de sub-plantilla
como sigue:

.. code-block:: xml

    <t t-call="follower_avatars">
        <t t-set="arg_max" t-value="3" />
    </t>

Todo el contenido dentro del elemento ``t-call`` también está disponible para
sub-plantilla a través de la variable mágica ``0``. En lugar del argumento
de las variables, puede definir un fragmento de código HTML que podría insertarse
en la sub-plantilla usando la sintaxis ``<t t-raw="0" />``.



Otras directivas QWeb
=====================

Usted ha revisado las directivas Qweb más importantes, pero hay algunos
más que debe tener en cuenta. Usted ha visto lo básico sobre Vistas
kanban y plantillas QWeb. Todavía hay algunas técnicas que puede utilizar
para brindar una experiencia de usuario más rica a nuestras tarjetas kanban.



Adición de un menú de opciones de la tarjeta Kanban
---------------------------------------------------

Las tarjetas Kanban pueden tener un menú de opciones, ubicado en la parte superior
derecha. Las acciones usuales son para editar o eliminar el registro, pero cualquier
acción invocable desde un el botón es posible. También hay disponible un widget para
configurar la tarjeta.

.. code-block:: xml

        </a>
      </li>
    </t>
    <t t-if="widget.view.is_action_enabled('delete')">
      <li><a type="delete">Delete</a></li>
    </t>
    <!-- Color picker option: -->
    <li>
      <ul class="oe_kanban_colorpicker"
          data-field="color"/>
      </ul>
    </li></div>

Básicamente es una lista HTML de elementos. Las opciones **Editar** y **Eliminar**
usa QWeb para hacerlos visibles solo cuando sus acciones estén habilitadas en el
ver. La función ``widget.view.is_action_enabled`` nos permite inspeccionar si las
acciones de edición y eliminación están disponibles y para decidir qué hacer
disponible para el usuario actual.



Adición de colores para tarjetas Kanban
----------------------------------------

La opción del selector de color permite al usuario elegir el color de una tarjeta
``kanban``. El color se almacena en un campo modelo como un índice numérico.

Debería comenzar agregando este campo al modelo de tareas pendientes, agregando
al archivo ``todo_kanban/todo_model.py`` en la siguiente línea:

.. code-block:: python

    color = fields.Integer('Color Index') 

Aquí usa el nombre habitual para el campo, el color, y esto es lo que es
esperado en el atributo de campo ``data-`` en el selector de color.

A continuación, para que los colores seleccionados con el selector tengan algún
efecto en el tarjeta, debe agregar algunos CSS dinámicos basados en el valor del
campo de color. En la vista ``kanban``, justo antes de la etiqueta, también debe
declarar el color campo, como se muestra a continuación:

.. code-block:: xml

    <field name="color" />

Y, necesita reemplazar el elemento superior de la tarjeta kanban,

.. code-block:: html

    <div class="oe_kanban_card">

con lo siguiente:

.. code-block:: xml

    <div t-attf-class="oe_kanban_card
                       #{kanban_color(record.color.raw_value)}"/>

La función auxiliar ``kanban_color`` hace la traducción del índice de
color al nombre de la clase CSS correspondiente.

Y eso. Una función auxiliar para esto está disponible en vistas ``kanban``.

Por ejemplo, para limitar nuestros títulos de tareas pendientes a los primeros
32 caracteres, debe reemplazar el elemento con lo siguiente:

.. code-block:: xml

    <t t-esc="kanban_text_ellipsis(record.name.value, 32)" />


Archivos CSS y JavaScript personalizados
----------------------------------------

Como usted ha visto, las vistas ``kanban`` son principalmente HTML y hacen
un uso intensivo de clases CSS. Usted ha estado introduciendo algunas clases
CSS de uso frecuente proporcionado por el producto estándar. Pero para obtener
mejores resultados, los módulos también pueden agregar su propio CSS.

Usted no va a entrar en detalles aquí sobre cómo escribir CSS, pero funciona,
dado que no tiene HTML en PDF. Probablemente no sea lo que obtendrá ahora
su sistema. Deje mostrar usted necesita ``Wkhtmltopdf`` para imprimir un pdf
versión de la biblioteca de tiempo de informes

-  ``user``: Este es el registro del usuario que ejecuta el informe.

-  ``res_company``: Este es el registro para el usuario actual. Diseño del
   Interfaz de usuario, con un widget adicional para configurar el widget
   a usar para representar el campo.

Un ejemplo común es un campo monetario, como se muestra a continuación:

.. code-block:: xml

    <span t-field="o.amount"
          t-field-options='{
                   "widget": "monetary",
                   "display_currency": "o.pricelist_id.currency_id"}'/>

Un caso más sofisticado es el widget de contacto, utilizado para formatear
direcciones, como se muestra a continuación:

.. code-block:: xml

    <div t-field="res_company.partner_id" t-field-options='{
            "widget": "contact",
            "fields": ["address", "name", "phone", "fax"],
                    "no_marker": true}' />

Por defecto, algunos pictogramas, como un teléfono, se muestran en la dirección.
La opción ``no_marker="true"`` los desactiva.



Habilitando la traducción de idiomas en reportes
------------------------------------------------

Una función auxiliar, ``translate_doc()``, está disponible para dinámicamente
traducir el contenido del informe a un idioma específico.

Necesita el nombre del campo donde se puede encontrar el idioma a utilizar.
Con frecuencia será el Socio (Partner) al que se enviará el documento,
generalmente almacenado en ``partner_id.lang``. En su caso, también tiene un
método menos eficiente.

Si puede ganar importancia en el conjunto de herramientas Odoo. Finalmente tuviste
una descripción general sobre cómo crear informes, también utilizando el motor QWeb.



Resumen
=======

En el **capítulo 8**, usted aprendió a trababar con los reportes QWeb.
En el siguiente capítulo, explorara cómo aprovechar la API RPC para
interactuar con Odoo desde aplicaciones externas.
