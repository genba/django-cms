#############
Template Tags
#############

****************
CMS templatetags
****************

.. highlightlang:: html+django

To use any of the following templatetags you first need to load them at the
top of your template::

    {% load cms_tags %}

.. templatetag:: placeholder

placeholder
===========
.. versionchanged:: 2.1
    The placeholder name became case sensitive.

The ``placeholder`` templatetag defines a placeholder on a page. All
placeholders in a template will be auto-detected and can be filled with
plugins when editing a page that is using said template. When rendering, the
content of these plugins will appear where the ``placeholder`` tag was.

Example::

    {% placeholder "content" %}

If you want additional content to be displayed in case the placeholder is
empty, use the ``or`` argument and an additional ``{% endplaceholder %}``
closing tag. Everything between ``{% placeholder "..." or %}`` and ``{%
endplaceholder %}`` is rendered in the event that the placeholder has no plugins or
the plugins do not generate any output.

Example::

    {% placeholder "content" or %}There is no content.{% endplaceholder %}

If you want to add extra variables to the context of the placeholder, you
should use Django's :ttag:`with` tag. For instance, if you want to resize images
from your templates according to a context variable called ``width``, you can
pass it as follows::

    {% with 320 as width %}{% placeholder "content" %}{% endwith %}

If you want the placeholder to inherit the content of a placeholder with the
same name on parent pages, simply pass the ``inherit`` argument::

    {% placeholder "content" inherit %}

This will walk up the page tree up until the root page and will show the first
placeholder it can find with content.

It's also possible to combine this with the ``or`` argument to show an
ultimate fallback if the placeholder and none of the placeholders on parent
pages have plugins that generate content::

    {% placeholder "content" inherit or %}There is no spoon.{% endplaceholder %}

See also the :setting:`CMS_PLACEHOLDER_CONF` setting where you can also add extra
context variables and change some other placeholder behavior.


static_placeholder
==================

The static_placeholder templatetag can be used anywhere in any template and is not bound to any page or model.
It needs a name and it will create a placeholder that you can fill with plugins afterwards.
The static_placeholder tag is normally used to display the same content on
multiple locations or inside of apphooks or other 3rd party apps. Static_placeholder need to be published
to show up on live pages.

Example::

    {% load cms_tags %}

    {% static_placeholder "footer" %}


.. warning::

    Static_placeholders are not included in the undo/redo and page history pages


.. templatetag:: show_placeholder

render_placeholder
==================

`{% render_placeholder %}` is used if you have a PlaceholderField in your own model and want
to render it in the template.

The :ttag:`render_placeholder` tag takes the following parameters:

* :class:`~cms.models.fields.PlaceholderField` instance
* ``width`` parameter for context sensitive plugins (optional)
* ``language`` keyword plus ``language-code`` string to render content in the
  specified language (optional)


The following example renders the my_placeholder field from the mymodel_instance and will render
only the english plugins:

.. code-block:: html+django

    {% load cms_tags %}

    {% render_placeholder mymodel_instance.my_placeholder language 'en' %}



show_placeholder
================

Displays a specific placeholder from a given page. This is useful if you want
to have some more or less static content that is shared among many pages, such
as a footer.

Arguments:

* ``placeholder_name``
* ``page_lookup`` (see `page_lookup`_ for more information)
* ``language`` (optional)
* ``site`` (optional)

Examples::

    {% show_placeholder "footer" "footer_container_page" %}
    {% show_placeholder "content" request.current_page.parent_id %}
    {% show_placeholder "teaser" request.current_page.get_root %}

page_lookup
===========

The ``page_lookup`` argument, passed to several templatetags to retrieve a
page, can be of any of the following types:

* :class:`str <basestring>`: interpreted as the ``reverse_id`` field of the desired page, which
  can be set in the "Advanced" section when editing a page.
* :class:`int`: interpreted as the primary key (``pk`` field) of the desired page
* :class:`dict`: a dictionary containing keyword arguments to find the desired page
  (for instance: ``{'pk': 1}``)
* :class:`~cms.models.Page`: you can also pass a page object directly, in which case there will
  be no database lookup.

