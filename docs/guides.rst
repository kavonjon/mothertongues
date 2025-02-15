.. _guides:

Guides
======

Here are some guides to help do some of the basic tasks required for creating a Mother Tongues Dictionary.

Schema validation
-----------------

Below are some resources to help you create the two documents you need to create your dictionary.

You will need to create one Language Configuration file for your dictionary
and one Resource Configuraiton file for *each* source of data.

1. MTD Language Configuration file
    Every language must have a configuration file, and it must be validated against the following schema `here <https://roedoejet.github.io/mothertongues/mtd/languages/config_schema.json>`_
    Below is a minimal Language Configuration file labelled danish_config.json. Note that it must point to each piece of data and where that data is located.

.. code-block:: json

    {
        "config":{
            "L1": "Danish",
            "L2": "English"
        },
        "data": [
            {
                "manifest": "/Users/pinea/danish_csv_manifest.json",
                "resource": "/Users/pinea/danish.csv"
            }
        ]
    }


2. MTD Data Resource Configuration file
    Every data resource must have a configuration, and it must be validating against the following schema `here <https://roedoejet.github.io/mothertongues/mtd/languages/manifest_schema.json>`_
    Below is a minimal Data Configuration file found at, for example, `/Users/pinea/danish_csv_manifest.json` on my machine. This must be referenced in the language configuration above.
    This configuration file describes a CSV dictionary resource that only has two columns where the first column includes the word in the target language and the second column includes the 'definition' of that word or 'gloss' in the L2 language.

.. code-block:: json

    {
        "name": "2017 Spreadsheet",
        "sorting": "word",
        "targets": {
            "word": "0",
            "definition": "1"
        }
    }

In the future, I would like to have an online schema validation tool in the vein of `SwaggerHub <https://swagger.io/tools/swaggerhub/faster-api-design/>`_

Basic
-----

There is a basic Google Colaboratory tutorial for how to make a Mother Tongues Dictionary using a Google sheet available `here <https://colab.research.google.com/drive/1Z2Isg5qAFdINpxnTnKPpa1GDOpTfSSFg>`_. The tutorial is from a workshop taught at `ICLDC 6 <https://icldc6.weebly.com/>`_ and also shows how to deploy using GitHub Pages. The following mini-guides explain how to edit your configuration files to be able to get more functionality than in the colab tutorial.

Customized Alphabet
~~~~~~~~~~~~~~~~~~~

Adding your custom alphabet allows your entries to be sorted based on that alphabet. In your Language Configuration file, set the `alphabet` key equal to an array containing the letters in your language's alphabet in alphabetical order:

.. code-block:: json

    { 
        "config": {
            "L1": ...,
            "L2": ...,   
            "alphabet": ["a", "b", "c"]
        }
    }

Optional Information
~~~~~~~~~~~~~~~~~~~~

To add information that can be optionally displayed in the UI, you must point to it in your Data Resource configuration file. For example, if you wanted to add "Part of Speech" information that could be displayed optionally and that was present in column "F" of an Excel spreadsheet and etymological information that was present in columng "G", you would add the following to your Data Resource configuration file:

.. code-block:: json

    {
        "targets": {
            "optional": [
                    { 
                        "Part of Speech": "F"
                    }
                ]
        }
    }


Images & Audio
~~~~~~~~~~~~~~

To add images and audio, you must have the filenames of your files in your dictionary data resource. Then, change your Data Resource configuration files to point to the location of the filenames.

For images, just add a target for the `img` key. Take the following example for an Excel spreadsheet with image filenames in column "D":

.. code-block:: json

    {
        "targets": {
            "img": "D"
        }
    }

For audio, you minimally have to add the filename, but you can also add a speaker name. You can also choose between `audio` for audio files in the target language, `definition_audio` for audio files of the definition, `example_sentence_audio` for audio files corresponding to an example sentence and `example_sentence_definition_audio` for audio files corresponding to the definitions of example sentences.

Take the following example for an Excel spreadsheet with audio in columns "B" & "C" and example sentence audio in column "D". The speaker names for audio files are in columns "E", "F", and "G" respectively.

.. code-block:: json

    {
        "targets": {
            "audio": [
                { 
                    "filename": "B",
                    "speaker": "E" 
                },
                { 
                    "filename": "C",
                    "speaker": "F" 
                } 
            ],
            "example_sentence_audio": [
                [
                    {
                        "filename": "D",
                        "speaker": "G"
                    }
                ]
            ]
        }

    }
 

Semantic Categories
~~~~~~~~~~~~~~~~~~~

To add semantic categories to your entries, you can make use of both the `theme` and `secondary_theme` keys in the Data Resource configuration file. Using these will allow your entries to be sorted based on semantic categories like "colours", or "animals" etc.

For example, suppose you have an Excel spreadsheet where column "A" has main categories like "Animals", and column "B" has sub-categories like "- Fish", and "- Reptiles". Your Data Resource congfiguration file would have to add the following targets:

.. code-block:: json

    {
        "targets": {
            "theme": "A",
            "secondary_theme": "B"
        }
    }



Approximate Search
~~~~~~~~~~~~~~~~~~

This is a guide for how to customize the approximate search algorithm used by MTD. See `this paper <http://roedoejet.github.io/cv/static/cv/pdfs/computel.pdf>`_ for further discussion.

Approximate search isn't just a *nice* feature for dictionaries of endangered languages - it's usually a requirement. 
Often, it's learners of languages that want to use dictionaries the most, and if your dictionary doesn't allow approximate search, beginners might have a hard time accessing entries in the dictionary.


Building a Dictionary
~~~~~~~~~~~~~~~~~~~~~

Once you have prepared a Language Configuration and Resource Configuraiton files for each source of data, you need to build the dictionary files, which contain the data for your dictionary in machine readable form.

First, run the ``mtd prepare`` command. You will want to point this command at the directory with your configuration files, either by declaring the directory path as an argument, as in

``mtd prepare <directory path>``

or you can run the following command from within the directory:

``mtd prepare .``

If successful, you will see the message::

  Successfully built static files for the following dictionaries: <dictionary>. You may now run the app.

Next, build the dictionary using the ``mtd export`` command to create .js files. For example, given a dictionary named 'abc', a Language configuration file named 'abc_config.json', and a desired output folder 'output', run the following:

``mtd export abc_config.json js output``

You will see various messages displayed, potentially including info, warnings, and errors, which are ordered in terms of severity. If there are only info messages, the command executed successfully, and the info might advise you on how to improve your configuration inputs. If there are warnings (which may be in addition to info messages), the command executed but there might be serious issues with the output files. If there are errors (which may be in addition to info messages and warnings), the command did not execute successfully.

Checking your output folder, you should see two files: config-abc.js and dict_cached-abc.js. These files contain the data for your dictionary.


Adding Dictionary Data to a User Interface
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Once you have built your dictionary files, you can add them to a dictionary UI such as an `MTD UI <https://github.com/roedoejet/mothertongues-ui>`_. As a simple example, you can make a functioning dictionary website by downloading the mothertongues-UI repository and extracting the files to your computer. Once complete, copy the two files outputed by the ``mtd export`` function to the following location in your Mothertongues-UI:

``mothertongues-UI-master/www/assets/js``

These files need to be renamed as config.js and dict_cached.js, respectively. Note that these files already exist, and contain a set of sample entries. You should overwrite these files with your dictionary data files.

Once done, open the file ``mothertongues-UI-master/www/index.html`` and your dictionary will load in your default browser.



*More info soon*
