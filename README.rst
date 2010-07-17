Coffin: Jinja2 adapter for Django
---------------------------------


Supported Django template functionality
=======================================

Coffin currently makes the following Django tags available in Jinja:

- {% cache %} - has currently an incompatibility: The second argument
  (the fragment name) needs to be specified with surrounding quotes
  if it is supposed to be a literal string, according to Jinja2 syntax.
  It will otherwise be considered an identifer and resolved as a
  variable.

- {% load %} - is actually a no-op in Coffin, since templatetag
  libraries are always loaded. See also "Custom Filters and extensions".

- {% spaceless %}

- {% url %} - additionally, a ``"view"|url()`` filter is also
  available.

- {% with %}

- {% csrf_token %}

Django filters that are ported in Coffin:

- date
- floatformat
- pluralize (expects an optional second parameter rather than the
  comma syntax)
- time
- timesince
- timeuntil
- truncatewords
- truncatewords_html

The template-related functionality of the following contrib modules has
been ported in Coffin:

- ``coffin.contrib.markup``
- ``coffin.contrib.syndication``.

Jinja 2's ``i18n`` extension is hooked up with Django, and a custom version
of makemessages supports string extraction from both Jinja2 and Django
templates.

Rendering
=========

Change the TEMPLATE_LOADERS settings to contain only the following loader::

   TEMPLATE_LOADERS = (
      'coffin.template.loaders.Loader',
   )

And move all previously defined template loaders to the
JINJA2_TEMPLATE_LOADERS setting directive::

   JINJA2_TEMPLATE_LOADERS = (
       'django.template.loaders.app_directories.Loader',
       'django.template.loaders.filesystem.Loader',
   )

From now on, all of your views, generic views and error pages will be handled
and rendered by Jinja2.

Using the django rendering engine
=================================

If your project uses some applications which needs to original django
templating engine to correctly render their templates, you can add their names
to a JINJA2_DISABLED_APPS settings and coffin will render the templates using
the django templating engine.

If you use the built-in admin app, you have then to add the following setting::

   JINJA2_DISABLED_APPS = (
       'admin',
   )
   
Please note coffin uses the folder root folder of the template to decide to
which application it belongs (the django.contrib.admin application stores all
its templates in the 'admin' subdirectory).

Custom filters and extensions
=============================

Coffin uses the same templatetag library approach as Django, meaning
your app has a ``templatetags`` directory, and each of it's modules
represents a "template library", providing new filters and tags.

A custom ``Library`` class in ``coffin.template.Library`` can be used
to register Jinja-specific components.

Coffin can automatically make your existing Django filters usable in
Jinja, but not your custom tags - you need to rewrite those as Jinja
extensions manually.

Example for a Jinja-enabled template library::

    from coffin import template
    register = template.Library()

    register.filter('plenk', plenk)   # Filter for both Django and Jinja
    register.tag('foo', do_foo)       # Django version of the tag
    register.tag(FooExtension)        # Jinja version of the tag
    register.object(my_function_name) # A global function/object
    register.test(my_test_name)       # A test function

You may also define additional extensions, filters, tests, and globas via your ``settings.py``::

    JINJA2_FILTERS = (
        'path.to.myfilter',
    )
    JINJA2_TESTS = {
        'test_name': 'path.to.mytest',
    }
    JINJA2_EXTENSIONS = (
        'jinja2.ext.do',
    )

Other things of note
====================

When porting Django functionality, Coffin currently tries to avoid
Django's silent-errors approach, instead opting to be explicit. Django was
discussing the same thing before it's 1.0 release (*), but was constrained
by backwards-compatibility  concerns. However, if you are converting your
templates anyway, it might be a good opportunity for this change.

(*) http://groups.google.com/group/django-developers/browse_thread/thread/f323338045ac2e5e

Jinja2's ``TemplateSyntaxError`` (and potentially other exception types)
are not compatible with Django's own template exceptions with respect to
the TEMPLATE_DEBUG facility. If TEMPLATE_DEBUG is enabled and Jinja2 raises
an exception, Django's error 500 page will sometimes not be able to handle
it and crash. The solution is to disable the TEMPLATE_DEBUG setting in
Django. See http://code.djangoproject.com/ticket/10216 for further
information.

``coffin.template.loader`` is a port of ``django.template.loader`` and
comes with a Jinja2-enabled version of ``get_template()``.

``coffin.template.Template`` is a Jinja2-Template that supports the
Django render interface (being passed an instance of Context), and uses
Coffin's global Jinja2 environment.

``coffin.interop`` exposes functionality to manually convert Django
filters to Jinja2 and vice-versa. This is also what Coffin's ``Library``
object uses.

A Jinja2-enabled version of ``add_to_builtins`` can be found in the
``django.template`` namespace.

You may specify additional arguments to send to the ``Environment`` via ``JINJA2_ENVIRONMENT_OPTIONS``::

    from jinja2 import StrictUndefined
    JINJA2_ENVIRONMENT_OPTIONS = {
        'autoescape': False,
        'undefined': StrictUndefined,
    }

Things not supported by Coffin
==============================

These is an incomplete list things that Coffin does not yet and possibly
never will, requiring manual changes on your part:

- The ``slice`` filter works differently in Jinja2 and Django.
  Replace it with Jinja's slice syntax: ``x[0:1]``.

- Jinja2's ``default`` filter by itself only tests the variable for
  **existance**. To match Django's behaviour, you need to pass ``True``
  as the second argument, so that it will also provide the default
  value for things that are defined but evalute to ``False``

- Jinja2's loop variable is called ``loop``, but Django's ``forloop``.

- Implementing an equivalent to Django's cycle-tag might be difficult,
  see also Django tickets #5908 and #7501. Jinja's own facilities
  are the ``forloop.cycle()`` function and the global function
  ``cycler``.

- The ``add`` filter might not be worth being implemented. ``{{ x+y }}``
  is a pretty basic feature of Jinja2, and could almost be lumped
  together with the other Django->Jinja2 syntax changes.

- Django-type safe strings passed through the context are not converted
  and therefore not recognized by Jinja2. For example, a notable place
  were this would occur is the HTML generation of Django Forms.

- The {% autoescape %} tag is immensily difficult to port and currently
  not supported.

- Literal strings from within a template are not automatically
  considered  "safe" by Jinja2, different from Django. According to
  Armin Ronacher, this is a design limitation that will not be changed,
  due to many Python builtin functions and methods, whichyou are free
  to use in Jinja2, expecting raw, untainted strings and thus not being
  able to work with Jinja2's ``Markup`` string.


Running the tests
====================

Use the nose framework:

    http://somethingaboutorange.com/mrl/projects/nose/
