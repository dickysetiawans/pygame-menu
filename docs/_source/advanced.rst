
====================
Package organization
====================

The :py:mod:`pygameMenu.widgets` package contains the widget support for the Menu.
Its structure consists of several sub-packages::

    pygameMenu/
        widgets/
            core/           Main object classes for Widget and Selector
            examples/       Some examples of widgets
            selection/      Selection effect applied to widgets
            widget/         Menu widget objects


===============
Menu navigation
===============

The :py:class:`pygameMenu.Menu` keep the link and history with all its sub-menus.
To be functional, the menu links (pointer to previous and next menus in nested submenus),
have to respect some rules:

- Public methods access to menu instance through ``_current`` attribute, because
  user can move through sub-menus and they should target the current Menu instance.
- Private methods access to menu instance through ``self`` (not ``_current``) because
  these methods are called by public (``_current``) or by themselves.
- Methods used to navigate through menus (:py:meth:`pygameMenu.Menu._open`,
  :py:meth:`pygameMenu.Menu._reset`, ...) should be the only place where the ``_top``
  attribute is used.


===============
Create a widget
===============

All widget classes shall inherit from :py:class:`pygameMenu.widgets.core.widget.Widget`,
and they must be located in the :py:mod:`pgameMenu.widgets.widget` package. The most
basic widget should contain this code:

.. code-block:: python

    from pygameMenu.widgets.core.widget import Widget

    class MyWidget(Widget):

        def __init__(self, params):
            super(MyWidget, self).__init__(params)
            ...

        def _apply_font(self):
            """
            Function triggered after a font is applied to the widget
            by Menu._configure_widget() method.
            """
            ...

        def draw(self, surface):
            """
            Draw the widget shape.
            """
            # Required, render first, then draw
            self._render()

            # Draw the background of the Widget (optional)
            self._fill_background_color(surface)

            # Draw the self._surface pygame object on the given surface
            surface.blit(self._surface, self._rect.topleft)

        def _render(self):
            """
            Render the widget surface.
            This method shall update the attribute _surface with a pygame.Surface
            representing the outer borders of the widget.
            """
            # Generate widget surface
            self._surface = pygame.surface....
            # Update the width and height of the Widget
            self._rect.width, self._rect.height = self._surface.get_size() + SomeConstants

        def update(self, events):
            """
            Update internal variable according to the given events list
            and fire the callbacks.
            """
            ...
            return False

.. warning:: After creating the widget, it must be added to  ``__init__.py`` file of the
             :py:mod:`pgameMenu.widgets` package.

             .. code-block:: python

                 from pygameMenu.widgets.widget.mywidget import MyWidget

For adding the widget to the :py:class:`pygameMenu.Menu` class, a public method
:py:meth:`pygameMenu.Menu.add_mywidget` with the following structure have to be
added:

.. code-block:: python

    import pygameMenu.widgets as _widgets

    class Menu(object):
        ...

        def add_mymenu(self, params, **kwargs):
            """
            Add MyWidget to the menu.
            """
            attributes = self._current._filter_widget_attributes(kwargs)

            # Create your widget
            widget = _widgets.MyWidget(..., **kwargs)

            self._current._configure_widget(widget=widget, **attributes)
            self._current._append_widget(widget)
            return widget

        ...

.. note:: This method uses **kwargs** parameter for defining the settings of the
          Widget as the background, margin, etc. This is applied automatically
          by the Menu in :py:meth:`pygameMenu.Menu._configure_widget`
          method. If **MyWidget** needs additional parameters please use some that
          are not named as the default kwargs used by the Menu Widget system.


=========================
Create a selection effect
=========================

The widgets in Menu are drawn with the following idea:

#. Each time a new Widget is added regenerate the position of them.
#. Widgets can be active or not. The active widget will catch user events as keyboard or mouse.
#. Active widgets have a decoration, named *Selection*
#. The drawing process is:

 #. Draw Menu background color/image
 #. Draw all widgets
 #. Draw *Selection* decoration on selected widget surface area
 #. Draw the menubar
 #. Draw the scrollbar

For defining a new selection effect, a new :py:class:`pygameMenu.widgets.core.selection.Selection`
sub-class must be added to :py:mod:`pgameMenu.widgets.selection` package. A basic class must
contain the following code:

.. code-block:: python

    from pygameMenu.widgets.core.selection import Selection

    class MySelection(Selection):

        def __init__(self):
            super(MySelection, self).__init__(params)

        def get_margin(self):
            """
            As selection decorations can be described with a box, this method must return
            the additional margin of the selection. If the margin is zero, then the selection
            size is the same as the original widget.

            The method must return the width of the bottom, left, top and right margins.

             --------------------------
            |          ^ top           | In this example, XXXX represents the
            | left  XXXXXXXXXXXX right | Widget to be Selected.
            |<----> XXXXXXXXXXXX<----->|
            |         v bottom         |
             --------------------------

             All distances must be in pixels (px).
            """
            return top, left, bottom, right

        def draw(self, surface, widget):
            """
            This method receives the surface to draw the selection and the
            widget itself. For retrieving the Selection coordinates the rect
            object from widget should be used.
            """
            surface.draw(.....)

.. warning:: After creating the selection effect, it must be added to  ``__init__.py`` file of the
             :py:mod:`pgameMenu.widgets` package.

             .. code-block:: python

                 from pygameMenu.widgets.selection.myselection import MySelection

Finally, this new selection effect can be set following one of these two ways:

1. Pass it when adding a new widget to the menu

    .. code-block:: python

        import pygameMenu

        menu = pygameMenu.Menu(...)

        menu.add_button(..., selection_effect=pygameMenu.widgets.MySelection(...))

2. To apply it on alls menus and widgets (and avoid passing it for each added widget),
   a theme can be created

    .. code-block:: python

        import pygameMenu

        MY_THEME = pygameMenu.Theme(
            ...,
            widget_selection_effect=pygameMenu.widgets.MySelection(...)
        )

        menu = pygameMenu.Menu(..., theme=MY_THEME)