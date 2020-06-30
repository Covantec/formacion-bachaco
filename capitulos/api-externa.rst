:banner: banners/web_service_api.jpg

===========
API Externa
===========



API Externa – Integración con otros Sistemas
=============================================

Hasta ahora, usted ha estado trabajando con el código del lado del servidor.
Sin embargo, el servidor Odoo también proporciona una API externa, que es
utilizada por su cliente web y también está disponible para otras aplicaciones
cliente.

En este capítulo, aprenderá cómo usar la API externa de Odoo de su propios
programas de clientes HTTP. Para simplificar, se centraran en la clientela
disponible para Python.


Configurar un cliente Python
----------------------------

Se puede acceder a la API de Odoo externamente usando dos (02) protocolos
diferentes: ``XML-RPC`` y ``JSON-RPC``. Cualquier programa externo capaz
de implementar un el cliente para uno de estos protocolos podrá interactuar
con un servidor Odoo. Para evitar introducir lenguajes de programación
adicionales, se seguirá usando Python para explorar la API externa.

Hasta ahora, usted ha estado ejecutando código Python solo en el servidor.
Esta vez, usara Python en el lado del cliente, por lo que es posible que
pueda necesita hacer una configuración adicional en su estación de trabajo.

Para seguir los ejemplos de este capítulo, deberá poder ejecutar archivos
Python en su computadora de trabajo. El servidor Odoo requiere **Python 2**,
pero su cliente ``RPC`` puede estar en cualquier idioma, por lo que **Python 3**
estará bien. Sin embargo, dado que algunos lectores pueden estar ejecutando
el servidor en el mismo máquina en la que están trabajando (¡hola usuarios de
Ubuntu!), será más simple para que todos sigan si sigue usando a **Python 2**.

Si está utilizando *Ubuntu* o *Macintosh*, probablemente **Python** ya esté
instalado Abra una consola de terminal, escriba ``python``, y debería estar
recibido con algo como lo siguiente:

.. code-block:: python

    Python 2.7.8 (default, Oct 20  2014, 15:05:29)
    [GCC 4.9.1] on linux2
    Type "help", "copyright",", "credits" or "license" for more information.
    >>>

.. note::
    Los usuarios de Windows pueden encontrar un instalador y también ponerse
    al día rápidamente. Los paquetes de instalación oficiales se pueden
    encontrar en https://www.python.org/downloads/.


Llamando a la API Odoo usando XML-RPC
=====================================

El método más simple para acceder al servidor es usar ``XML-RPC``. Usted
puede usar la biblioteca ``xmlrpclib`` de la biblioteca estándar de **Python**
para esto. Recuerda que esta programando un cliente para conectarse a un
servidor, entonces necesita una instancia del servidor Odoo ejecutándose
para conectarse. En sus ejemplos, se asumirá que una instancia del servidor
Odoo se está ejecutando en la misma máquina (``localhost``), pero puede usar
cualquier dirección IP o nombre de servidor, si el servidor se está
ejecutando en otra máquina.


Abriendo una conexión XML-RPC
-----------------------------

Usted va a tener un primer contacto con la API externa. Iniciar una consola
de **Python** y escriba lo siguiente:

.. code-block:: python

    >>> import xmlrpclib 
    >>> srv, db = 'http://localhost:8069', 'v8dev' >>> user, pwd = 'admin', 'admin' 
    >>> common = xmlrpclib.ServerProxy('%s/xmlrpc/2/common' % srv)
    >>> common.version()
    {'server_version_info': [8, 0, 0, 'final', 0], 'server_serie': '8.0', 'server_version': '8.0', 'protocol_version': 1} 

Aquí, importa la biblioteca ``xmlrpclib`` y luego la configura algunas
variables con la información para la ubicación del servidor y las credenciales
de conexión. Siéntase libre de adaptarlos a su configuración específica.

A continuación, configura el acceso a los servicios públicos del servidor
(no requiere un inicio de sesión), expuesto en el *endpoint* ``/xmlrpc/2/common``.
Uno de los métodos que están disponibles es ``version()``, que inspecciona la
versión del servidor. Este se usa para confirmar que puede comunicar con el servidor.

Otro método público es ``authenticate()``. De hecho, esto no crea un sesión,
como puede ser llevado a creer. Este método solo confirma que el nombre de usuario
y la contraseña son aceptados y devuelve la identificación de usuario que debe
usarse en solicitudes en lugar del nombre de usuario, como se muestra aquí:

.. code-block:: python

    >>> uid = common.authenticate(db, user, pwd, {}) 
    >>> print uid
    1


Leyendo data desde el servidor
------------------------------

Con ``XML-RPC``, no se mantiene ninguna sesión y la autenticación de
las credenciales se envían con cada solicitud. Esto agrega algo de
sobrecarga al protocolo, pero hace que sea más fácil de usar. A continuación,
configure el acceso a métodos de servidor que necesitan un inicio de sesión
para acceder. Estos están expuestos en el punto final ``/xmlrpc/2/object``,
como se muestra a continuación:

.. code-block:: python

    >>> api = xmlrpclib.ServerProxy('%s/xmlrpc/2/object' % srv) 
    >>> api.execute_kw(db, uid, pwd, 'res.partner', 'search_count', [[]])
    70

Aquí, esta haciendo su primer acceso a la API del servidor, realizando
un conteo con los registros de socios (*Partners*). Los métodos se llaman
usando el método ``execute_kw()`` que toma los siguientes argumentos:

- El nombre de la base de datos a conectarse.

- La conexión ID de usuario.

- La contraseña de usuario.

- El nombre del modelo de destino identificador.

- El método para llamar Una lista de argumentos posicionales.

- Un diccionario opcional con argumentos de palabras clave.

El ejemplo anterior llama al método ``search_count`` del modelo ``res.partner``
con un argumento posicional, ``[]``, y sin argumentos de palabras clave. Los
argumento posicional es un dominio de búsqueda; ya que esta proporcionando una
lista vacía, cuenta todos los socios (*Partners*).

Las acciones frecuentes son ``search`` y ``read``. Cuando se llama desde el ``RPC``,
el método ``search`` devuelve una lista de ID que coinciden con un dominio. El método
de navegación no está disponible desde el ``RPC``, y el método ``read`` debe usarse en
su lugar para, dada una lista de ID de registro, recupere sus datos, como se muestra
en el siguiente código:

.. code-block:: python

    >>> api.execute_kw(db, uid, pwd, 'res.partner', 'search', [[('country_id', '=', 'be'), ('parent_id', '!=', False)]])
    [43,  42] 
    >>> api.execute_kw(db, uid, pwd, 'res.partner', 'read', [[43]], {'fields': ['id', 'name', 'parent_id']})
    [{'parent_id': [7, 'Agrolait'], 'id':43, 'name': 'Michel Fletcher'}]

Tenga en cuenta que para el método ``read``, esta utilizando un argumento
posicional para la lista de ID, ``[43]`` y un argumento de palabra clave,
campos. También puede observar que los campos relacionales se recuperan
como un par, con los ID de registro y nombre para mostrar. Eso es algo a
tener en cuenta cuando procesando los datos en su código.

La combinación de búsqueda y lectura es tan frecuente que un método ``search_read``
se proporciona el método para realizar ambas operaciones en un solo paso.
El mismo resultado ya que los dos pasos anteriores se pueden obtener con
lo siguiente:

.. code-block:: python

    >>> api.execute_kw(db, uid, pwd, 'res.partner', 'search_read', [[('country_id', '=', 'be'), ('parent_id', '!=', False)]], {'fields': ['id', 'name', 'parent_id']}) 

El método ``search_read`` se comporta como leído, pero espera como
primero argumento posicional un dominio en lugar de una lista de ID.
Merece la pena mencionando que el argumento de campo en ``read`` y
``search_read`` no es obligatorio. Si no se proporciona, se recuperarán
todos los campos.


Llamando otros métodos
======================

Todos los métodos de modelo restantes están expuestos a través de ``RPC``,
excepto aquellos que comienzan con ``_`` que se consideran privados. Esto
significa que usted puede usar ``create``, ``write`` y ``unlink`` para
modificar datos en el servidor como sigue:

.. code-block:: python

    >>> api.execute_kw(db, uid, pwd, 'res.partner', 'create', [{'name':'Packt'}])
    75
    >>> api.execute_kw(db, uid, pwd, 'res.partner', 'write', [[75], {'name': 'Packt Pub'}])
    True 
    >>> api.execute_kw(db, uid, pwd, 'res.partner', 'read', [[75], ['id', 'name']])
    [{'id':  75, 'name': 'Packt Pub'}]
    >>> api.execute_kw(db, uid, pwd, 'res.partner', 'unlink', [[75]])
    True

Una limitación del protocolo ``XML-RPC`` es que no admite los valores
``None``. La implicación es que los métodos que no devuelven nada no
ser utilizable a través de ``XML-RPC``, ya que están devolviendo
implícitamente ``None``. Es por eso que los métodos siempre deben terminar
con al menos una declaración de retorno ``True``.

