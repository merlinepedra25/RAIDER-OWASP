.. _plugins:
.. module:: raider.plugins.common

Plugins
=======

Plugins in **Raider** are pieces of code that are used to get inputs
from, and put them in the HTTP request, and/or to extract some values
from the response. This is used to facilitate the information exchange
between :ref:`Flows <flows>`. Below there's a list of predefined
Plugins. The users are also encouraged to write their own plugins.


Common
------

Plugin
++++++

.. autoclass:: Plugin

Parser
++++++

.. autoclass:: Parser

Processor
+++++++++

.. autoclass:: Processor

Empty
+++++

.. autoclass:: Empty


.. module:: raider.plugins.basic

Basic
-----

.. _plugin_variable:

Variable
++++++++

The Variable plugin extracts the value of a variable.

.. autoclass:: Variable

Example:

.. code-block:: hylang

   (setv username (Variable "username"))

.. autoclass:: Variable
   :members:	       

.. _plugin_prompt:

Prompt
++++++

The prompt plugin accepts user input mid-flow.

Example:

.. code-block:: hylang

   (setv mfa_code (Prompt "Input code here:"))

.. autoclass:: Prompt
   :members:	       

.. _plugin_cookie:      

Cookie
++++++

The cookie plugin extracts and sets new cookies.

Example:

.. code-block:: hylang

   (setv session_cookie (Cookie "PHPSESSID"))

.. autoclass:: Cookie
   :members:	       
	       

.. _plugin_header:      

Header
++++++

The Header plugin extracts and sets new headers.

Example:

.. code-block:: hylang

   (setv x-header (Header "x-header"))
   (setv y-header (Header "y-header" "y-value"))

   (setv z-header (Header.basicauth "username" "password"))


   (setv access_token
         (Regex
           :name "access_token"
           :regex "\"accessToken\":\"([^\"]+)\""))
      
   (setv z-header (Header.bearerauth access_token))

.. autoclass:: Header
   :members:	       

File
++++

The File plugin sets the plugin's value to the contents of a provided file
and allows string substitution within the content.

Example:

.. autoclass:: File
   :members:	       


.. _plugin_command:

Command
+++++++

The Command plugin runs shell commands and extracts their output. 

Example:

.. code-block:: hylang

   (setv mfa_code (Command
                   :name "otp"
		   :command "pass otp personal/app1"))

.. autoclass:: Command
   :members:	       


.. _plugin_regex:

Regex
+++++

The Regex plugin extracts a matched expression from a provided string.

Example:

.. code-block:: hylang
		
   (setv access_token
         (Regex
           :name "access_token"
           :regex "\"accessToken\":\"([^\"]+)\""))


.. autoclass:: Regex
   :members:	       
	       

.. _plugin_html:      

Html
++++

The Html plugin extracts tags matching attributes specified by the user.

Example:

.. code-block:: hylang
		
    (setv csrf_token
          (Html
            :name "csrf_token"
            :tag "input"
            :attributes
            {:name "csrf_token"
             :value "^[0-9a-f]{40}$"
             :type "hidden"}
            :extract "value"))


.. autoclass:: Html
   :members:	       

.. _plugin_json:
      
Json
++++

The Json plugin extracts fields from JSON tables.

.. autoclass:: Json
   :members:	       

.. module:: raider.plugins.modifiers


Modifiers
---------

Alter
+++++

.. autoclass:: Alter
   :members:	       

Combine
+++++++

.. autoclass:: Combine
   :members:	       


      
.. module:: raider.plugins.parsers

Parsers
-------

UrlParser
+++++++++

.. autoclass:: UrlParser
   :members:	       
      


.. _plugin_api:

Writing custom plugins
----------------------


In case the existing plugins are not enough, the user can write
their own to add the new functionality. Those new plugins should be
written in the project's configuration directory in a ".hy" file. To
do this, a new class has to be defined, which will inherit from
*Raider*'s Plugin class:


Let's assume we want a new plugin that will use `unix password store
<https://www.passwordstore.org/>`_ to extract the OTP from our website.


.. code-block:: hylang


    (defclass PasswordStore [Plugin]
    ;; Define class PasswordStore which inherits from Plugin

      (defn __init__ [self path]
      ;; Initiatialize the object given the path

        (.__init__ (super)
                   :name path
                   :function (. self run_command)))
      ;; Call the super() class, i.e. Plugin, and give it the
      ;; path as the name identifier, and the function
      ;; self.run_command() as a function to get the value.
      ;;
      ;; We don't need the response nor the user data to use
      ;; this plugin, so no flags will be set.
		   
      (defn run_command [self]
        (import os)
	;; We need os.popen() to run the command
	
        (setv self.value
              ((. ((. (os.popen
                        (+ "pass otp " self.path))
                      read))
                  strip)))
	;; set self.value to the output from "pass otp",
	;; with the newline stripped.
	
        (return self.value)))


And we can create a new variable that will use this class:

.. code-block:: hylang

    (setv mfa_code (PasswordStore "personal/reddit"))


Now whenever we use the ``mfa_code`` in our requests, its value will
be extracted from the password store.


      
