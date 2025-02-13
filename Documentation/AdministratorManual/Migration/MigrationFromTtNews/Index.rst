.. ==================================================
.. FOR YOUR INFORMATION
.. --------------------------------------------------
.. -*- coding: utf-8 -*- with BOM.

.. include:: /Includes.rst.txt


Migration from tt_news to news
------------------------------
This tutorial will help you to migrate data from tt_news to news.

Requirements
^^^^^^^^^^^^
- Installed extension news
- Installed extension news_ttnewsimport

.. hint:: If you experience any troubles during imports, test the latest versions of the 2 extensions which can be found at https://git.typo3.org/TYPO3CMS/Extensions/news.git and https://github.com/fsaris/news_ttnewsimport.

Migration
^^^^^^^^^

Migration of records
""""""""""""""""""""
To be able to migrate records, you need to activate the import module.
This needs to be done in the configuration of EXT:news inside the Extension Manager.

#. Activate the checkbox "Show importer", save and reload the backend. Now you should see the backend module "News Import".
#. Switch to the backend module.
#. Select "Import tt_news category records" from the select box and start the import of categories.
#. Select "Import tt_news news records" from the select box and start the import of news records.


Migration of plugins
""""""""""""""""""""
The plugins of tt_news can be migrated to plugins of EXT:news as well. This is done by using the CLI:

.. code-block:: bash

	./typo3/cli_dispatch.phpsh extbase ttnewspluginmigrate:run
	./typo3/cli_dispatch.phpsh extbase ttnewspluginmigrate:removeOldPlugins

Read more about the migration and its limitation in the documentation of news_ttnewsimport at https://github.com/fsaris/news_ttnewsimport.

Migration of unique aliases
"""""""""""""""""""""""""""
If a lot of similar titles are used it might be a good a idea to migrate the unique aliases to be sure that the same alias is used.

.. code-block:: sql

   # temporary table
   CREATE TABLE tx_realurl_uniqalias_migration LIKE tx_realurl_uniqalias;
   # copy
   INSERT INTO tx_realurl_uniqalias_migration SELECT * FROM tx_realurl_uniqalias WHERE tablename='tt_news';
   # fix it
   UPDATE tx_realurl_uniqalias_migration SET value_id = (SELECT tx_news_domain_model_news.uid FROM `tx_news_domain_model_news` WHERE tx_news_domain_model_news.import_id=tx_realurl_uniqalias_migration.value_id),tablename='tx_news_domain_model_news' WHERE tablename='tt_news';
   # remove wrong alias (news which have not been imported)
   DELETE FROM tx_realurl_uniqalias_migration WHERE tablename='tx_news_domain_model_news' AND value_id=0;
   # insert alias back into realurl table
   INSERT INTO tx_realurl_uniqalias (tstamp,tablename,field_id,value_alias,value_id,lang,expire) SELECT tstamp,tablename,field_id,value_alias,value_id,lang,expire FROM tx_realurl_uniqalias_migration


Not migrated
""""""""""""
Be aware that some things are **not** migrated:

- Templates
- TypoScript configuration
- Backenduser & -group configuration
- Additional fields added to tt_news by 3rd party extensions