Escribir una aplicación de escritorio de **Notes** haga algo interesante
con la *API RPC*. ¿Qué pasaría si los usuarios pudieran administrar sus
tareas pendientes de Odoo directamente desde el escritorio de su computadora?
Usted va a escribir una aplicación Python simple hacer exactamente eso, como
se muestra en la siguiente captura de pantalla:

.. figure:: images/328_1.jpg
  :align: center
  :alt: Gráfico 9.1 - Cliente Python Tk

  Gráfico 9.1 - Cliente Python Tk

Para mayor claridad, lo divide en dos archivos: uno para interactuar con
el servidor backend, en el archivo ``note_api.py``, y otro con la interfaz
gráfico de usuario, en el archivo ``note_gui.py``.


Capa de comunicación con Odoo
-----------------------------

Cree una clase para configurar la conexión y almacenar su información. Debería
exponer dos métodos:

- El método ``get()`` para recuperar datos de la tarea.

- El método ``set()`` para crear o actualizar tareas.

Seleccione un directorio para alojar los archivos de aplicación y cree el
archivo ``note_api.py``. Puede empezar por agregando el constructor de clase,
de la siguiente manera:

.. code-block:: python

    import  xmlrpclib

    class NoteAPI():

        def __init__(self, srv, db, user, pwd):

            common = xmlrpclib.ServerProxy('%s/xmlrpc/2/common' % srv)
            self.api = xmlrpclib.ServerProxy('%s/xmlrpc/2/object' % srv)
            self.uid = common.authenticate(db, user, pwd, {})
            self.pwd = pwd
            self.db = db
            self.model = 'todo.task' 

Aquí almacena en el objeto creado toda la información necesaria para
ejecutar llamadas en un modelo: la referencia API, ``uid``, ``cpassword``,
``database name`` y el ``model`` a usar. A continuación definirá un método
helper para ejecutar las llamadas. Aprovecha los datos almacenados del objeto
para proporcione una firma de función más pequeña, como se muestra a
continuación:

.. code-block:: python

        def execute(self, method, arg_list, kwarg_dict=None):
            return self.api.execute_kw(
                                       self.db,
                                       self.uid,
                                       self.pwd,
                                       self.model,
                                       method,
                                       arg_list,
                                       kwarg_dict or {}) 

Ahora puede usarlo para implementar los métodos de nivel superior ``get()`` y
``set()``. El método ``get()`` aceptará una lista opcional de ID para recuperar.
Si ninguno está en la lista, todos los registros serán devueltos, como se muestra
aquí:

.. code-block:: python

        def get(self, ids=None):
            domain = [('id', 'in', ids)]
            if ids else []
            fields = ['id', 'name']
            return  self.execute('search_read', [domain, fields]) 

El método ``set()`` tendrá como argumentos el texto de la tarea a escribir,
y un ID opcional. Si no se proporciona ID, se creará un nuevo registro. Eso
devuelve la ID del registro escrito o creado, como se muestra aquí:

.. code-block:: python

        def set(self, text, id=None):
            if id:
                self.execute('write', [[id], {'name': text}])
            else:
                vals = {'name': text, 'user_id': self.uid}
                id = self.execute('create', [vals])
            return id

Termine el archivo con un pequeño fragmento de código de prueba que se ejecutará
si ejecuta el archivo Python:

.. code-block:: python

    if  __name__ == '__main__':
        srv, db = 'http://localhost:8069', 'v8dev'
        user, pwd = 'admin', 'admin'
        api =  NoteAPI(srv, db, user, pwd)
        from pprint import pprint
        pprint(api.get()) 

Si ejecuta el script **Python**, debería ver el contenido de su tareas pendientes
impresas. Ahora que tiene un contenedor simple alrededor de su backend de Odoo,
trate con la interfaz de usuario de escritorio.


Creando la GUI
==============

Su objetivo aquí era aprender a escribir la interfaz entre una aplicación
externo y el servidor Odoo, y esto se hizo en el anterior sección. Pero
sería una pena no ir más allá y, de hecho, poniéndolo a disposición del
usuario final.

Para mantener la configuración tan simple como posible, usara la librería
``Tkinter`` para implementar la interfaz gráfica de usuario. Como es parte
de la biblioteca estándar, no requiere ninguna instalación adicional.
No es el objetivo explicar cómo funciona ``Tkinter``, por lo que faltarán
explicaciones al respecto.