If you know the exact page you are referring to, it is a good idea to use a
``reverse_id`` (a string used to uniquely name a page) rather than a
hard-coded numeric ID in your template. For example, you might have a help
page that you want to link to or display parts of on all pages. To do this,
you would first open the help page in the admin interface and enter an ID
(such as ``help``) under the 'Advanced' tab of the form. Then you could use
that ``reverse_id`` with the appropriate templatetags::

    {% show_placeholder "right-column" "help" %}
    <a href="{% page_url "help" %}">Help page</a>

If you are referring to a page `relative` to the current page, you'll probably
have to use a numeric page ID or a page object. For instance, if you want the
content of the parent page to display on the current page, you can use::

    {% show_placeholder "content" request.current_page.parent_id %}

Or, suppose you have a placeholder called ``teaser`` on a page that, unless a
content editor has filled it with content specific to the current page, should
inherit the content of its root-level ancestor::

    {% placeholder "teaser" or %}
        {% show_placeholder "teaser" request.current_page.get_root %}
    {% endplaceholder %}


.. templatetag:: show_uncached_placeholder

show_uncached_placeholder
=========================

The same as :ttag:`show_placeholder`, but the placeholder contents will not be
cached.

Arguments:

- ``placeholder_name``
- ``page_lookup`` (see `page_lookup`_ for more information)
- ``language`` (optional)
- ``site`` (optional)

Example::

    {% show_uncached_placeholder "footer" "footer_container_page" %}

.. templatetag:: page_url


page_url
========

Displays the URL of a page in the current language.

Arguments:

- ``page_lookup`` (see `page_lookup`_ for more information)

Example::

    <a href="{% page_url "help" %}">Help page</a>
    <a href="{% page_url request.current_page.parent %}">Parent page</a>

If a matching page isn't found and :setting:`django:DEBUG` is ``True``, an
exception will be raised. However, if :setting:`django:DEBUG` is ``False``, an
exception will not be raised. Additionally, if
:setting:`django:SEND_BROKEN_LINK_EMAILS` is ``True`` and you have specified
some addresses in :setting:`django:MANAGERS`, an email will be sent to those
addresses to inform them of the broken link.

.. templatetag:: page_attribute

page_attribute
==============

This templatetag is used to display an attribute of the current page in the
current language.

Arguments:

- ``attribute_name``
- ``page_lookup`` (optional; see `page_lookup`_ for more
  information)

Possible values for ``attribute_name`` are: ``"title"``, ``"menu_title"``,
``"page_title"``, ``"slug"``, ``"meta_description"``, ``"changed_date"``, ``"changed_by"``
(note that you can also supply that argument without quotes, but this is
deprecated because the argument might also be a template variable).

Example::

    {% page_attribute "page_title" %}

If you supply the optional ``page_lookup`` argument, you will get the page
attribute from the page found by that argument.

Example::

    {% page_attribute "page_title" "my_page_reverse_id" %}
    {% page_attribute "page_title" request.current_page.parent_id %}
    {% page_attribute "slug" request.current_page.get_root %}

.. versionadded:: 2.3.2
    This template tag supports the ``as`` argument. With this you can assign the result
    of the template tag to a new variable that you can use elsewhere in the template.

    Example::

        {% page_attribute "page_title" as title %}
        <title>{{ title }}</title>

    It even can be used in combination with the ``page_lookup`` argument.

    Example::

        {% page_attribute "page_title" "my_page_reverse_id" as title %}
        <a href="/mypage/">{{ title }}</a>

.. templatetag:: render_plugin
.. versionadded:: 2.4

render_plugin
=============

This templatetag is used to render child plugins of the current plugin and should be used inside plugin templates.

Arguments:

- ``plugin``

Plugin needs to be an instance of a plugin model.

Example::

    {% load cms_tags %}
    <div class="multicolumn">
    {% for plugin in instance.child_plugin_instances %}
        <div style="width: {{ plugin.width }}00px;">
            {% render_plugin plugin %}
        </div>
    {% endfor %}
    </div>
	
Normally the children of plugins can be accessed via the ``child_plugins`` attribute of plugins.
Plugins need the ``allow_children`` attribute to set to `True` for this to be enabled.

.. templatetag:: render_model
.. versionadded:: 3.0

render_model
============

``render_model`` works by showing the content of the given attribute in
the model instance and eventually makes it clickable to edit the related model.

