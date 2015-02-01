# django-sekizai-processors

Processors to compile files from markup languages such as SASS/SCSS, using the ``preprocessor``
option on Sekizai's templatetag ``addtoblock``. In the future other markup languages and the
``postprocessor`` option for Sekizai's templatetag  ``render_block`` will also be supported.

Additionally, **django-sekizai-processors** is shipped with a management command, which can convert
the content of all occurrences inside the templatetag ``addtoblock`` as an offline operation. Hence
the **libsass** compiler is not required in a production environment.

During development, a [sourcemap](https://developer.chrome.com/devtools/docs/css-preprocessors) is
generated along side with the compiled ``*.css`` file. This allows to debug style sheet errors much
easier.

By using this preprocessor, you can safely remove your Ruby projects “Compass” and “SASS”.

## Installation

```
pip install libsass django-sekizai django-sekizai-processors django-compressor
```

If using offline compilation, ``libsass`` is not required on the production environment.

In ``settings.py``:

```python
INSTALLED_APPS = (
    ...
    'sekizai',
    'sekizai_processors',
    ...
)
```

Optionally, add a list of additional search pathes, the SASS compiler may examine when using the
``@import "...";`` tag in SASS files:

```python
import os

SEKIZAI_PROCESSOR_INCLUDE_DIRS = (
    os.path.join(PROJECT_PATH, 'styles/scss'),
    os.path.join(PROJECT_PATH, 'node_modules/compass-mixins/lib'),
    os.path.join(PROJECT_PATH, 'node_modules/bootstrap-sass/assets/stylesheets'),
)
```

Annotation: For this example, you are encouraged to install the dependencies for Compass and/or
Bootstrap-SASS, using ``npm install compass-mixins bootstrap-sass``.


## Preprocessing SASS

**django-sekizai-processors** is shipped with a built-in preprocessor to convert
``*.scss`` or ``*.sass`` files into ``*.css`` while rendering the template. For performance reasons
this is done only once, but the preprocessor keeps track on the timestamps and recompiles if any
of the SASS file is younger than the generated CSS file.

### In your Django templates

```html
{% load sekizai %}

{% addtoblock "css" preprocessor "sekizai_processors.sass_preprocessor.compilescss" %}<link href="{% static 'myapp/css/mystyle.scss' %}" rel="stylesheet" type="text/css" />{% endaddtoblock %}
```

## Offline compilation

If you want to precompile all occurences of your SASS files, on the command line invoke:

```
./manage.py compilescss
```

This is useful for production environments, where SASS files can't be compiled on the fly. To
simplify the deployment, the compiled ``*.css`` files are stored side-by-side with their
corresponding ``*.scss`` files; just run ``./manage.py collectstatic`` the usual way. In case you
don't want to expose the ``*.scss`` files in a production environment, deploy with
``./manage.py collectstatic --ignore=.scss``.
