---
title: Mapping Tutorial
permalink: /extend/generic-extractor/tutorial/mapping/
---

* TOC
{:toc}

In the previous part of the tutorial, you [extracted the content of a MailChimp campaign](/extend/generic-extractor/tutorial/jobs/). 
Now it's time to clean up the response.

This is the initial configuration:

{% highlight json %}
{
    "parameters": {
        "api": {
            "baseUrl": "https://us13.api.mailchimp.com/3.0/",
            "authentication": {
                "type": "basic"
            },
            "pagination": {
                "method": "offset",
                "offsetParam": "offset",
                "limitParam": "count",
                "limit": 1
            }
        },
        "config": {
            "debug": true,
            "username": "dummy",
            "#password": "c40xxxxxxxxxxxxxxxxxxxxxxxxxxxxx-us13",
            "outputBucket": "ge-tutorial",
            "jobs": [
                {
                    "endpoint": "campaigns",
                    "dataField": "campaigns",
                    "children": [
                        {
                            "endpoint": "campaigns/{campaign_id}/send-checklist",
                            "dataField": "items",
                            "placeholders": {
                                "campaign_id": "id"
                            }
                        },
                        {
                            "endpoint": "campaigns/{campaign_id}/content",
                            "dataField": ".",
                            "placeholders": {
                                "campaign_id": "id"
                            }
                        }
                    ]
                }
            ]
        }
    }
}
{% endhighlight %}

It extracts MailChimp campaigns together with the `send-checklist` items and campaign `content`. 
However, there are some parts of the content resource you are probably not 
interested in. Also,the table contains duplicates.

***Technical note on duplicates:** If you examine the job events, you will see 
the request `GET /3.0/campaigns/f7ed43aaea/content?count=1&offset=0` sent. That is to say, the 
pagination applies to **all API requests**. Generic Extractor tries to page the 
unpaged `/content` resource. This may ultimately lead to duplicates because the extraction of that
resource is terminated only after the resource returns the same response twice.*

## Mapping
A mapping defines the shape of Generic Extractor outputs. It is stored
in the `config.mappings` property and is identified by the resource data type. 
When a resource is assigned an internal `dataType`, a mapping can be created 
for it. To be able to use a mapping, first define a `dataType` in the job property. 
For example:


{% highlight json %}
{
    "endpoint": "campaigns/{campaign_id}/content",
    "dataField": ".",
    "dataType": "content",
    "placeholders": {
        "campaign_id": "id"
    }
}
{% endhighlight %}

The value of the `dataType` property is an arbitrary name. Apart from identifying
the resource type, it is also used as the **output table name**. If you run
the job, the content will be stored in `in.c-ge-tutorial.content`.

Each mapping item is identified by the property name of the resource and must contain 
`mapping.destination` with the target column name in the output table. For example:

{% highlight json %}
"mappings": {
    "content": {
        "plain_text": {
            "mapping": {
                "destination": "text"
            }
        }
{% endhighlight %}

The above mapping setting defines that for the `content` data type, the 
resource property `plain_text` will be stored in the table column `text`. No other
properties of the content resource will be imported. In other words, the mapping defines
all columns of the output table.

To give an example, if you are interested in having the `plain_text` and `html` versions of the 
campaign content, use a mapping like this:

{% highlight json %}
"mappings": {
    "content": {
        "plain_text": {
            "mapping": {
                "destination": "text"
            }
        },
        "html": {
            "mapping": {
                "destination": "html"
            }
        }
    }
}
{% endhighlight %}

Note that the `destination` value is arbitrary, but it must be a valid column name.
The data type name (`content`) must match the value of the `dataType` property 
as defined in some of the jobs.

## Parent Reference
The above mapping works, but is missing the campaign id and you would not be able to 
match the content to some campaign records. Therefore you need to extract the campaign id 
from the context (i.e. from the job parameter). This can be done using a special `user` mapping.

When the mapping `type` is set to `user`, use the special prefix `parent_` to refer to
a `placeholder` defined in the job. You can create the following mapping:

{% highlight json %}
"mappings": {
    "content": {
        "parent_id": {
            "type": "user",
            "mapping": {
                "destination": "campaign_id"
            }
        }
    }
}
{% endhighlight %}

The above configuration defines a mapping for the `content` data type.
In the result table named `content`, the column `campaign_id` will be created.
Its content will be the value of the `id` placeholder 
(`parent_id` minus the `parent_` prefix) in the respective job.

Apart from specifying what columns should be present in the output table, the 
mapping allows you to set a column to be part of a primary key. The entire configuration would 
then look like this:

{% highlight json %}
{
    "parameters": {
        "api": {
            "baseUrl": "https://us13.api.mailchimp.com/3.0/",
            "authentication": {
                "type": "basic"
            },
            "pagination": {
                "method": "offset",
                "offsetParam": "offset",
                "limitParam": "count",
                "limit": 10
            }
        },
        "config": {
            "debug": true,
            "username": "dummy",
            "#password": "c40xxxxxxxxxxxxxxxxxxxxxxxxxxxxx-us13",
            "outputBucket": "ge-tutorial",
            "jobs": [
                {
                    "endpoint": "campaigns",
                    "dataField": "campaigns",
                    "children": [
                        {
                            "endpoint": "campaigns/{campaign_id}/send-checklist",
                            "dataField": "items",
                            "placeholders": {
                                "campaign_id": "id"
                            }
                        },
                        {
                            "endpoint": "campaigns/{campaign_id}/content",
                            "dataField": ".",
                            "dataType": "content",
                            "placeholders": {
                                "campaign_id": "id"
                            }
                        }
                    ]
                }
            ],
            "mappings": {
                "content": {
                    "parent_id": {
                        "type": "user",
                        "mapping": {
                            "destination": "campaign_id",
                            "primaryKey": true
                        }
                    },
                    "plain_text": {
                        "mapping": {
                            "destination": "text"
                        }
                    },
                    "html": {
                        "mapping": {
                            "destination": "html"
                        }
                    }
                }
            }
        }
    }
}
{% endhighlight %}

## Review
Now, let's review what parts are connected and how. Note that the values in blue 
have been chosen arbitrarily when the configuration was created:

{: .image-popup}
![Configuration Schema](/extend/generic-extractor/tutorial/configuration-schema.svg)

## Summary
Mapping lets you define precisely what the extraction output will look like; it also 
defines primary keys. 

If you are doing a one time ad-hoc extraction, you may skip setting up the mapping and clean 
the extracted data later in [Transformations](https://help.keboola.com/manipulation/transformations/). 
However, if you intend to use your configuration regularly, or want to make it into its own component, 
setting up a mapping is recommended.

## Tips and trics

The key of the mapping supports dot notation to traverse into children. So if the key contains a dot, you need to change the delimiter. See the following example: 

{% highlight json %}
"mappings": {
    "content": {
        "created.date": {
            "delimiter": "/",
            "type": "column",
            "mapping": {
                "destination": "parent_category"
            }
        }
    }
}
{% endhighlight %}

As you changed the delimiter from the default `.` to `/`, it's no longer parsed as two separate keys `created` and `date`, but rather just a single key `created.date`.