If the toolbar is not enabled, the value of the attribute is rendered in the
template without further action.

If the toolbar is enabled, frontend code is added to make the attribute value
clickable.

Using this templatetag you can show and edit page titles as well as fields in
standard django models, see :ref:`frontend-editable-fields` for examples and
further documentation.

Example:

.. code-block:: html+django

    <h1>{% render_model_block my_model "title" "title,abstract" %}</h1>

This will render to:

.. code-block:: html+django

    <!-- The content of the H1 is the active area that triggers the frontend editor -->
    <h1><div class="cms_plugin cms_plugin-myapp-mymodel-title-1">{{ my_model.title }}</div></h1>

**Arguments:**

* ``instance``: instance of your model in the template
* ``attribute``: the name of the attribute you want to show in the template; it
  can be a context variable name; it's possible to target field, property or
  callable for the specified model;
* ``edit_fields`` (optional): a comma separated list of fields editable in the
  popup editor;
* ``language`` (optional): the admin language tab to be linked. Useful only for
  `django-hvad`_ enabled models.
* ``filters`` (optional): a string containing chained filters to apply to the
  output content; works the same way as :ttag:`django:filter` templatetag;
* ``view_url`` (optional): the name of a url that will be reversed using the
  instance ``pk`` and the ``language`` as arguments;
* ``view_method`` (optional): a method name that will return a URL to a view;
  the method must accept ``request`` as first parameter.
* ``varname`` (optional): the templatetag output can be saved as a context
  variable for later use.

.. warning::

    ``render_model`` marks as safe the content of the rendered model
    attribute. This may be a security risk if used on fields which may hold
    non-trusted content. Be aware, and use the templatetag accordingly.


.. templatetag:: render_model_block
.. versionadded:: 3.0

render_model_block
==================

``render_model_block`` is the block-level equivalent of ``render_model``:

.. code-block:: html+django

    {% render_model_block my_model %}
        <h1>{{ instance.title }}</h1>
        <div class="body">
            {{ instance.date|date:"d F Y" }}
            {{ instance.text }}
        </div>
    {% endrender_model_block %}

This will render to:

.. code-block:: html+django

    <!-- This whole block is the active area that triggers the frontend editor -->
    <div class="cms_plugin cms_plugin-myapp-mymodel-1">
        <h1>{{ my_model.title }}</h1>
        <div class="body">
            {{ my_model.date|date:"d F Y" }}
            {{ my_model.text }}
        </div>
    </div>

In the block the ``my_model`` is aliased as ``instance`` and every attribute and
method is available; also templatetags and filters are available in the block.

**Arguments:**

* ``instance``: instance of your model in the template
* ``edit_fields`` (optional): a comma separated list of fields editable in the
  popup editor;
* ``language`` (optional): the admin language tab to be linked. Useful only for
  `django-hvad`_ enabled models.
* ``view_url`` (optional): the name of a url that will be reversed using the
  instance ``pk`` and the ``language`` as arguments;
* ``view_method`` (optional): a method name that will return a URL to a view;
  the method must accept ``request`` as first parameter.
* ``varname`` (optional): the templatetag output can be saved as a context
  variable for later use.


.. templatetag:: render_model_icon
.. versionadded:: 3.0

render_model_icon
=================

``render_model_icon`` is intended for use where the relevant object attribute
is not available for user interaction (for example, already has a link on it,
think of a title in a list of items and the titles are linked to the object
detail view); when in edit mode, it renders an **edit** icon, which will trigger
the editing changeform for the provided fields.


.. code-block:: html+django

    <h3><a href="{{ my_model.get_absolute_url }}">{{ my_model.title }}</a> {% render_model_icon my_model %}</h3>

It will render to something like:

.. code-block:: html+django

    <h3>
        <a href="{{ my_model.get_absolute_url }}">{{ my_model.title }}</a>
        <div class="cms_plugin cms_plugin-myapp-mymodel-1 cms_render_model_icon">
            <!-- The image below is the active area that triggers the frontend editor -->
            <img src="/static/cms/img/toolbar/render_model_placeholder.png">
        </div>
    </h3>

.. note::

        Icon and position can be customized via CSS by setting a background
        to the ``.cms_render_model_icon img`` selector.

