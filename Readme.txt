###############################################
#				Basic Usage                   #
###############################################
This application uses 2 main configuration files:
 
*config.yaml:
	It stores the language of the app (and of the course) and the maximum number of
	students

*course.yaml:
	It stores some configuration about the course (like the course title, etc). On each
	'locale/[ANY_LANGUAGE]' folder there is a version of this file on that language. So, 
	the app will look at first if there is any course.yaml on the folder of the language
	specified on the config.yaml file. If there is, it will load that file, if not it will
	load by default the 'course.yaml' on the root folder.

You will also have the change some things on the app.yaml file, according to your Google App Engine settings.


###############################################
#        Uploading Unit and Lesson data       #
###############################################
*For uploading the units
	appcfg.py upload_data --url=http://localhost:PORT/_ah/remote_api --config_file=bulkloader.yaml --filename=data/unit.csv --kind=Unit
*For uploading the lessons
	appcfg.py upload_data --url=http://localhost:PORT/_ah/remote_api --config_file=bulkloader.yaml --filename=data/lesson.csv --kind=Lesson


###############################################
#              Internationalization           #
###############################################
1. In order to be able to upload changes to the language definitions some tools need to be installed.
First in Python directory intall Babel: see http://babel.edgewall.org/
The easiest way is to first install setuptools from: https://pypi.python.org/pypi/setuptools 
Make sure that Python folders are in your PATH variable. I use Python 2.7, so to make Babel work I’ll need the following values in PATH: “C:\Python27;C:\Python27\Scripts”. 
Run easy_install babel from the command prompt.
The Scripts folder contains the pybabel executable. You’ll need its comman ‘pybabel’ to generate locale-specific files.

2. Install jinja2: http://jinja.pocoo.org
Once again, you need to install it, as Babel will need it to parse strings in templates. Just run easy_install Jinja2


3. Put Babel and gaepytz libraries inside your GAE application. They are required for i18n module.

4. Configure jinja2 to be used in your application. You’ll need the following entry in app.yaml:

libraries:
- name: jinja2
  version: "2.6"
and your webhandler.py will look something similar to this:

import webapp2
from webapp2_extras import jinja2
from webapp2_extras import i18n
from google.appengine.ext.webapp.util import run_wsgi_app
 
class MainHandler(webapp2.RequestHandler):
    @webapp2.cached_property
    def jinja2(self):
        return jinja2.get_jinja2(app=self.app)
 
    def get(self):
        i18n.get_i18n().set_locale('ru_RU') # sample locale assigned
        ... # your web site functionality goes here
 
# jinja2 config with i18n enabled
config = {'webapp2_extras.jinja2': {
             'template_path': 'templates',
             'environment_args': { 'extensions': ['jinja2.ext.i18n'] }
           }
          }
application = webapp2.WSGIApplication([('.*', MainHandler)], config=config)
 
def main():
    run_wsgi_app(application)
 
if __name__ == "__main__":
    main()
This code will work if you put your jinja2 templates into “templates” folder.

4. Create the translations markup. This means, you define the translatable strings in python code with a commonly used ‘_’ alias:

from webapp2_extras.i18n import gettext as _
 
def do_some_text():
    return _('some text')
or in jijna2 template with {% trans %} block:

{% block buttons %}
<div> 
    <div onclick="window.print()">{% trans %}Print{% endtrans %}</div>
</div>
{% endblock %}
5. Create a Babel configuration file babel.cfg (put it into the application folder for now):

[jinja2: **/templates/**.html]
encoding = utf-8
[python: source/*.py]
[extractors] 
jinja2 = jinja2.ext:babel_extract
This file instructs Babel to extract translatable strings from html jinja2 templates in “templates” folder and python files in “source” folder.

6. Now it’s time to create translations. First, add a “locale” folder in application root. Still being in root folder, run the following pybabel command to extract the translatable strings from the code

pybabel extract -F ./babel.cfg -o ./locale/messages.pot ./
then initialize the locales with

pybabel init -l en_US -d ./locale -i ./locale/messages.pot
pybabel init -l ru_RU -d ./locale -i ./locale/messages.pot
Now open locale\ru_RU\LC_MESSAGES\messages.po file in your favorite text editor, and produce the translations (you have to change ‘msgstr’ only):

#: templates/sample.html:10
msgid "Print"
msgstr "Печать"
#: source/test.py:13
msgid "some text"
msgstr "немного текста"
And finally compile the texts with

pybabel compile -f -d ./locale
7. Every time you need to add more strings, you should do the same steps as in 6, but use “update” instead of “init”:

pybabel update -l en_US -d ./locale -i ./locale/messages.pot
pybabel update -l ru_RU -d ./locale -i ./locale/messages.pot
Done! You should be able to run the application and see the strings translated.

Create babel.cfg in the main directory for your coursebuilder files 
The txt in babel.cfg should read:

[jinja2: **/views/**.html]
encoding = utf-8
[python: controllers/*.py]
[extractors]
jinja2 = jinja2.ext:babel_extract

The current language catalogs are on './modules/i18n/resources/locale'

For maintaining the internationalization:

*Update the current languages (when adding more strings):
	pybabel extract -F ./babel.cfg -o ./modules/i18n/resources/locale/'language[no_NO]/messages.pot ./
	pybabel update -l 'language[no_NO] -d ./modules/i18n/resources/locale -i ./modules/i18n/resources/locale/messages.pot

*Add more languages
	pybabel init -l language[es_ES] -d ./locale -i ./locale/messages.pot

*Compile the languages
	pybabel compile -f -d ./modules/i18n/resources/locale

	se: http://mikeshilkov.wordpress.com/2012/07/26/enable-jinja2-and-i18n-translations-on-google-appengine/