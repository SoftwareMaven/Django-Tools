#!/usr/bin/env python
"""
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
"""
import sys, os, os.path, re, shutil

template_ext = 'html'

if len(sys.argv) > 1:
    paths = sys.argv[1:]
else:
    paths = [ os.getcwd() ]

LOAD_URL = "{% load url from future %}"
URL_EXP = r'\{%\s+url\s+([A-Za-z0-9_\-]+)\s+([^%]*?)%\}'
EXTEND_EXP = r'\{%\s+extends\s+.*?\s+%\}'
LOAD_EXP = r"\{%\s+load\s+url\s+from\s+future\s+%\}"
TAG_STR = """{{% url {q}{name}{q} {params}%}}"""

url_re = re.compile(URL_EXP)
extend_re = re.compile(EXTEND_EXP)
load_re = re.compile(LOAD_EXP)

def print_status(path, status):
    sys.stderr.write('{0} - {1}\n'.format(path, status))

def print_err(path, line, err):
    sys.stderr.write('{0}:{1} - {2}\n'.format(path, line, err))

def fix_file(path):
    ''' Fixes the {% url %} tags in django templates to use the
    newer format of {% url 'name' %} (including putting a
    {% load url from future %} in if necessary)'''

    with open(path, 'r') as in_file:
        file_lines = in_file.readlines()

    # Do a pre-run to see if we even care about this template and,
    # if we do, that we put the load tag in the right place
    has_extends = False
    has_url_tag = False
    has_load_tag = False
    extends_line = 1
    for line in file_lines:
        if not has_extends:
            if extend_re.search(line):
                has_extends = True
            else:
                extends_line += 1
        if not has_url_tag and url_re.search(line):
            has_url_tag = True
        if not has_load_tag and load_re.search(line):
            has_load_tag = True
        if has_url_tag and has_extends:
            break

    # Don't do anything if the template is already upgraded or  isn't a url tag
    if has_load_tag or not has_url_tag:
        reason = 'already converted' if has_load_tag else 'no urls'
        print_status(path, "Skipping file: {0}".format(reason))
        return

    try:
        # Make a backup copy of the file before we munge things
        print_status(path, "Url-izing file")

        backup_path = '{0}.bak'.format(path)
        if has_url_tag:
            shutil.copy(path, backup_path)

        with open(path, 'w') as out_file:
            line_no = 1
            for line in file_lines:
                if has_extends:
                    if line_no == (extends_line + 1):
                        out_file.write(LOAD_URL)
                        out_file.write('\n')
                else:
                    if line_no == 1:
                        out_file.write(LOAD_URL)
                        out_file.write('\n')

                last_spot = 0
                combined = []
                for m in url_re.finditer(line):
                    quote = "'"
                    if "'" in m.group(1):
                        quote = '"'
                    if quote == '"' and '"' in  m.group(1):
                        print_err(path, line_no, 'URL contains both quote types')
                    combined.append(line[last_spot:m.start()])
                    params = '' if len(m.groups()) < 2 else m.group(2)
                    combined.append(TAG_STR.format(q=quote, name=m.group(1),
                                                   params=params))
                    last_spot = m.end()
                combined.append(line[last_spot:])
                out_file.write(''.join(combined))

                line_no += 1
        os.unlink(backup_path)
    except:
        shutil.copy(backup_path, path)
        raise

def convert_tree(path):
    ext = '.{0}'.format(template_ext)
    for root, dir, files in os.walk(path):
        for fname in files:
            if not fname.endswith(ext):
                continue
            fpath = os.path.join(root, fname)
            fix_file(fpath)

for path in paths:
    convert_tree(path)