**Arguments:**

* ``instance``: instance of your model in the template
* ``edit_fields`` (optional): a comma separated list of fields editable in the
  popup editor;
* ``language`` (optional): the admin language tab to be linked. Useful only for
  `django-hvad`_ enabled models.
* ``view_url`` (optional): the name of a url that will be reversed using the
  instance ``pk`` and the ``language`` as arguments;
* ``view_method`` (optional): a method name that will return a URL to a view;
  the method must accept ``request`` as first parameter.
* ``varname`` (optional): the templatetag output can be saved as a context
  variable for later use.


.. templatetag:: render_model_add
.. versionadded:: 3.0

render_model_add
================

``render_model_add`` is similar to ``render_model_icon`` but it will enable to
create instances of the given instance class; when in edit mode, it renders an
**add** icon, which will trigger the editing addform for the provided model.


.. code-block:: html+django

    <h3><a href="{{ my_model.get_absolute_url }}">{{ my_model.title }}</a> {% render_model_add my_model %}</h3>

It will render to something like:

.. code-block:: html+django

    <h3>
        <a href="{{ my_model.get_absolute_url }}">{{ my_model.title }}</a>
        <div class="cms_plugin cms_plugin-myapp-mymodel-1 cms_render_model_add">
            <!-- The image below is the active area that triggers the frontend editor -->
            <img src="/static/cms/img/toolbar/render_model_placeholder.png">
        </div>
    </h3>

.. note::

        Icon and position can be customized via CSS by setting a background
        to the ``.cms_render_model_add img`` selector.

..warning::

    You **must** pass an instance of your model as instance parameter.

**Arguments:**

* ``instance``: instance of your model in the template
* ``edit_fields`` (optional): a comma separated list of fields editable in the
  popup editor;
* ``language`` (optional): the admin language tab to be linked. Useful only for
  `django-hvad`_ enabled models.
* ``view_url`` (optional): the name of a url that will be reversed using the
  instance ``pk`` and the ``language`` as arguments;
* ``view_method`` (optional): a method name that will return a URL to a view;
  the method must accept ``request`` as first parameter.
* ``varname`` (optional): the templatetag output can be saved as a context
  variable for later use.



.. _django-hvad: https://github.com/kristianoellegaard/django-hvad


*****************
Menu Templatetags
*****************

.. highlightlang:: html+django

To use any of the following templatetags you first need to load them at the
top of your template::

    {% load menu_tags %}
	
.. templatetag:: show_menu

show_menu
=========

The ``show_menu`` tag renders the navigation of the current page. You can
overwrite the appearance and the HTML if you add a ``cms/menu.html`` template
to your project or edit the one provided with django CMS. ``show_menu`` takes
four optional parameters: ``start_level``, ``end_level``, ``extra_inactive``,
and ``extra_active``.

The first two parameters, ``start_level`` (default=0) and ``end_level``
(default=100) specify from which level the navigation shoud be rendered
and at which level it should stop. If you have home as a root node and don't
want to display home you can render the navigation only after level 1.

The third parameter, ``extra_inactive`` (default=0), specifies how many levels
of navigation should be displayed if a node is not a direct ancestor or
descendant of the current active node.

Finally, the fourth parameter, ``extra_active`` (default=100), specifies how
many levels of descendants of the currently active node should be displayed.

show_menu Examples
------------------

Complete navigation (as a nested list)::

    <ul>
        {% show_menu 0 100 100 100 %}
    </ul>

Navigation with active tree (as a nested list)::

    <ul>
        {% show_menu 0 100 0 100 %}
    </ul>

Navigation with only one active extra level::

    <ul>
        {% show_menu 0 100 0 1 %}
    </ul>

Level 1 navigation (as a nested list)::

    <ul>
        {% show_menu 1 %}
    </ul>

Navigation with a custom template::

    {% show_menu 0 100 100 100 "myapp/menu.html" %}


.. templatetag:: show_menu_below_id

show_menu_below_id
==================

If you have set an id in the advanced settings of a page, you can display the
submenu of this page with a template tag. For example, we have a page called
meta that is not displayed in the navigation and that has the id "meta"::

    <ul>
        {% show_menu_below_id "meta" %}
    </ul>