Cada tarea debe tener una pequeña ventana amarilla en el escritorio. Estas
ventanas tendrá un solo widget de texto. Al presionar *Ctrl* + *N* se abrirá
una nueva *Nota*, y presionando *Ctrl* + *S* escribirá el contenido de la
nota actual al servidor Odoo.

Ahora, junto con el archivo ``note_api.py``, cree un nuevo archivo ``note_gui.py``.
Primero importará los módulos y widgets de ``Tkinter`` que usara, y luego
la clase ``NoteAPI``, como se muestra a continuación:

.. code-block:: python

    from Tkinter import Text, Tk
    import tkMessageBox
    from note_api import NoteAPI

A continuación, cree su propio widget de texto derivado del ``Tkinter``.
Cuando al crear una instancia, esperará una referencia de API que se utilizará
para guardar la acción, y también el texto y la ID de la tarea, como se muestra
a continuación:

.. code-block:: python

    class NoteText(Text):
        def __init__(self, api, text='', id=None):
            self.master = Tk()
            self.id = id
            self.api = api
            Text.__init__(self, self.master, bg='#f9f3a9',
                          wrap='word', undo=True)
            self.bind('<Control-n>', self.create)
            self.bind('<Control-s>', self.save)
            if id:
                self.master.title('#%d' % id)
                self.delete('1.0', 'end')
                self.insert('1.0', text)
                self.master.geometry('220x235')
                self.pack(fill='both',  expand=1) 

El método constructor ``Tk()`` crea una nueva ventana de IU y el widget de
texto coloca dentro de él, de modo que crear una nueva instancia de ``NoteText``
automáticamente abre una ventana de escritorio. A continuación, implementara
las acciones ``create`` y ``save``. La acción ``create`` abre una nueva ventana
vacía, pero será almacenado en el servidor solo cuando se realiza una acción
``save``, como se muestra en el siguiente código:


.. code-block:: python

        def create(self, event=None):
            NoteText(self.api, '')

        def save(self,  event=None): 
            text = self.get('1.0', 'end')
            self.id = self.api.set(text,  self.id)
            tkMessageBox.showinfo('Info', 'Note %d Saved.' % self.id) 

La acción ``save`` se puede realizar en tareas existentes o nuevas, pero
no hay necesidad de preocuparse por eso aquí ya que esos casos ya están
manejado por el método ``set()`` de la clase ``NoteAPI``.

Finalmente, agregara el código que recupera y crea todas las ventanas notas
cuando se inicia el programa, como se muestra en el siguiente código:

.. code-block:: python

    if  __name__    ==  '__main__':
        srv, db  = 'http://localhost:8069', 'v8dev'
        user, pwd = 'admin', 'admin'
        api = NoteAPI(srv, db, user, pwd)
        for note in api.get():
            x = NoteText(api, note['name'], note['id'])
            x.master.mainloop() 

El último comando ejecuta ``mainloop()`` en la última ventana de Nota creada,
para iniciar a esperar eventos de ventana.

Esta es una aplicación muy básica, pero el punto aquí es hacer un
ejemplo de formas interesantes de aprovechar la API de Odoo RPC.


Introduciendo al cliente ERPpeek
================================

``ERPpeek`` is a versatile tool that can be used both as an interactive
Command-line Interface (CLI ) and as a Python library , with a more
convenient API than the one provided by ``xmlrpclib``. It is available from
the PyPi index and can be installed with the following:

``ERPpeek`` es una herramienta versátil que se puede utilizar tanto como
una aplicación interactiva de interfaz de línea de comandos (*Command-line Interface - CLI*)
y como biblioteca de **Python**, con más API conveniente que la proporcionada
por ``xmlrpclib``. Está disponible desde el índice PyPi y se puede instalar
con lo siguiente:

.. code-block:: console

    $ pip install -U erppeek

En un sistema Unix, si lo está instalando en todo el sistema, es posible
que necesite anteponer ``sudo`` al comando.


La API ERPpeek
--------------

La biblioteca ``erppeek`` proporciona una interfaz de programación, envolviendo
la biblioteca ``xmlrpclib``, que es similar a la interfaz de programación que
tiene para el código del lado del servidor. Su punto aquí es proporcionar una
idea de lo que ``ERPpeek`` tiene para ofrecer, y no para proporcionar una explicación
completa de todas sus características.

Puede comenzar reproduciendo sus primeros pasos con la biblioteca ``xmlrpclib``
usando ``erppeek`` como lo sigue:

