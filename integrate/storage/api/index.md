---
title: Storage API
permalink: /integrate/storage/api/
---

* TOC
{:toc}

For a general introduction to working with KBC APIs, see the [API Introduction](/overview/api/).
The [Storage API](http://docs.keboola.apiary.io/) provides a number of functions. These are the most important ones:

- [Component configurations](http://docs.keboola.apiary.io/#reference/component-configurations/)
- [Storage tables](http://docs.keboola.apiary.io/#reference/tables)
- [File uploads](http://docs.keboola.apiary.io/#reference/files)
- [Storage buckets](http://docs.keboola.apiary.io/#reference/buckets)

Virtually, all API calls require a [Storage API token](https://help.keboola.com/storage/tokens/) to
be passed as the `X-StorageApi-Token` header. 
Please note that the Storage API calls require the request to be sent
as `form-data` (unlike the rest of KBC API, which is sent as `application/json`). 

For exporting tables from and importing tables to Storage, we highly recommend that you use one of the
[available clients](/integrate/storage/) or the [Storage API Importer service](/integrate/storage/api/importer/).

Continue reading the following sections for guidance on how to get started:

- [Storage importer service for the easiest upload of data via API](/integrate/storage/api/importer/)
- [Getting started with component configurations](/integrate/storage/api/configurations/)
- [Importing and exporting data](/integrate/storage/api/import-export/)
- [TDE exporter for exporting data to Tableau Data Extracts](/integrate/storage/api/tde-exporter/)