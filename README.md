Django-Tools
============

This is a set of tools for working with Django repositories. These are not
Django applications, but rather things that can help interacting with a
set of Django code.

The utilities are:

update_django_urls
------------------
Beginning with Django 1.3, the {% url %} tag was optionally changed. Previously,
{% url foo %} would look for a url named 'foo'. Once we get to Django 1.5,
{% url foo %} will look for a variable named 'foo' in the context. To get the
previous behavior, you need to do {% url 'foo' %}.

In between Django 1.3 and 1.5, you can get the 1.5 behavior by adding
{% load url from future %}. This utility will convert a directory (or
set of directories) from the old behavior to the new behavior, including
adding the {% load url from future %}.

If you have templates that have been converted already, the utility will
recognize that and not do anything to those templates.

_Usage:_ update_django_urls [template_dir1...]

If no template directory is specified, it will work in the current 
directory. All subdirectories are processed.