.. code-block:: python

    >>> import  erppeek 
    >>> api = erppeek.Client('http://localhost:8069', 'v8dev', 'admin', 'admin') 
    >>> api.common.version()
    >>> api.count('res.partner', [])
    >>> api.search('res.partner', [('country_id', '=', 'be'), ('parent_id', '!=', False)])
    >>> api.read('res.partner', [43], ['id',  'name', 'parent_id'])

Como puede ver, las llamadas a la API usan menos argumentos y son similares a las
contrapartes del lado del servidor.

Pero ``ERPpeek`` no se detiene aquí, y también proporciona una representación para
*Modelos*. Tiene las siguientes dos formas alternativas de obtener una instancia
para un modelo, ya sea utilizando el método ``model()`` o accediendo a un atributo
en caso de camello:

.. code-block:: python

    >>> m = api.model('res.partner') 
    >>> m = api.ResPartner 

Ahora puede realizar acciones en ese modelo de la siguiente manera:

.. code-block:: python

    >>> m.count([('name', 'like', 'Packt%')])
    1 
    >>> m.search([('name', 'like', 'Packt%')])
    [76] 

También proporciona representación de objetos del lado del cliente para registros como
sigue:

.. code-block:: python

    >>> recs = m.browse([('name', 'like', 'Packt%')]) 
    >>> recs <RecordList 'res.partner,[76]'> 
    >>> recs.name ['Packt'] 

Como puede ver, ``ERPpeek`` recorre un largo camino desde el simple ``xmlrpclib``, y
hace es posible escribir código que se pueda reutilizar del lado del servidor con poco
o sin modificaciones.


El CLI ERPpeek
--------------

No solo se puede usar como una biblioteca de Python, sino que también es una
CLI que se puede usar para realizar acciones administrativas en el servidor.
Donde el comando *odoo shell* proporcionó una sesión interactiva local en el
servidor host, ``erppeek`` proporciona una sesión interactiva remota en un
cliente a través de la red.

Al abrir una línea de comando, puede echar un vistazo a las opciones disponibles,
como se muestra a continuación:

.. code-block:: console

    $ erppeek --help  

Vea una sesión de muestra de la siguiente manera:

.. code-block:: console

    $ erppeek --server='http://localhost:8069' -d v8dev -u admin

    Usage (some commands): models(name)

    # List models matching pattern model(name)
    # Return a Model instance (...)
    Password for 'admin':
    Logged in as 'admin' v8dev
    >>> model('res.users').count()
    3 v8dev
    >>> rec = model('res.partner').browse(43)
    v8dev
    >>> rec.name 'Michel Fletcher'  

Como puede ver, se realizó una conexión con el servidor y la ejecución
del contexto proporcionó una referencia al método ``model()`` para obtener
el modelo instancias y realizar acciones sobre ellos.

La instancia ``erppeek.Client`` utilizada para la conexión también está
disponible a través de la variable cliente. En particular, proporciona
una alternativa a la cliente web para gestionar los siguientes módulos
instalados:

-  ``client.modules()``: Esto puede buscar y enumerar módulos disponibles
   o instalados

-  ``client.install()``: Esto realiza la instalación del módulo

-  ``client.upgrade()``: Esto ordena que los módulos se actualicen

-  ``client.uninstall()``: Esto desinstala módulos

Entonces, ``ERPpeek`` también puede proporcionar un buen servicio como
administración remota herramienta para servidores Odoo.


Resumen
=======

El objetivo para el **capítulo 9** fue aprender cómo funciona la API externa
y de lo que es capaz. Usted inicio a explorarlo usando un simple cliente
``XML-RPC`` en Python, pero la API externa se puede usar desde cualquier
programación idioma. De hecho, los documentos oficiales proporcionan
ejemplos de código para Java, PHP y Ruby.

Hay varias bibliotecas para manejar ``XML-RPC`` o ``JSON-RPC``, algunas
genéricos y algunos específicos para usar con Odoo. No intento señalar
ninguno bibliotecas en particular, a excepción de ``erppeek``, ya que no
es solo un contenedor comprobado para el ``XML-RPC`` *Odoo/OpenERP* pero
porque también es un herramienta invaluable para la gestión e inspección
remota del servidor.

Hasta ahora, utiliza sus instancias de servidor Odoo para desarrollo y pruebas.
Pero para tener un servidor de grado de producción, hay seguridad adicional y
configuraciones de optimización que deben hacerse. En el siguiente capitulo,
Usted se centrara en ellos.