You can give it the same optional parameters as ``show_menu``::

    <ul>
        {% show_menu_below_id "meta" 0 100 100 100 "myapp/menu.html" %}
    </ul>

.. templatetag:: show_sub_menu

show_sub_menu
=============

Displays the sub menu of the current page (as a nested list).

The first argument, ``levels`` (default=100), specifies how many levels deep the submenu should be
displayed

The second argument, ``root_level`` (default=None), specifies at what level, if any, the menu should root at.
For example, if root_level is 0 the menu will start at that level regardless of what level the current page is on.

The third argument, ``nephews`` (default=100), specifies how many levels of nephews (children of siblings) are show.

The template can be found at ``cms/sub_menu.html``::

    <ul>
        {% show_sub_menu 1 %}
    </ul>

Rooted at level 0::

    <ul>
        {% show_sub_menu 1 0 %}
    </ul>

Or with a custom template::

    <ul>
        {% show_sub_menu 1 "myapp/submenu.html" %}
    </ul>

.. templatetag:: show_breadcrumb

show_breadcrumb
===============

Renders the breadcrumb navigation of the current page.
The template for the HTML can be found at ``cms/breadcrumb.html``::

    {% show_breadcrumb %}

Or with a custom template and only display level 2 or higher::

    {% show_breadcrumb 2 "myapp/breadcrumb.html" %}
    
Usually, only pages visible in the navigation are shown in the
breadcrumb. To include *all* pages in the breadcrumb, write::

    {% show_breadcrumb 0 "cms/breadcrumb.html" 0 %}

If the current URL is not handled by the CMS or by a navigation extender,
the current menu node can not be determined.
In this case you may need to provide your own breadcrumb via the template.
This is mostly needed for pages like login, logout and third-party apps.
This can easily be accomplished by a block you overwrite in your templates.

For example in your base.html::

    <ul>
        {% block breadcrumb %}
        {% show_breadcrumb %}
        {% endblock %}
    <ul>

And then in your app template::

    {% block breadcrumb %}
    <li><a href="/">home</a></li>
    <li>My current page</li>
    {% endblock %}

.. templatetag:: page_language_url


page_language_url
=================

Returns the url of the current page in an other language::

    {% page_language_url de %}
    {% page_language_url fr %}
    {% page_language_url en %}

If the current url has no cms-page and is handled by a navigation extender and
the url changes based on the language, you will need to set a language_changer
function with the set_language_changer function in cms.utils.

For more information, see :doc:`i18n`.

.. templatetag:: language_chooser


language_chooser
================

The ``language_chooser`` template tag will display a language chooser for the
current page. You can modify the template in ``menu/language_chooser.html`` or
provide your own template if necessary.

Example::

    {% language_chooser %}

or with custom template::

    {% language_chooser "myapp/language_chooser.html" %}
    
The language_chooser has three different modes in which it will display the
languages you can choose from: "raw" (default), "native", "current" and "short".
It can be passed as the last argument to the ``language_chooser tag`` as a string.
In "raw" mode, the language will be displayed like its verbose name in the
settings. In "native" mode the languages are displayed in their actual language
(eg. German will be displayed "Deutsch", Japanese as "日本語" etc). In "current"
mode the languages are translated into the current language the user is seeing
the site in (eg. if the site is displayed in German, Japanese will be displayed
as "Japanisch"). "Short" mode takes the language code (eg. "en") to display.

If the current url has no cms-page and is handled by a navigation extender and
the url changes based on the language, you will need to set a language_changer
function with the set_language_changer function in menus.utils.

For more information, see :doc:`i18n`.

********************
Toolbar Templatetags
********************

.. highlightlang:: html+django

The ``cms_toolbar`` templatetag is included in the ``cms_tags`` library and will add the 
required css and javascript to the sekizai blocks in the base template. The templatetag 
has to be placed after the ``<body>`` tag and before any ``{% cms_placeholder %}`` occurrences 
within your HTML.

Example::

    <body>
    {% cms_toolbar %}
    {% placeholder "home" %}
    ...


.. note::

    Be aware that you can not surround the cms_toolbar tag with block tags.
    The toolbar tag will render everything below it to collect all plugins and placeholders, before
    it renders itself. Block tags interfere with this.

